# Federated Learning with Different Facilities - Normalization Strategy

## The Challenge

Different facilities in an ICS environment have:
- **Different equipment** (different temperature ranges, pressure limits)
- **Different processes** (power plant vs water treatment)
- **Different baselines** (Facility A runs at 300°C, Facility B at 280°C)
- **Different scales** (different sensor ranges)

**Question:** How can they share the same LSTM model?

**Answer:** Through standardization and normalization!

---

## The Solution: Three-Layer Approach

### Layer 1: Standardized Feature Set

All facilities must have the **same 10 features** (even if values differ):

```
Feature 1: Temperature (°C)
Feature 2: Pressure (psi)
Feature 3: Flow rate (L/min)
Feature 4: Valve position (%)
Feature 5: Pump status (0/1)
Feature 6: Setpoint temperature (°C)
Feature 7: Setpoint pressure (psi)
Feature 8: Control signal (%)
Feature 9: Error signal (deviation from setpoint)
Feature 10: Time of day (0-1 normalized)
```

**Key Point:** The feature **types** are the same, but the **values** differ.

### Layer 2: Local Normalization (Critical!)

Each facility normalizes data to **0-1 range** using **their own** min/max values:

**Facility A (Power Plant):**
```
Temperature range: 280-320°C
Normalized: (value - 280) / (320 - 280)

Example: 300°C → (300 - 280) / 40 = 0.5
```

**Facility B (Water Treatment):**
```
Temperature range: 250-290°C
Normalized: (value - 250) / (290 - 250)

Example: 270°C → (270 - 250) / 40 = 0.5
```

**Result:** Both facilities represent "middle of normal range" as 0.5!

### Layer 3: Pattern Learning

The LSTM learns **patterns**, not absolute values:

```
Pattern: "Temperature gradually increasing over 60 seconds"

Facility A sees:
[0.5, 0.52, 0.54, 0.56, 0.58, ...] (300°C → 310°C)

Facility B sees:
[0.5, 0.52, 0.54, 0.56, 0.58, ...] (270°C → 280°C)

Model learns: "Gradual increase pattern = potential attack"
```

---

## Shared Model Architecture

All facilities use the **exact same architecture**:

```
Input: (60 timesteps, 10 features) - normalized to [0, 1]
       ↓
LSTM Layer 1: 128 units
       ↓
LSTM Layer 2: 64 units
       ↓
Latent Space: 64 dimensions
       ↓
LSTM Layer 3: 64 units
       ↓
LSTM Layer 4: 128 units
       ↓
Output: (60 timesteps, 10 features) - normalized to [0, 1]
```

**Same architecture = Compatible weights**

---

## Federated Learning Process

### Step 1: Local Training (Each Facility)

**Facility A:**
```python
data_A = load_sensor_data("facility_a")
normalized_A = normalize(data_A, min_A, max_A)  # Use Facility A's ranges
model_A.train(normalized_A)
weights_A = model_A.get_weights()
```

**Facility B:**
```python
data_B = load_sensor_data("facility_b")
normalized_B = normalize(data_B, min_B, max_B)  # Use Facility B's ranges
model_B.train(normalized_B)
weights_B = model_B.get_weights()
```

**Facility C:**
```python
data_C = load_sensor_data("facility_c")
normalized_C = normalize(data_C, min_C, max_C)  # Use Facility C's ranges
model_C.train(normalized_C)
weights_C = model_C.get_weights()
```

### Step 2: Aggregation (FL Server)

```python
# FL Server receives weights from all facilities
weights_global = aggregate([weights_A, weights_B, weights_C])

# FedMedian: Take coordinate-wise median
# This is robust to outliers and Byzantine attacks
```

### Step 3: Distribution (Back to Facilities)

```python
# Each facility receives the same global weights
model_A.set_weights(weights_global)
model_B.set_weights(weights_global)
model_C.set_weights(weights_global)

# But each facility still uses their own normalization!
```

---

## Concrete Example: Gradual Drift Attack

### Facility A (Power Plant)

**Equipment:**
- Normal temperature: 280-320°C
- Max safe temperature: 350°C

**Attack:**
- Temperature: 300°C → 380°C over 60 seconds

**Normalized Data:**
```
[0.5, 0.52, 0.54, 0.56, ..., 0.95]
(exceeds normal range of 0-1)
```

**LSTM Detection:**
- High reconstruction error
- Pattern: Gradual increase beyond normal

### Facility B (Water Treatment)

**Equipment:**
- Normal temperature: 250-290°C
- Max safe temperature: 320°C

**Attack:**
- Temperature: 270°C → 350°C over 60 seconds

**Normalized Data:**
```
[0.5, 0.52, 0.54, 0.56, ..., 0.95]
(same pattern as Facility A!)
```

**LSTM Detection:**
- High reconstruction error
- Pattern: Same gradual increase

**Key Insight:** The **pattern** is identical, even though absolute temperatures differ!

---

## What Gets Shared vs What Stays Local

### Shared (via Federated Learning):
✅ Model weights (~10 MB)
✅ Pattern knowledge ("gradual increase = attack")
✅ Behavioral anomaly detection capability
✅ Attack signatures (learned patterns)

### Stays Local (Never Shared):
❌ Raw sensor data
❌ Normalization parameters (min/max values)
❌ Facility-specific thresholds
❌ Equipment configurations
❌ Operational details

---

## Handling Different Equipment Types

### Option 1: Common Feature Set (Recommended)

All facilities must have these core sensors:
- Temperature sensor
- Pressure sensor
- Flow sensor
- Valve position
- Pump status

**If a facility doesn't have a sensor:** Use a constant value (e.g., 0.5) or interpolate.

### Option 2: Feature Padding

```python
# Facility A has 10 sensors
features_A = [temp, pressure, flow, valve, pump, ...]  # 10 features

# Facility B has only 8 sensors
features_B = [temp, pressure, flow, valve, pump, ..., 0.5, 0.5]  # Pad with neutral
```

### Option 3: Facility Clustering (Advanced)

Group similar facilities:
- **Cluster 1:** Power plants (high temperature)
- **Cluster 2:** Water treatment (low temperature)
- **Cluster 3:** Manufacturing (medium temperature)

Each cluster has its own federated model.

---

## Why This Works

### 1. Relative Patterns, Not Absolute Values

The model learns:
- "Temperature increasing faster than normal"
- "Pressure oscillating unexpectedly"
- "Flow rate not correlated with valve position"

These patterns are **universal** across facilities!

### 2. Normalization Abstracts Differences

```
Facility A: 300°C is "normal" → normalized to 0.5
Facility B: 270°C is "normal" → normalized to 0.5
Facility C: 310°C is "normal" → normalized to 0.5

All facilities: 0.5 = "middle of normal range"
```

### 3. Attack Patterns Are Similar

Attacks have similar characteristics:
- Sudden spikes: 0.5 → 0.9 in 1 second
- Gradual drifts: 0.5 → 0.8 over 60 seconds
- Oscillations: 0.4 ↔ 0.6 every 2 seconds

These patterns transfer across facilities!

---

## Potential Issues and Solutions

### Issue 1: Very Different Processes

**Problem:** Power plant vs water treatment have fundamentally different processes

**Solution:**
- Use facility clustering
- Or use facility-specific fine-tuning after FL

```python
# After FL round
global_weights = receive_from_fl_server()
model.set_weights(global_weights)

# Fine-tune on local data (1-2 epochs)
model.train(local_data, epochs=2)
```

### Issue 2: Different Sensor Counts

**Problem:** Facility A has 10 sensors, Facility B has 8

**Solution:**
- Pad missing features with neutral values (0.5)
- Or use a subset of common features (8 features for all)

### Issue 3: Different Sampling Rates

**Problem:** Facility A samples at 1 Hz, Facility B at 0.5 Hz

**Solution:**
- Standardize to 1 Hz
- Interpolate or downsample as needed

```python
# Facility B (0.5 Hz) → Upsample to 1 Hz
data_B_upsampled = interpolate(data_B, target_rate=1.0)
```

---

## Implementation Details

### Normalization Storage

Each facility stores its own normalization parameters:

```python
# Facility A
normalization_params_A = {
    "temperature": {"min": 280, "max": 320},
    "pressure": {"min": 100, "max": 150},
    "flow_rate": {"min": 50, "max": 100},
    "valve_position": {"min": 0, "max": 100},
    "pump_status": {"min": 0, "max": 1},
    "setpoint_temp": {"min": 280, "max": 320},
    "setpoint_pressure": {"min": 100, "max": 150},
    "control_signal": {"min": 0, "max": 100},
    "error_signal": {"min": -20, "max": 20},
    "time_of_day": {"min": 0, "max": 1}
}

# Facility B
normalization_params_B = {
    "temperature": {"min": 250, "max": 290},
    "pressure": {"min": 90, "max": 140},
    "flow_rate": {"min": 40, "max": 90},
    # ... other features with different ranges
}
```

### Training Code Example

```python
class FacilityLSTM:
    def __init__(self, normalization_params):
        self.norm_params = normalization_params
        self.model = build_lstm_autoencoder()
    
    def normalize(self, data):
        """Normalize data using facility-specific parameters"""
        normalized = {}
        for feature, values in data.items():
            min_val = self.norm_params[feature]["min"]
            max_val = self.norm_params[feature]["max"]
            normalized[feature] = (values - min_val) / (max_val - min_val)
        return normalized
    
    def denormalize(self, normalized_data):
        """Convert normalized data back to original scale"""
        denormalized = {}
        for feature, values in normalized_data.items():
            min_val = self.norm_params[feature]["min"]
            max_val = self.norm_params[feature]["max"]
            denormalized[feature] = values * (max_val - min_val) + min_val
        return denormalized
    
    def train_local(self, data, epochs=5):
        """Train model on local data"""
        # Normalize using facility-specific parameters
        normalized_data = self.normalize(data)
        
        # Train model
        self.model.fit(normalized_data, epochs=epochs)
        
        # Return weights for FL
        return self.model.get_weights()
    
    def update_from_fl(self, global_weights):
        """Receive global weights from FL server"""
        self.model.set_weights(global_weights)
    
    def detect_anomaly(self, data):
        """Detect anomalies in new data"""
        # Normalize
        normalized = self.normalize(data)
        
        # Reconstruct
        reconstructed = self.model.predict(normalized)
        
        # Calculate error
        error = mean_squared_error(normalized, reconstructed)
        
        # Compare to threshold
        return error > self.threshold
```

### FL Server Code Example

```python
class FederatedLearningServer:
    def __init__(self):
        self.global_model = build_lstm_autoencoder()
        self.clients = []
    
    def aggregate_weights(self, client_weights):
        """Aggregate weights using FedMedian"""
        # Stack all weights
        stacked = [np.array(w) for w in zip(*client_weights)]
        
        # Take coordinate-wise median
        aggregated = [np.median(w, axis=0) for w in stacked]
        
        return aggregated
    
    def run_fl_round(self):
        """Execute one FL round"""
        # 1. Send global model to clients
        global_weights = self.global_model.get_weights()
        
        # 2. Clients train locally
        client_weights = []
        for client in self.clients:
            weights = client.train_local(epochs=5)
            client_weights.append(weights)
        
        # 3. Aggregate weights
        new_global_weights = self.aggregate_weights(client_weights)
        
        # 4. Update global model
        self.global_model.set_weights(new_global_weights)
        
        # 5. Distribute to clients
        for client in self.clients:
            client.update_from_fl(new_global_weights)
```

---

## Visualization of the Process

### Data Flow Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                    Facility A                               │
│                                                             │
│  Raw Data: [300°C, 305°C, 310°C, ...]                     │
│      ↓                                                      │
│  Normalize (280-320°C range)                               │
│      ↓                                                      │
│  Normalized: [0.5, 0.625, 0.75, ...]                      │
│      ↓                                                      │
│  Train LSTM (5 epochs)                                     │
│      ↓                                                      │
│  Weights_A → FL Server                                     │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                    Facility B                               │
│                                                             │
│  Raw Data: [270°C, 275°C, 280°C, ...]                     │
│      ↓                                                      │
│  Normalize (250-290°C range)                               │
│      ↓                                                      │
│  Normalized: [0.5, 0.625, 0.75, ...]                      │
│      ↓                                                      │
│  Train LSTM (5 epochs)                                     │
│      ↓                                                      │
│  Weights_B → FL Server                                     │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                    FL Server                                │
│                                                             │
│  Receive: Weights_A, Weights_B, Weights_C                 │
│      ↓                                                      │
│  Aggregate (FedMedian)                                     │
│      ↓                                                      │
│  Global_Weights                                            │
│      ↓                                                      │
│  Distribute → All Facilities                               │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│              All Facilities (After FL)                      │
│                                                             │
│  Same Global Model                                         │
│  Different Normalization Parameters                        │
│  Can detect same attack patterns                           │
└─────────────────────────────────────────────────────────────┘
```

---

## Summary

### How It Works:

1. **Standardized Features** - All facilities use same 10 features
2. **Local Normalization** - Each facility normalizes to [0, 1] using their own ranges
3. **Pattern Learning** - Model learns relative patterns, not absolute values
4. **Weight Sharing** - Only model weights are shared, not data
5. **Local Inference** - Each facility uses global model with local normalization

### Key Insight:

**The model doesn't learn "300°C is normal"**

**It learns "0.5 is normal, and gradual increase to 0.9 is an attack"**

This abstraction allows the same model to work across different facilities!

### Benefits:

✅ **Privacy-Preserving** - No raw data shared
✅ **Collaborative Learning** - All facilities benefit from each other's experience
✅ **Facility-Specific** - Each facility maintains its own operational parameters
✅ **Scalable** - Easy to add new facilities
✅ **Robust** - FedMedian aggregation handles outliers

---

## Learning Resources

### Federated Learning Fundamentals

**1. Original Federated Learning Paper**
- Title: "Communication-Efficient Learning of Deep Networks from Decentralized Data"
- Authors: McMahan et al. (Google)
- Link: https://arxiv.org/abs/1602.05629
- **Why Read:** Introduces FedAvg algorithm, foundational concepts

**2. Federated Learning: Challenges, Methods, and Future Directions**
- Authors: Li et al.
- Link: https://arxiv.org/abs/1908.07873
- **Why Read:** Comprehensive survey, covers challenges like non-IID data

**3. Flower Framework Documentation**
- Link: https://flower.dev/docs/
- **Why Read:** Practical implementation guide for the framework we're using

**4. TensorFlow Federated Tutorial**
- Link: https://www.tensorflow.org/federated/tutorials/federated_learning_for_image_classification
- **Why Read:** Hands-on tutorial with code examples

### Differential Privacy

**5. The Algorithmic Foundations of Differential Privacy**
- Authors: Dwork and Roth
- Link: https://www.cis.upenn.edu/~aaroth/Papers/privacybook.pdf
- **Why Read:** Comprehensive textbook on differential privacy theory

**6. Opacus Documentation (PyTorch DP)**
- Link: https://opacus.ai/
- **Why Read:** Practical guide for implementing DP in PyTorch

**7. Deep Learning with Differential Privacy**
- Authors: Abadi et al. (Google)
- Link: https://arxiv.org/abs/1607.00133
- **Why Read:** Introduces DP-SGD algorithm we're using

### LSTM Autoencoders

**8. LSTM-based Encoder-Decoder for Multi-sensor Anomaly Detection**
- Authors: Malhotra et al.
- Link: https://arxiv.org/abs/1607.00148
- **Why Read:** Directly relevant to our use case

**9. Understanding LSTM Networks**
- Author: Christopher Olah
- Link: https://colah.github.io/posts/2015-08-Understanding-LSTMs/
- **Why Read:** Best visual explanation of LSTM architecture

**10. Autoencoders for Anomaly Detection**
- Link: https://www.deeplearning.ai/ai-notes/anomaly-detection/
- **Why Read:** Clear explanation of autoencoder-based anomaly detection

### ICS Security and Anomaly Detection

**11. A Survey on ICS Security**
- Authors: Stouffer et al. (NIST)
- Link: https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-82r2.pdf
- **Why Read:** NIST guidelines for ICS security

**12. Machine Learning for ICS Security**
- Authors: Goh et al.
- Link: https://ieeexplore.ieee.org/document/7469060
- **Why Read:** Discusses ML approaches for ICS, includes SWaT dataset

**13. MITRE ATT&CK for ICS**
- Link: https://attack.mitre.org/matrices/ics/
- **Why Read:** Essential for understanding ICS attack techniques

### Federated Learning for ICS (Recent Research)

**14. Federated Learning for Industrial IoT**
- Authors: Nguyen et al.
- Link: https://arxiv.org/abs/2106.07976
- **Why Read:** Discusses FL challenges specific to industrial environments

**15. Privacy-Preserving Collaborative Learning for ICS**
- Authors: Zhang et al.
- Link: https://ieeexplore.ieee.org/document/9174988
- **Why Read:** Addresses privacy concerns in ICS collaborative learning

**16. Federated Learning in Critical Infrastructure**
- Authors: Liu et al.
- Link: https://arxiv.org/abs/2012.09936
- **Why Read:** Discusses FL deployment in critical infrastructure

### Data Normalization and Preprocessing

**17. Feature Scaling and Normalization**
- Link: https://scikit-learn.org/stable/modules/preprocessing.html
- **Why Read:** Practical guide to normalization techniques

**18. Handling Non-IID Data in Federated Learning**
- Authors: Zhao et al.
- Link: https://arxiv.org/abs/1806.00582
- **Why Read:** Addresses the challenge of different data distributions

### Video Courses

**19. Federated Learning Course (Coursera)**
- Link: https://www.coursera.org/learn/federated-learning
- **Why Watch:** Structured course with hands-on exercises

**20. Deep Learning Specialization (Coursera)**
- Link: https://www.coursera.org/specializations/deep-learning
- **Why Watch:** Covers LSTM and autoencoders in depth

**21. Privacy-Preserving Machine Learning (YouTube)**
- Link: https://www.youtube.com/playlist?list=PLtmWHNX-gukIU6V33Bc8eP8OD41I4GywR
- **Why Watch:** OpenMined tutorials on privacy-preserving ML

### GitHub Repositories

**22. Flower (Federated Learning Framework)**
- Link: https://github.com/adap/flower
- **Why Explore:** Production-ready FL framework with examples

**23. PySyft (Privacy-Preserving ML)**
- Link: https://github.com/OpenMined/PySyft
- **Why Explore:** Tools for encrypted, privacy-preserving ML

**24. Federated Learning Examples**
- Link: https://github.com/tensorflow/federated
- **Why Explore:** TensorFlow Federated examples and tutorials

**25. ICS Security Datasets**
- Link: https://github.com/icsdataset
- **Why Explore:** Collection of ICS security datasets

### Blogs and Articles

**26. Google AI Blog - Federated Learning**
- Link: https://ai.googleblog.com/2017/04/federated-learning-collaborative.html
- **Why Read:** Real-world FL deployment insights

**27. Towards Data Science - FL Articles**
- Link: https://towardsdatascience.com/tagged/federated-learning
- **Why Read:** Practical tutorials and case studies

**28. OpenMined Blog**
- Link: https://blog.openmined.org/
- **Why Read:** Privacy-preserving ML techniques and tools

### Research Papers - Advanced Topics

**29. Byzantine-Robust Federated Learning**
- Authors: Blanchard et al.
- Link: https://arxiv.org/abs/1703.02757
- **Why Read:** Explains FedMedian and Byzantine-robust aggregation

**30. Personalized Federated Learning**
- Authors: Fallah et al.
- Link: https://arxiv.org/abs/2003.13461
- **Why Read:** Discusses facility-specific fine-tuning after FL

### Tools and Libraries

**31. PyTorch Documentation**
- Link: https://pytorch.org/docs/stable/index.html
- **Why Read:** Essential for implementing LSTM models

**32. Scikit-learn Documentation**
- Link: https://scikit-learn.org/stable/
- **Why Read:** For Isolation Forest and preprocessing

**33. Neo4j Graph Database**
- Link: https://neo4j.com/docs/
- **Why Read:** For implementing GNN attack prediction

### Community and Forums

**34. Federated Learning Subreddit**
- Link: https://www.reddit.com/r/FederatedLearning/
- **Why Join:** Community discussions and Q&A

**35. OpenMined Slack**
- Link: https://openmined.slack.com/
- **Why Join:** Active community for privacy-preserving ML

**36. ICS Security Forum**
- Link: https://www.reddit.com/r/ICS_Security/
- **Why Join:** ICS security discussions and news

---

## Recommended Learning Path

### Week 1: Fundamentals
1. Read "Communication-Efficient Learning" paper (#1)
2. Watch Deep Learning Specialization - LSTM module (#20)
3. Read "Understanding LSTM Networks" (#9)

### Week 2: Federated Learning
4. Complete Flower tutorials (#3)
5. Read "Federated Learning: Challenges" survey (#2)
6. Explore Flower GitHub examples (#22)

### Week 3: Privacy
7. Read "Deep Learning with Differential Privacy" (#7)
8. Study Opacus documentation (#6)
9. Watch Privacy-Preserving ML videos (#21)

### Week 4: ICS Security
10. Read NIST ICS Security guidelines (#11)
11. Study MITRE ATT&CK for ICS (#13)
12. Read "Machine Learning for ICS Security" (#12)

### Week 5: Implementation
13. Implement LSTM autoencoder for anomaly detection
14. Add Flower for federated learning
15. Integrate Opacus for differential privacy

### Week 6: Advanced Topics
16. Read "Byzantine-Robust Federated Learning" (#29)
17. Study "Handling Non-IID Data" (#18)
18. Explore personalized FL (#30)

---

## Quick Reference

### Key Concepts to Master

1. **LSTM Autoencoders** - Sequence modeling and reconstruction
2. **Federated Learning** - Distributed training without data sharing
3. **Differential Privacy** - Mathematical privacy guarantees
4. **Data Normalization** - Min-max scaling, standardization
5. **Non-IID Data** - Handling different data distributions
6. **Byzantine Robustness** - Defending against malicious clients
7. **ICS Protocols** - Modbus, DNP3, OPC-UA
8. **MITRE ATT&CK** - ICS attack techniques and tactics

### Essential Libraries

- **PyTorch** - Deep learning framework
- **Flower** - Federated learning framework
- **Opacus** - Differential privacy for PyTorch
- **Scikit-learn** - Machine learning utilities
- **NumPy/Pandas** - Data manipulation
- **Neo4j** - Graph database for attack prediction

---

**Document Version:** 1.0  
**Last Updated:** October 14, 2025  
**Status:** Ready for Learning
