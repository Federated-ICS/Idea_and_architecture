# Handling Missing Sensors in Federated Learning

## The Problem: Different Sensor Configurations

```
Facility A (Power Plant) - 12 sensors:
✅ Temperature, Pressure, Flow, Valve, Pump
✅ Radiation, Turbine Speed, Steam Pressure
✅ Setpoint Temp, Setpoint Pressure, Control Signal, Error Signal

Facility B (Water Treatment) - 10 sensors:
✅ Temperature, Pressure, Flow, Valve, Pump
❌ Radiation, Turbine Speed, Steam Pressure (don't have)
✅ pH Level, Chlorine Level (unique to water treatment)
✅ Setpoint Temp, Setpoint Pressure, Control Signal, Error Signal

Facility C (Chemical Plant) - 11 sensors:
✅ Temperature, Pressure, Flow, Valve, Pump
❌ Radiation, Turbine Speed (don't have)
✅ Concentration, Viscosity, Reaction Rate (unique to chemical)
✅ Setpoint Temp, Setpoint Pressure, Control Signal, Error Signal
```

**Challenge:** How do they share the same LSTM model?

---

## Solution Comparison

### Option 1: Common Features Only (Simplest)

**Use only sensors ALL facilities have:**

Common features: temperature, pressure, flow_rate, valve_position, pump_status, setpoint_temp, setpoint_pressure, control_signal, error_signal, time_of_day (Total: 10 features)

**Pros:** ✅ Simple, ✅ All compatible, ✅ No missing data
**Cons:** ❌ Loses unique sensor info, ❌ Can't detect attacks on unique sensors

---

### Option 2: Feature Padding (Recommended)

**Create universal feature set, pad missing with 0.5:**

Universal features (15 total):
- Common (all have): temperature, pressure, flow_rate, valve_position, pump_status
- Power plant specific: radiation_level, turbine_speed, steam_pressure
- Water treatment specific: ph_level, chlorine_level
- Chemical plant specific: concentration, viscosity, reaction_rate
- Common: setpoint_temp, setpoint_pressure

**Example:**
```
Facility A (Power Plant) - has 12/15:
[0.6, 0.5, 0.7, 0.8, 1.0,  # Common
 0.4, 0.6, 0.7,              # Power plant (HAS)
 0.5, 0.5,                   # Water treatment (MISSING → 0.5)
 0.5, 0.5, 0.5,              # Chemical (MISSING → 0.5)
 0.6, 0.5]                   # Common

Facility B (Water Treatment) - has 10/15:
[0.5, 0.6, 0.5, 0.7, 1.0,  # Common
 0.5, 0.5, 0.5,              # Power plant (MISSING → 0.5)
 0.3, 0.4,                   # Water treatment (HAS)
 0.5, 0.5, 0.5,              # Chemical (MISSING → 0.5)
 0.5, 0.6]                   # Common
```

**Why 0.5?** It's neutral (middle of normalized [0,1] range)

**Pros:** ✅ All facilities participate, ✅ Unique sensors preserved
**Cons:** ❌ Larger input (15 vs 10 features), ❌ Some wasted space

---

### Option 3: Facility Clustering (For Very Different Types)

**Group similar facilities:**

```
Cluster 1: Power Plants (Facilities A, D, F)
- Features: temp, pressure, flow, radiation, turbine_speed, steam_pressure
- FL Model 1

Cluster 2: Water Treatment (Facilities B, E)
- Features: temp, pressure, flow, ph_level, chlorine_level, turbidity
- FL Model 2

Cluster 3: Chemical Plants (Facilities C, G)
- Features: temp, pressure, flow, concentration, viscosity, reaction_rate
- FL Model 3
```

**Pros:** ✅ No wasted features, ✅ Better accuracy within cluster
**Cons:** ❌ More complex, ❌ Smaller clusters, ❌ No cross-cluster learning

---

## How LSTM Learns to Ignore Missing Features

### Training Behavior

**Facility A trains on:**
```
Timestep 1: [0.6, 0.5, 0.7, 0.8, 1.0, 0.4, 0.6, 0.7, 0.5, 0.5, 0.5, 0.5, 0.5, 0.6, 0.5]
Timestep 2: [0.6, 0.5, 0.7, 0.8, 1.0, 0.4, 0.6, 0.7, 0.5, 0.5, 0.5, 0.5, 0.5, 0.6, 0.5]
Timestep 3: [0.6, 0.5, 0.7, 0.8, 1.0, 0.4, 0.6, 0.7, 0.5, 0.5, 0.5, 0.5, 0.5, 0.6, 0.5]
```

**LSTM learns:**
- Features 0-7: Vary over time → Important!
- Features 8-12: Always 0.5 → Ignore (no information)
- Features 13-14: Vary over time → Important!

**Facility B trains on:**
```
Timestep 1: [0.5, 0.6, 0.5, 0.7, 1.0, 0.5, 0.5, 0.5, 0.3, 0.4, 0.5, 0.5, 0.5, 0.5, 0.6]
Timestep 2: [0.5, 0.6, 0.5, 0.7, 1.0, 0.5, 0.5, 0.5, 0.3, 0.4, 0.5, 0.5, 0.5, 0.5, 0.6]
Timestep 3: [0.5, 0.6, 0.5, 0.7, 1.0, 0.5, 0.5, 0.5, 0.3, 0.4, 0.5, 0.5, 0.5, 0.5, 0.6]
```

**LSTM learns:**
- Features 0-4: Vary over time → Important!
- Features 5-7: Always 0.5 → Ignore
- Features 8-9: Vary over time → Important!
- Features 10-12: Always 0.5 → Ignore
- Features 13-14: Vary over time → Important!

### After Federated Learning

**Global model learns:**
- Features 0-4: Important for ALL facilities
- Features 5-7: Important for SOME facilities (power plants)
- Features 8-9: Important for SOME facilities (water treatment)
- Features 10-12: Important for SOME facilities (chemical plants)
- Features 13-14: Important for ALL facilities

**Result:** Model works for everyone!

---

## Attack Detection Example

### Scenario: Temperature Attack

**Attack on Facility A (Power Plant):**
```
Normal:  [0.5, 0.5, 0.7, 0.8, 1.0, 0.4, 0.6, 0.7, 0.5, 0.5, 0.5, 0.5, 0.5, 0.5, 0.5]
Attack:  [0.9, 0.5, 0.7, 0.8, 1.0, 0.4, 0.6, 0.7, 0.5, 0.5, 0.5, 0.5, 0.5, 0.5, 0.5]
          ↑ Temperature spike!
```

**After FL, attack on Facility B (Water Treatment):**
```
Normal:  [0.5, 0.6, 0.5, 0.7, 1.0, 0.5, 0.5, 0.5, 0.3, 0.4, 0.5, 0.5, 0.5, 0.5, 0.6]
Attack:  [0.9, 0.6, 0.5, 0.7, 1.0, 0.5, 0.5, 0.5, 0.3, 0.4, 0.5, 0.5, 0.5, 0.5, 0.6]
          ↑ Same temperature spike pattern!
```

**LSTM detects:** "Feature 0 (temperature) jumped from 0.5 to 0.9 = attack!"

**Works because:** The pattern (temperature spike) is in a common feature that both facilities have.

---

## What About Attacks on Unique Sensors?

### Scenario: Radiation Attack (Only Power Plants Have This)

**Attack on Facility A:**
```
Normal:  [0.5, 0.5, 0.7, 0.8, 1.0, 0.4, 0.6, 0.7, 0.5, 0.5, 0.5, 0.5, 0.5, 0.5, 0.5]
Attack:  [0.5, 0.5, 0.7, 0.8, 1.0, 0.9, 0.6, 0.7, 0.5, 0.5, 0.5, 0.5, 0.5, 0.5, 0.5]
                                      ↑ Radiation spike!
```

**Can Facility B (Water Treatment) benefit?**

**Answer: Indirectly, yes!**

1. **Facility A learns:** "Feature 5 spike = attack"
2. **FL aggregation:** Facility A's weights influence global model
3. **Facility B receives:** Updated model with radiation attack knowledge
4. **Facility B's radiation feature:** Always 0.5 (padded)
5. **If Facility B gets radiation sensor later:** Immediately protected!

**Also:** Attacks often affect multiple sensors:
```
Radiation attack might also cause:
- Temperature increase (Feature 0) ← Facility B HAS this!
- Pressure change (Feature 1) ← Facility B HAS this!
```

So Facility B can still detect the attack via correlated features!

---

## Solution 4: Multi-Task Learning with Masking (Most Advanced)

### Concept: Explicit Masking

Instead of relying on the LSTM to learn that 0.5 means "missing", we **explicitly tell** the model which features are valid using a mask.

**Mask:** Binary array where 1 = "real sensor", 0 = "missing sensor"

```
Facility A (Power Plant):
features = [0.6, 0.5, 0.7, 0.8, 1.0, 0.4, 0.6, 0.7, 0.5, 0.5, 0.5, 0.5, 0.5, 0.6, 0.5]
mask =     [1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 0.0, 0.0, 0.0, 0.0, 0.0, 1.0, 1.0]
            ↑ Real sensors (1.0)                      ↑ Missing sensors (0.0)

Facility B (Water Treatment):
features = [0.5, 0.6, 0.5, 0.7, 1.0, 0.5, 0.5, 0.5, 0.3, 0.4, 0.5, 0.5, 0.5, 0.5, 0.6]
mask =     [1.0, 1.0, 1.0, 1.0, 1.0, 0.0, 0.0, 0.0, 1.0, 1.0, 0.0, 0.0, 0.0, 1.0, 1.0]
            ↑ Real sensors          ↑ Missing        ↑ Real      ↑ Missing
```

### How Masking Works

**Architecture: Masked LSTM Autoencoder**

1. **Input Masking:** Multiply input by mask (zero out missing sensors)
   - `x_masked = x * mask`

2. **Encoding:** LSTM processes masked input
   - Encoder LSTM Layer 1: input_size → hidden_size (128)
   - Encoder LSTM Layer 2: hidden_size → latent_size (64)

3. **Decoding:** LSTM reconstructs from latent representation
   - Decoder LSTM Layer 1: latent_size → hidden_size (128)
   - Decoder LSTM Layer 2: hidden_size → input_size

4. **Output Masking:** Multiply output by mask (only reconstruct valid sensors)
   - `reconstructed_masked = reconstructed * mask`

5. **Loss Computation:** Calculate error only on valid features
   - `squared_error = (x - reconstructed)²`
   - `masked_error = squared_error * mask`
   - `loss = masked_error.sum() / mask.sum()`

### Advanced: Attention-Based Masking

For even better performance, use attention mechanism to dynamically weight features:

**Components:**
1. **Encoder LSTM:** Processes masked input
2. **Attention Mechanism:** Learns which timesteps are important
   - Computes attention scores for each timestep
   - Masks attention scores (don't attend to missing features)
   - Applies softmax over valid timesteps
3. **Context Vector:** Weighted sum of encoded representations
4. **Latent Representation:** Compressed context
5. **Decoder LSTM:** Reconstructs from latent representation

**Benefits:**
- Dynamically focuses on important timesteps
- Better handles variable-length sequences
- More interpretable (can visualize attention weights)

### Federated Learning with Masked LSTM

**Process:**

1. **FL Server:** Initializes global model
2. **FL Clients:** Each facility has:
   - Local data
   - Facility-specific mask
   - Copy of global model
3. **Local Training:**
   - Train for 5 epochs with masking
   - Loss computed only on valid features
4. **Weight Upload:** Send updated weights to server
5. **Aggregation:** Server aggregates using FedMedian
6. **Distribution:** Updated model sent to all clients

**Key Advantage:** Each facility trains only on their valid sensors, but benefits from all facilities' knowledge.

### Anomaly Detection with Masking

**Detection Process:**

1. **Forward Pass:** Reconstruct input with masking
2. **Per-Feature Errors:** Calculate error for each feature
   - `feature_errors = masked_error.mean(over_time)`
3. **Overall Error:** Average over valid features only
   - `total_error = masked_error.sum() / num_valid_features`
4. **Anomaly Decision:** Compare to threshold
   - `is_anomaly = total_error > threshold`

**Benefits:**
- Can identify which specific sensor caused the anomaly
- Only considers valid sensors in error calculation
- More accurate than padding approach

### Advantages of Masking

**1. Explicit Feature Validity**
- Without masking: LSTM must learn that 0.5 = missing
- With masking: We explicitly tell LSTM which features are valid

**2. Better Gradient Flow**
- Without masking: Gradients computed for all features (including padded)
- With masking: Gradients only computed for valid features
- Result: Faster convergence, better accuracy

**3. Interpretability**
- Can analyze which features contributed to anomaly detection
- Per-feature error attribution
- Example: "Temperature error: 0.8, Radiation error: 0.1"

**4. Dynamic Sensor Addition**
- Facility adds new sensor
- Just update mask from 0.0 to 1.0 for that feature
- No retraining needed!

### Comparison: Padding vs Masking

| Aspect | Feature Padding | Explicit Masking |
|--------|----------------|------------------|
| **Complexity** | Simple | More complex |
| **Implementation** | Easy | Requires custom loss |
| **Training Speed** | Slower (computes gradients for padded features) | Faster (only valid features) |
| **Accuracy** | Good | Better |
| **Interpretability** | Limited | High (per-feature errors) |
| **Dynamic Sensors** | Requires retraining | Just update mask |
| **Memory** | Same | Same |
| **Recommended For** | MVP, simple cases | Production, complex cases |

### When to Use Masking

**Use Masking if:**
- ✅ You have many facilities with very different sensor configurations
- ✅ Sensors are frequently added/removed
- ✅ You need per-feature anomaly attribution
- ✅ You want maximum accuracy
- ✅ You have time for more complex implementation

**Use Padding if:**
- ✅ You want simplicity
- ✅ Most facilities have similar sensors
- ✅ Sensors rarely change
- ✅ You're building an MVP
- ✅ You want faster development

---

## Recommendation for Your Project

**Use Feature Padding (Solution 2)** because:

1. ✅ **Simple to implement** - Just pad with 0.5
2. ✅ **All facilities participate** - No one excluded
3. ✅ **Preserves unique sensors** - Power plants keep radiation, water treatment keeps pH
4. ✅ **Flexible** - Easy to add new facilities with different sensors
5. ✅ **LSTM learns automatically** - No manual feature selection

**Implementation Steps:**
1. Define universal feature set (18 features in your case: 10 process + 8 network)
2. Each facility specifies which sensors they have
3. Pad missing sensors with 0.5
4. Train LSTM on padded data
5. LSTM learns to ignore always-0.5 features
6. FL aggregation works normally

**For November 30 Demo:** Use **Feature Padding** (Solution 2)
- Simpler to implement
- Good enough accuracy
- Faster development

**For Production:** Consider upgrading to **Masking** (Solution 4)
- Better accuracy
- More interpretable
- More flexible for adding facilities

---

## Summary

**Problem:** Different facilities have different sensors

**Solution:** Feature padding with neutral values (0.5)

**How it works:**
1. Create universal feature set (union of all sensors)
2. Facilities with sensor: Use normalized real value
3. Facilities without sensor: Pad with 0.5 (neutral)
4. LSTM learns to ignore features that are always 0.5
5. FL aggregation combines knowledge from all facilities
6. Everyone benefits from patterns in common features

**Key insight:** LSTM automatically learns which features are informative and which are just padding!

**Result:** All facilities can participate in federated learning, regardless of sensor configuration!
