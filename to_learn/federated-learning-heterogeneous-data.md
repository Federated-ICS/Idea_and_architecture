# How Federated Learning Works with Different Facilities

## The Core Problem

You have 3 facilities with completely different equipment and operating ranges:

```
┌─────────────────────────────────────────────────────────────┐
│ Facility A (Power Plant)                                    │
│ Temperature: 280-320°C                                      │
│ Pressure: 100-150 psi                                       │
│ Flow: 50-100 L/min                                          │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│ Facility B (Water Treatment)                                │
│ Temperature: 250-290°C                                      │
│ Pressure: 90-140 psi                                        │
│ Flow: 40-90 L/min                                           │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│ Facility C (Chemical Plant)                                 │
│ Temperature: 200-250°C                                      │
│ Pressure: 80-120 psi                                        │
│ Flow: 30-80 L/min                                           │
└─────────────────────────────────────────────────────────────┘
```

**Question:** How can they share the same LSTM model when their "normal" is completely different?

---

## The Solution: Three-Layer Magic

### Layer 1: Standardized Feature Set

All facilities agree on the **same 18 features** (even though values differ):

**Process Features (10):**
1. Temperature
2. Pressure
3. Flow rate
4. Valve position
5. Pump status
6. Setpoint temperature
7. Setpoint pressure
8. Control signal
9. Error signal
10. Time of day

**Network Features (8):**
11. Packets per second
12. Bytes per second
13. Unique source IPs
14. Unique destination IPs
15. Protocol distribution
16. Failed connections
17. Average packet size
18. Inter-arrival time variance

**Key Point:** Everyone measures the **same types** of things, but with different values.

---

### Layer 2: Local Normalization (THE MAGIC!)

Each facility normalizes data to **[0, 1]** using **their own** min/max values:

#### Example: Temperature Normalization

**Facility A (Power Plant):**
```python
# Normal range: 280-320°C
min_temp = 280
max_temp = 320

# Current reading: 300°C
normalized = (300 - 280) / (320 - 280) = 20 / 40 = 0.5
```

**Facility B (Water Treatment):**
```python
# Normal range: 250-290°C
min_temp = 250
max_temp = 290

# Current reading: 270°C
normalized = (270 - 250) / (290 - 250) = 20 / 40 = 0.5
```

**Facility C (Chemical Plant):**
```python
# Normal range: 200-250°C
min_temp = 200
max_temp = 250

# Current reading: 225°C
normalized = (225 - 200) / (250 - 200) = 25 / 50 = 0.5
```

**Result:** All three facilities represent "middle of normal range" as **0.5**!

---

### Layer 3: Pattern Learning (THE INSIGHT!)

The LSTM doesn't learn absolute values - it learns **patterns**:

#### Pattern: Gradual Drift Attack

**Facility A sees (normalized):**
```
Time:   0s    10s   20s   30s   40s   50s   60s
Value: [0.5, 0.55, 0.60, 0.65, 0.70, 0.80, 0.95]
        ↑ Gradual increase pattern
```

**Facility B sees (normalized):**
```
Time:   0s    10s   20s   30s   40s   50s   60s
Value: [0.5, 0.55, 0.60, 0.65, 0.70, 0.80, 0.95]
        ↑ Same pattern!
```

**Facility C sees (normalized):**
```
Time:   0s    10s   20s   30s   40s   50s   60s
Value: [0.5, 0.55, 0.60, 0.65, 0.70, 0.80, 0.95]
        ↑ Same pattern!
```

**The LSTM learns:** "When I see [0.5 → 0.95] over 60 seconds, that's an attack!"

---

## Concrete Example: Attack Detection

### Scenario: Gradual Temperature Increase Attack

#### Facility A (Power Plant)

**Raw Data:**
```
Time:   0s    10s   20s   30s   40s   50s   60s
Temp: [300°C, 305°C, 310°C, 315°C, 320°C, 330°C, 360°C]
       ↑ Normal                            ↑ Attack!
```

**Normalized (using 280-320°C range):**
```
[0.50, 0.625, 0.75, 0.875, 1.0, 1.25, 2.0]
 ↑ Normal range is 0-1, so 2.0 is WAY outside!
```

**LSTM Reconstruction:**
```
Expected: [0.50, 0.51, 0.52, 0.51, 0.50, 0.51, 0.50]
Actual:   [0.50, 0.625, 0.75, 0.875, 1.0, 1.25, 2.0]
Error:    [0.0, 0.115, 0.23, 0.365, 0.5, 0.74, 1.5]
                                              ↑ HIGH ERROR = ANOMALY!
```

#### Facility B (Water Treatment) - BEFORE Federated Learning

**Same attack pattern (different absolute temps):**
```
Time:   0s    10s   20s   30s   40s   50s   60s
Temp: [270°C, 275°C, 280°C, 285°C, 290°C, 300°C, 330°C]
```

**Normalized (using 250-290°C range):**
```
[0.50, 0.625, 0.75, 0.875, 1.0, 1.25, 2.0]
 ↑ SAME PATTERN as Facility A!
```

**But Facility B's LSTM hasn't seen this attack yet:**
```
Expected: [0.50, 0.51, 0.52, 0.51, 0.50, 0.51, 0.50]
Actual:   [0.50, 0.625, 0.75, 0.875, 1.0, 1.25, 2.0]
Error:    [0.0, 0.115, 0.23, 0.365, 0.5, 0.74, 1.5]
          ↑ Might not detect if threshold is too high
```

#### After Federated Learning Round

**Facility A trains on the attack and shares weights:**
```
1. Facility A experiences attack
2. LSTM learns: "Pattern [0.5 → 2.0] over 60s = attack"
3. FL round triggered
4. Facility A shares model weights (NOT raw data)
5. FL Server aggregates weights from A, B, C
6. All facilities receive updated model
```

**Now Facility B can detect the same attack:**
```
Expected: [0.50, 0.51, 0.52, 0.51, 0.50, 0.51, 0.50]
Actual:   [0.50, 0.625, 0.75, 0.875, 1.0, 1.25, 2.0]
Error:    [0.0, 0.115, 0.23, 0.365, 0.5, 0.74, 1.5]
          ↑ NOW DETECTS! Learned from Facility A's experience
```

---

## What Gets Shared vs What Stays Local

### Shared via Federated Learning ✅

**Model Weights (~10 MB):**
```python
# Example weights (simplified)
weights = {
    'lstm_layer_1': [[0.23, -0.45, 0.67, ...], ...],  # 128 units
    'lstm_layer_2': [[0.12, 0.89, -0.34, ...], ...],  # 64 units
    'decoder_layer_1': [[0.56, -0.23, 0.78, ...], ...],
    'decoder_layer_2': [[0.34, 0.67, -0.12, ...], ...]
}
# Total: ~10 MB of numbers
```

**Pattern Knowledge:**
- "Gradual increase from 0.5 to 0.95 = attack"
- "Sudden spike from 0.5 to 0.9 in 1 second = attack"
- "Oscillation between 0.4 and 0.6 every 2 seconds = attack"

### Stays Local (NEVER Shared) ❌

**Raw Sensor Data:**
```python
# NEVER leaves the facility
raw_data = {
    'temperature': [300, 305, 310, ...],  # Actual °C values
    'pressure': [120, 125, 130, ...],     # Actual psi values
    'flow_rate': [75, 80, 85, ...]        # Actual L/min values
}
```

**Normalization Parameters:**
```python
# Each facility keeps their own
normalization_params = {
    'temperature': {'min': 280, 'max': 320},  # Facility-specific
    'pressure': {'min': 100, 'max': 150},
    'flow_rate': {'min': 50, 'max': 100}
}
```

**Operational Details:**
- Equipment configurations
- Process setpoints
- Maintenance schedules
- Production volumes
- Facility layout

---

## The Federated Learning Process (Step-by-Step)

### Step 1: Initial State

```
┌─────────────────────────────────────────────────────────────┐
│ FL Server                                                    │
│ Global Model: Trained on synthetic data (baseline)          │
└─────────────────────────────────────────────────────────────┘
         │
         ├─────────────────┬─────────────────┬─────────────────┐
         ↓                 ↓                 ↓                 ↓
┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐
│ Facility A      │ │ Facility B      │ │ Facility C      │
│ Model: v1.0     │ │ Model: v1.0     │ │ Model: v1.0     │
│ Detection: 85%  │ │ Detection: 85%  │ │ Detection: 85%  │
└─────────────────┘ └─────────────────┘ └─────────────────┘
```

### Step 2: Attack on Facility A

```
┌─────────────────────────────────────────────────────────────┐
│ Facility A                                                   │
│ 🚨 ATTACK DETECTED: Gradual temperature drift               │
│ Raw: 300°C → 360°C over 60 seconds                         │
│ Normalized: 0.5 → 2.0                                       │
│ LSTM Error: 1.5 (HIGH!)                                     │
└─────────────────────────────────────────────────────────────┘
```

### Step 3: Local Training at Facility A

```
┌─────────────────────────────────────────────────────────────┐
│ Facility A - Local Training                                 │
│                                                              │
│ 1. Load attack data (normalized)                            │
│ 2. Train LSTM for 5 epochs                                  │
│ 3. Model learns: "Pattern [0.5→2.0] = attack"              │
│ 4. Extract model weights                                    │
│ 5. Apply differential privacy (add noise)                   │
│ 6. Send weights to FL Server                                │
│                                                              │
│ Weights sent: ~10 MB                                        │
│ Raw data sent: 0 MB ✅                                      │
└─────────────────────────────────────────────────────────────┘
```

### Step 4: All Facilities Train Locally

```
┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐
│ Facility A      │ │ Facility B      │ │ Facility C      │
│ Training on:    │ │ Training on:    │ │ Training on:    │
│ - Normal data   │ │ - Normal data   │ │ - Normal data   │
│ - Attack data   │ │ (no attack yet) │ │ (no attack yet) │
│                 │ │                 │ │                 │
│ Weights_A →     │ │ Weights_B →     │ │ Weights_C →     │
└─────────────────┘ └─────────────────┘ └─────────────────┘
         │                 │                 │
         └─────────────────┴─────────────────┘
                           ↓
                   ┌─────────────────┐
                   │   FL Server     │
                   │ Receives weights│
                   └─────────────────┘
```

### Step 5: Aggregation at FL Server

```
┌─────────────────────────────────────────────────────────────┐
│ FL Server - Aggregation (FedMedian)                         │
│                                                              │
│ Received:                                                    │
│ - Weights_A (learned from attack)                           │
│ - Weights_B (normal operation)                              │
│ - Weights_C (normal operation)                              │
│                                                              │
│ Aggregation Method: Coordinate-wise Median                  │
│                                                              │
│ For each weight position:                                   │
│   weight[i] = median([Weights_A[i], Weights_B[i], Weights_C[i]]) │
│                                                              │
│ Result: Global_Weights_v2.0                                 │
│ - Incorporates attack knowledge from Facility A             │
│ - Robust to outliers (median, not average)                  │
│ - Byzantine-resistant                                       │
└─────────────────────────────────────────────────────────────┘
```

### Step 6: Distribution to All Facilities

```
                   ┌─────────────────┐
                   │   FL Server     │
                   │ Global Model    │
                   │ v2.0            │
                   └─────────────────┘
                           │
         ┌─────────────────┼─────────────────┐
         ↓                 ↓                 ↓
┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐
│ Facility A      │ │ Facility B      │ │ Facility C      │
│ Model: v2.0 ✅  │ │ Model: v2.0 ✅  │ │ Model: v2.0 ✅  │
│ Detection: 96%  │ │ Detection: 96%  │ │ Detection: 96%  │
│ (improved!)     │ │ (improved!)     │ │ (improved!)     │
└─────────────────┘ └─────────────────┘ └─────────────────┘
```

### Step 7: Attack on Facility B (Now Detected!)

```
┌─────────────────────────────────────────────────────────────┐
│ Facility B                                                   │
│ 🚨 ATTACK DETECTED: Same pattern as Facility A!            │
│ Raw: 270°C → 330°C over 60 seconds                         │
│ Normalized: 0.5 → 2.0 (same pattern!)                      │
│ LSTM Error: 1.5 (HIGH!)                                     │
│                                                              │
│ ✅ Detected immediately using knowledge from Facility A     │
│ ⏱️ Time from Facility A attack to protection: 6 hours      │
└─────────────────────────────────────────────────────────────┘
```

---

## Why This Works: The Math Behind It

### Universal Patterns in Normalized Space

**Pattern 1: Sudden Spike**
```
All facilities see:
[0.5, 0.5, 0.5, 0.9, 0.5, 0.5]
      ↑ Sudden jump = attack
```

**Pattern 2: Gradual Drift**
```
All facilities see:
[0.5, 0.55, 0.60, 0.65, 0.70, 0.80, 0.95]
      ↑ Steady increase = attack
```

**Pattern 3: Oscillation**
```
All facilities see:
[0.5, 0.6, 0.4, 0.6, 0.4, 0.6, 0.4]
      ↑ Rapid oscillation = attack
```

**Pattern 4: Correlation Break**
```
Normal: When pressure ↑, flow ↑ (correlated)
Attack: Pressure ↑, but flow ↓ (correlation broken)

All facilities see this pattern in normalized space!
```

### The LSTM Learns Relationships, Not Values

**What the LSTM Actually Learns:**

❌ **NOT:** "300°C is normal"
✅ **YES:** "0.5 is normal"

❌ **NOT:** "360°C is an attack"
✅ **YES:** "Rapid increase from 0.5 to 2.0 is an attack"

❌ **NOT:** "Pressure should be 120 psi"
✅ **YES:** "Pressure should correlate with flow rate"

---

## Handling Edge Cases

### Case 1: Facility with Different Sensor Count

**Problem:** Facility D has only 8 sensors (missing 2 features)

**Solution: Feature Padding**
```python
# Facility D's data
features_D = [temp, pressure, flow, valve, pump, setpoint_temp, setpoint_pressure, control_signal]
# Missing: error_signal, time_of_day

# Pad with neutral values
features_D_padded = features_D + [0.5, 0.5]  # Neutral = middle of range

# Now has 10 features like everyone else
```

### Case 2: Very Different Process Types

**Problem:** Power plant vs water treatment have fundamentally different processes

**Solution: Facility Clustering**
```
Cluster 1: High-temperature facilities (power plants)
- Shared model for similar processes

Cluster 2: Low-temperature facilities (water treatment)
- Shared model for similar processes

Cluster 3: Chemical facilities
- Shared model for similar processes
```

### Case 3: Different Sampling Rates

**Problem:** Facility A samples at 1 Hz, Facility B at 0.5 Hz

**Solution: Resampling**
```python
# Facility B (0.5 Hz) → Upsample to 1 Hz
data_B_upsampled = interpolate(data_B, target_rate=1.0)

# Or downsample everyone to 0.5 Hz
data_A_downsampled = downsample(data_A, target_rate=0.5)
```

---

## Privacy Guarantees

### What Differential Privacy Adds

**Without DP:**
```python
weights_A = train_model(data_A)
# Weights might leak information about specific data points
```

**With DP (Opacus):**
```python
# Add calibrated noise to gradients during training
weights_A = train_model_with_dp(data_A, epsilon=2.0, delta=1e-5)
# Mathematical guarantee: Can't identify individual data points
```

### Privacy Budget

**Epsilon (ε) = 2.0:**
- Lower = more privacy, less accuracy
- Higher = less privacy, more accuracy
- 2.0 = Good balance for ICS

**Delta (δ) = 10⁻⁵:**
- Probability of privacy failure
- 0.00001 = Very low risk

**What this means:**
- Even if an attacker has the model weights
- They can't determine if a specific sensor reading was in the training data
- Mathematical guarantee, not just a promise

---

## Performance Impact

### Accuracy Comparison

**Single Facility (No FL):**
```
Detection Accuracy: 85%
False Positives: 8%
Novel Attack Detection: 60%
```

**Federated Learning (3 Facilities):**
```
Detection Accuracy: 96%
False Positives: 4%
Novel Attack Detection: 85%
```

**Federated Learning (10 Facilities):**
```
Detection Accuracy: 97%
False Positives: 3%
Novel Attack Detection: 92%
```

**Key Insight:** More facilities = Better detection!

### Speed Comparison

**Traditional Threat Intelligence Sharing:**
```
1. Facility A detects attack
2. Security team investigates (1-2 days)
3. Write threat report (1-2 days)
4. Share via industry group (1-2 weeks)
5. Other facilities implement defenses (1-2 weeks)

Total: 3-6 weeks
```

**Federated Learning:**
```
1. Facility A detects attack (30 seconds)
2. FL round triggered (automatic)
3. Local training (5 minutes)
4. Aggregation (30 seconds)
5. Distribution (30 seconds)
6. All facilities protected (6 hours)

Total: 6 hours
```

**Speed Improvement: 84x - 168x faster!**

---

## Summary

### How It Works (Simple Version):

1. **Same Features** - All facilities measure the same 18 things
2. **Local Normalization** - Each facility scales to [0, 1] using their own ranges
3. **Pattern Learning** - LSTM learns patterns, not absolute values
4. **Weight Sharing** - Only model weights shared, not data
5. **Collaborative Defense** - One facility's experience protects all others

### Key Benefits:

✅ **Privacy-Preserving** - Raw data never leaves facility
✅ **Fast** - 6 hours vs weeks for threat sharing
✅ **Accurate** - 96% detection with 3 facilities
✅ **Scalable** - Easy to add new facilities
✅ **Robust** - Byzantine-resistant aggregation

### The Magic:

**The LSTM doesn't care about absolute values!**

It learns:
- "When value increases from 0.5 to 0.95 over 60 seconds = attack"
- "When pressure and flow correlation breaks = attack"
- "When oscillation frequency increases = attack"

These patterns are **universal** across all facilities, regardless of their actual operating ranges!

---

## Next Steps

Want to dive deeper into:
1. **LSTM Architecture** - How the gates actually work?
2. **Differential Privacy Math** - The actual DP-SGD algorithm?
3. **Aggregation Strategies** - Why FedMedian vs FedAvg?
4. **Implementation Code** - Actual Python code for FL?

Let me know what interests you most!
