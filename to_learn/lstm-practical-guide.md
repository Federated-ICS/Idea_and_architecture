# LSTM for ICS Anomaly Detection - Practical Guide

## The Real-World Problem

You're monitoring a water treatment plant with sensors measuring:
- **Temperature**: Should be 20-25°C during normal operation
- **Pressure**: Should be 2.0-2.5 bar
- **Flow Rate**: Should be 100-120 L/min

**The Challenge**: Detect when something abnormal happens (equipment failure, cyberattack, sensor malfunction) in real-time.

---

## Why LSTM? The Temporal Pattern Problem

### Scenario 1: Simple Threshold Alert (Doesn't Work Well)
```
Temperature > 30°C → ALERT!
```

**Problem**: Misses subtle attacks. An attacker could slowly increase temperature from 22°C to 28°C over 2 hours. Each individual reading looks "okay" but the pattern is dangerous.

### Scenario 2: LSTM Autoencoder (Works Great!)
The LSTM learns **temporal patterns**:
- "Temperature usually rises 0.1°C per minute during startup"
- "When pressure drops, flow rate drops 2 minutes later"
- "Night shift has different patterns than day shift"

When patterns break, it detects anomalies even if individual values seem normal.

---

## How LSTM Autoencoder Works for Anomaly Detection

### Step 1: Training on Normal Data

```
Normal Operation Data (1 week):
┌─────────────────────────────────────┐
│ Time    Temp   Press   Flow         │
│ 00:00   22.1   2.2     105          │
│ 00:01   22.1   2.2     106          │
│ 00:02   22.2   2.2     105          │
│ ...     ...    ...     ...          │
└─────────────────────────────────────┘
```

**What the LSTM learns:**
1. **Encoder LSTM**: Compresses the sequence into a compact representation
   - "This looks like normal nighttime operation"
   
2. **Decoder LSTM**: Tries to reconstruct the original sequence
   - "Based on my understanding, the next values should be..."

**Training Goal**: Minimize reconstruction error on normal data

```
Input Sequence:  [22.1, 22.1, 22.2, 22.3, 22.4]
Reconstructed:   [22.0, 22.1, 22.2, 22.3, 22.5]
Error:           [0.1,  0.0,  0.0,  0.0,  0.1]  ← Very low!
```

---

### Step 2: Detection During Operation

When new data arrives, the LSTM tries to reconstruct it:

#### Normal Data Example:
```
Input:         [22.1, 22.2, 22.3, 22.4, 22.5]
Reconstructed: [22.0, 22.2, 22.3, 22.4, 22.6]
Error:         0.08  ← Low error = Normal ✓
```

#### Anomaly Example (Cyberattack):
```
Input:         [22.1, 22.2, 28.5, 29.1, 30.2]  ← Sudden spike!
Reconstructed: [22.0, 22.2, 22.4, 22.5, 22.6]  ← LSTM expects normal pattern
Error:         6.2   ← High error = ANOMALY! 🚨
```

**Why it works**: The LSTM has never seen this attack pattern during training, so it can't reconstruct it well.

---

## The Magic: How LSTM Remembers Patterns

This is where we dive into the theory...

### The Problem with Regular Neural Networks

Imagine trying to detect an attack that unfolds over 30 minutes (1800 timesteps):

```
Regular Feedforward Network:
Input: [temp, pressure, flow] → Output: normal/anomaly
❌ Can't see that pressure has been slowly dropping for 20 minutes
```

```
Regular RNN:
Can see previous timesteps BUT...
❌ Forgets information from 1000+ steps ago (vanishing gradient)
```

### LSTM Solution: The Cell State Highway

Think of the LSTM cell state as a **protected information highway** that runs through time:

```
Timestep 1 → Timestep 2 → Timestep 3 → ... → Timestep 1800
    ↓            ↓            ↓                    ↓
[Cell State carries important info across all timesteps]
```

**Key Innovation**: Information can flow across thousands of timesteps without degrading!

---

## The Three Gates (Simplified)

At each timestep, the LSTM asks three questions:

### 1. Forget Gate: "What should I forget?"
```python
# Pseudocode
if new_shift_started:
    forget_old_shift_patterns = True
else:
    keep_recent_patterns = True
```

**Real Example**: 
- At 8 AM, day shift starts
- Forget gate: "Forget night shift patterns, they're not relevant anymore"

### 2. Input Gate: "What new information should I remember?"
```python
# Pseudocode
if pressure_drop_detected:
    remember_this = True  # Important for detecting cascading failures
else:
    ignore_this = False
```

**Real Example**:
- Pressure drops from 2.3 to 2.1 bar
- Input gate: "This is important! Remember this pressure drop"

### 3. Output Gate: "What should I output now?"
```python
# Pseudocode
based_on_everything_i_know:
    output_current_prediction()
```

**Real Example**:
- Based on the pressure drop 5 minutes ago + current temperature rise
- Output gate: "This looks like the start of an attack pattern"

---

## Practical Architecture for Your ICS System

### Input Shape
```python
# Looking at last 50 timesteps (50 minutes of data)
# 3 features per timestep (temp, pressure, flow)
input_shape = (50, 3)
```

### LSTM Autoencoder Architecture
```
INPUT: (50 timesteps, 3 features)
    ↓
ENCODER LSTM Layer 1: 64 units
    ↓ (learns high-level patterns)
ENCODER LSTM Layer 2: 32 units  
    ↓ (compresses to compact representation)
BOTTLENECK: 16 units (compressed knowledge)
    ↓
DECODER LSTM Layer 1: 32 units
    ↓ (starts reconstructing)
DECODER LSTM Layer 2: 64 units
    ↓ (full reconstruction)
OUTPUT: (50 timesteps, 3 features)
```

### Detection Logic
```python
# 1. Calculate reconstruction error
error = mean_squared_error(input, reconstructed_output)

# 2. Compare to threshold (learned from training)
threshold = 0.5  # Determined during validation

# 3. Detect anomaly
if error > threshold:
    trigger_alert("Anomaly detected!")
    log_for_forensics()
```

---

## Why This Works for Your Federated Learning Project

### Each Facility Trains Locally
```
Facility A (Power Plant):
- Trains LSTM on their normal operation data
- Learns: "Our temperature cycles every 4 hours"

Facility B (Water Treatment):
- Trains LSTM on their normal operation data  
- Learns: "Our pressure spikes during cleaning cycles"
```

### Federated Aggregation
```
1. Each facility trains LSTM locally
2. Share only model weights (not data!)
3. Central server aggregates weights
4. Everyone gets improved model that learned from all facilities
```

**Result**: Facility A can now detect attacks similar to ones Facility B experienced, without ever seeing Facility B's data!

---

## Next Steps: Understanding the Math

Now that you see WHY and HOW LSTMs work for your use case, we can dive deeper into:

1. **Gate Equations**: The actual math behind forget/input/output gates
2. **Backpropagation Through Time**: How LSTMs learn from sequences
3. **Hyperparameter Tuning**: Choosing sequence length, hidden units, etc.
4. **Advanced Techniques**: Bidirectional LSTMs, attention mechanisms

**Ready to go deeper? Let me know which topic interests you most!**

