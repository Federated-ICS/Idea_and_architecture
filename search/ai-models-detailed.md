# AI Models - Detailed Explanation

## Overview

This document provides an in-depth explanation of all four detection models used in the Federated ICS Threat Correlation Engine, including their architecture, training process, federated learning status, and role in the system.

---

## Model Summary

| Model | Type | Federated? | Purpose | Detection Time |
|-------|------|-----------|---------|----------------|
| LSTM Autoencoder | Deep Learning | ✅ YES | Behavioral anomalies | < 30 seconds |
| Isolation Forest | Machine Learning | ❌ NO | Point anomalies | < 5 seconds |
| Physics Rules | Rule-Based | ❌ NO | Safety violations | < 1 second |
| GNN (GAT) | Deep Learning | ✅ YES | Attack prediction | Real-time |

---

## 1. LSTM Autoencoder

### What It Is

**Type:** Deep Learning - Recurrent Neural Network (RNN)

**Architecture:** Encoder-Decoder with LSTM layers

**Purpose:** Detect behavioral anomalies by learning normal temporal patterns

**Federated:** ✅ YES - This is the primary federated model

### How It Works

**Concept:**
- Learns what "normal" sensor behavior looks like
- Tries to reconstruct input sequences
- High reconstruction error = anomaly

**Architecture Details:**

```
Input: 60 timesteps × 10 features
       ↓
┌──────────────────────────────────┐
│         ENCODER                  │
│                                  │
│  LSTM Layer 1: 128 units         │
│  (learns temporal patterns)      │
│         ↓                        │
│  LSTM Layer 2: 64 units          │
│  (compresses to latent space)    │
│         ↓                        │
│  Latent Representation: 64-dim   │
└──────────────────────────────────┘
       ↓
┌──────────────────────────────────┐
│         DECODER                  │
│                                  │
│  LSTM Layer 1: 64 units          │
│  (expands from latent space)     │
│         ↓                        │
│  LSTM Layer 2: 128 units         │
│  (reconstructs sequence)         │
│         ↓                        │
│  Dense Layer: 10 units           │
│  (output layer)                  │
└──────────────────────────────────┘
       ↓
Output: 60 timesteps × 10 features
       ↓
Reconstruction Error (MSE)
```

### Input/Output

**Input:**
- 60 timesteps (60 seconds at 1 Hz)
- 10 features per timestep:
  1. Temperature (°C)
  2. Pressure (psi)
  3. Flow rate (L/min)
  4. Valve position (%)
  5. Pump status (0/1)
  6. Setpoint temperature (°C)
  7. Setpoint pressure (psi)
  8. Control signal (%)
  9. Error signal (deviation)
  10. Time of day (normalized)

**Output:**
- Reconstruction error (MSE - Mean Squared Error)
- Threshold: 95th percentile of training errors
- If error > threshold → Anomaly detected

**Example:**
```
Normal operation:
Input: [300, 301, 299, 300, 302, ...]
Reconstructed: [300.1, 300.9, 299.2, 300.1, 301.8, ...]
Error: 0.15 (LOW - Normal)

Attack (gradual drift):
Input: [300, 302, 304, 306, 308, ...]
Reconstructed: [300.1, 300.9, 301.2, 301.5, 301.8, ...]
Error: 8.45 (HIGH - Anomaly!)
```

### Training Process

**Phase 1: Local Training (Each Facility)**
1. Collect 7 days of normal operation data
2. Create sliding windows (60 timesteps)
3. Train autoencoder to minimize reconstruction error
4. Train for 5 epochs locally
5. Apply differential privacy (add noise to gradients)

**Phase 2: Federated Learning**
1. Each facility sends model weights to FL server
2. FL server aggregates weights using FedMedian
3. FL server sends updated global model back
4. Each facility updates local model
5. Repeat every 6 hours or when attack detected

**Training Data:**
- Only normal operation data
- No attack data in training
- Minimum: 24 hours
- Recommended: 7 days

**Hyperparameters:**
- Learning rate: 0.001
- Batch size: 32
- Optimizer: Adam
- Loss function: Mean Squared Error (MSE)
- Epochs per FL round: 5

### Why Federated?

**Benefits:**
- Facility A learns attack pattern
- Shares knowledge with B and C
- All facilities protected within 6 hours
- No raw data sharing (privacy preserved)

**Example:**
```
Before FL:
- Facility A: Detects gradual drift (trained on A's data)
- Facility B: Misses gradual drift (never seen it)
- Facility C: Misses gradual drift (never seen it)

After FL Round:
- Facility A: Still detects (original knowledge)
- Facility B: Now detects! (learned from A)
- Facility C: Now detects! (learned from A)
```

### Detection Latency

**Time to Detect:**
- Needs 60 seconds of data (sliding window)
- Inference time: < 1 second
- **Total: ~30 seconds** (half window + inference)

**Why 30 seconds?**
- Anomaly becomes clear after ~30 timesteps
- Don't need full 60-second window
- Early detection possible

### Strengths

✅ Detects subtle behavioral changes
✅ Learns complex temporal patterns
✅ Adapts to normal variations
✅ Benefits from federated learning
✅ Low false positive rate (after training)

### Weaknesses

❌ Requires training data (7 days)
❌ Slower detection (30 seconds)
❌ Computationally expensive
❌ May miss sudden spikes (needs time window)
❌ Black box (hard to interpret)

### Best For

- Gradual drift attacks
- Behavioral anomalies
- Stealthy attacks
- Pattern changes over time
- Multi-sensor correlation

---

## 2. Isolation Forest

### What It Is

**Type:** Machine Learning - Ensemble Method

**Algorithm:** Tree-based anomaly detection

**Purpose:** Detect point anomalies (sudden outliers)

**Federated:** ❌ NO - Local model per facility

### How It Works

**Concept:**
- Anomalies are "few and different"
- Easier to isolate anomalies than normal points
- Uses random decision trees

**Algorithm:**
1. Build random trees by randomly selecting features and split values
2. Anomalies require fewer splits to isolate
3. Calculate anomaly score based on path length
4. Short path = anomaly, long path = normal

**Visual Example:**
```
Normal points cluster together:
    ●●●●●
    ●●●●●  ← Many splits needed to isolate
    ●●●●●

Anomaly stands alone:
         ★  ← Few splits needed to isolate
```

### Input/Output

**Input:**
- Single data point (no time window)
- 10 features (same as LSTM):
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

**Output:**
- Anomaly score: -1 to 1
  - Score > 0.6 → Anomaly
  - Score < 0.6 → Normal

**Example:**
```
Normal operation:
Temperature: 300°C, Pressure: 120 psi, Flow: 75 L/min
Anomaly Score: 0.15 (Normal)

Sudden spike attack:
Temperature: 380°C, Pressure: 120 psi, Flow: 75 L/min
Anomaly Score: 0.95 (Anomaly!)
```

### Training Process

**Training (Each Facility Independently):**
1. Collect normal operation data (10,000+ samples)
2. Fit Isolation Forest on normal data only
3. Set contamination parameter (1% expected anomalies)
4. No federated learning - each facility trains locally

**Why No Federated Learning?**
- Simple model (100 trees, ~1 MB)
- Fast to train (< 1 minute)
- Facility-specific baselines work better
- Each facility has unique "normal" patterns

**Hyperparameters:**
- n_estimators: 100 (number of trees)
- max_samples: 256 (samples per tree)
- contamination: 0.01 (1% expected anomalies)
- max_features: 1.0 (use all features)

### Detection Latency

**Time to Detect:**
- No time window needed
- Inference time: < 100 ms
- **Total: < 5 seconds** (including data retrieval)

**Why So Fast?**
- Single data point analysis
- Simple tree traversal
- No complex computations

### Strengths

✅ Very fast detection (< 5 seconds)
✅ Detects sudden spikes immediately
✅ No time window needed
✅ Easy to train and deploy
✅ Interpretable (tree-based)
✅ Low computational cost

### Weaknesses

❌ Misses gradual changes
❌ Doesn't learn temporal patterns
❌ Facility-specific (no collaborative learning)
❌ Requires retraining for new normal patterns
❌ May have higher false positive rate

### Best For

- Sudden spike attacks
- Outlier detection
- Immediate response needed
- Single-point anomalies
- Fast screening

---

## 3. Physics Rules Engine

### What It Is

**Type:** Rule-Based System (Not Machine Learning)

**Algorithm:** Threshold-based logic

**Purpose:** Validate against known safety constraints

**Federated:** ❌ NO - Facility-specific rules

### How It Works

**Concept:**
- Industrial processes have known safety limits
- Violations indicate problems or attacks
- Simple if-then rules

**Rule Types:**

**1. Range Rules:**
```
IF temperature > 350°C THEN CRITICAL_ALERT
IF temperature < 250°C THEN WARNING_ALERT
IF pressure > 160 psi THEN CRITICAL_ALERT
IF pressure < 90 psi THEN WARNING_ALERT
```

**2. Rate-of-Change Rules:**
```
IF (temperature_now - temperature_1min_ago) > 10°C THEN WARNING_ALERT
IF (pressure_now - pressure_1sec_ago) > 5 psi THEN CRITICAL_ALERT
```

**3. Dependency Rules:**
```
IF valve_open == TRUE AND pump_running == FALSE THEN CRITICAL_ALERT
IF temperature > setpoint + 20°C THEN WARNING_ALERT
```

**4. Correlation Rules:**
```
IF flow_rate > 0 AND valve_position == 0 THEN CRITICAL_ALERT
IF pump_running == TRUE AND flow_rate == 0 THEN CRITICAL_ALERT
```

### Input/Output

**Input:**
- Real-time sensor values
- Previous values (for rate-of-change)
- Equipment states

**Output:**
- Alert severity: CRITICAL, WARNING, INFO
- Rule ID that triggered
- Violated constraint details

**Example:**
```
Rule: TEMP-MAX
Condition: temperature > 350°C
Current Value: 380°C
Severity: CRITICAL
Message: "Temperature exceeds safe limit (380°C > 350°C)"
```

### Configuration

**Rule Definition Format:**
```yaml
rules:
  - id: "TEMP-MAX"
    sensor: "reactor.temperature"
    condition: "value > 350"
    severity: "CRITICAL"
    message: "Temperature exceeds safe limit"
    action: "shutdown_reactor"
  
  - id: "PRESSURE-RATE"
    sensor: "reactor.pressure"
    condition: "rate_of_change > 10"
    window: "1_minute"
    severity: "WARNING"
    message: "Pressure rising too quickly"
  
  - id: "PUMP-VALVE-DEPENDENCY"
    sensors: ["valve.inlet", "pump.main"]
    condition: "valve.inlet == OPEN AND pump.main == OFF"
    severity: "CRITICAL"
    message: "Valve open without pump running"
```

### Why No Federated Learning?

**Reasons:**
1. **Not Machine Learning** - Rule-based, no learning
2. **Equipment-Specific** - Each facility has different limits
3. **Domain Knowledge** - Based on engineering specs
4. **Regulatory** - May be required by law
5. **Deterministic** - Same input = same output

**Example of Facility Differences:**
```
Facility A (Newer Equipment):
- Max temperature: 350°C
- Max pressure: 160 psi

Facility B (Older Equipment):
- Max temperature: 340°C
- Max pressure: 150 psi

Facility C (Different Process):
- Max temperature: 330°C
- Max pressure: 140 psi
```

### Detection Latency

**Time to Detect:**
- Rule evaluation: < 1 ms
- **Total: < 1 second** (including data retrieval)

**Why So Fast?**
- Simple comparisons
- No complex computations
- Direct threshold checks

### Strengths

✅ Extremely fast (< 1 second)
✅ 100% explainable (clear rules)
✅ Based on domain knowledge
✅ No training required
✅ Deterministic and reliable
✅ Regulatory compliance
✅ Zero false negatives (if rules are correct)

### Weaknesses

❌ Requires domain expertise
❌ Doesn't learn or adapt
❌ Can't detect unknown attacks
❌ May have false positives (tight thresholds)
❌ Maintenance overhead (rule updates)
❌ Facility-specific (no sharing)

### Best For

- Known safety violations
- Regulatory compliance
- Immediate critical alerts
- Deterministic detection
- Backup detection layer

---

## 4. Graph Neural Network (GAT)

### What It Is

**Type:** Deep Learning - Graph Neural Network

**Architecture:** Graph Attention Network (GAT)

**Purpose:** Predict next attack techniques (proactive defense)

**Federated:** ✅ YES - Learns attack patterns collaboratively

### How It Works

**Concept:**
- Attacks follow patterns (kill chains)
- MITRE ATT&CK documents these patterns
- GNN learns which techniques follow others
- Predicts attacker's next move

**Graph Structure:**
```
Nodes: MITRE ATT&CK techniques
Edges: "LEADS_TO" relationships

Example:
T0846 (Discovery) --67%--> T0843 (Program Download)
T0846 (Discovery) --23%--> T0800 (Lateral Movement)
T0800 (Lateral Movement) --72%--> T0843 (Program Download)
```

**Architecture Details:**

```
Input: Current technique + graph structure
       ↓
┌──────────────────────────────────┐
│    Graph Attention Layer 1       │
│                                  │
│  - 4 attention heads             │
│  - 64 features per head          │
│  - Learns which neighbors matter │
│         ↓                        │
│    Dropout (0.6)                 │
│         ↓                        │
│    Graph Attention Layer 2       │
│                                  │
│  - 4 attention heads             │
│  - 32 features per head          │
│  - Refines predictions           │
│         ↓                        │
│    Dense Layer                   │
│  - Output: 81 techniques         │
│  - Softmax activation            │
└──────────────────────────────────┘
       ↓
Output: Probability distribution
       ↓
Top-3 Predictions with probabilities
```

### Input/Output

**Input:**
- Current attack technique (e.g., T0846)
- Graph structure (MITRE ATT&CK)
- Technique features:
  - One-hot encoding of technique ID
  - Tactic embedding
  - Historical frequency
  - Time since last seen
  - Target asset type

**Output:**
- Probability distribution over 81 ICS techniques
- Top-3 most likely next techniques
- Confidence scores
- Estimated timeframe

**Example:**
```
Current: T0846 (Remote System Discovery)

Predictions:
1. T0843 (Program Download) - 67% probability
   Timeframe: 15-60 minutes
   Target: PLC-REACTOR-01

2. T0800 (Lateral Movement) - 23% probability
   Timeframe: 30-90 minutes
   Target: Network segment B

3. T0858 (Change Operating Mode) - 10% probability
   Timeframe: 60-120 minutes
   Target: HMI console
```

### Training Process

**Phase 1: Graph Construction**
1. Download MITRE ATT&CK ICS JSON
2. Parse techniques and relationships
3. Load into Neo4j graph database
4. Extract attack sequences from historical data

**Phase 2: Local Training (Each Facility)**
1. Collect attack sequences from facility logs
2. Create training pairs: (current_technique, next_technique)
3. Train GNN for 10 epochs locally
4. Apply differential privacy

**Phase 3: Federated Learning**
1. Each facility sends GNN weights to FL server
2. FL server aggregates using FedMedian
3. Updated model distributed back
4. Repeat after new attacks observed

**Training Data:**
- Historical attack sequences
- MITRE documented attack chains
- Synthetic attack paths
- Incident reports

**Hyperparameters:**
- Learning rate: 0.005
- Batch size: 16
- Optimizer: Adam
- Loss function: Cross-entropy
- Dropout: 0.6
- Attention heads: 4

### Why Federated?

**Benefits:**
- Facility A sees new attack chain
- Shares pattern with B and C
- All facilities can now predict this chain
- Collective intelligence improves accuracy

**Example:**
```
Before FL:
- Facility A: Sees T0846 → T0843 chain (can predict)
- Facility B: Never seen this chain (can't predict)
- Facility C: Never seen this chain (can't predict)

After FL Round:
- Facility A: Still predicts (original knowledge)
- Facility B: Now predicts! (learned from A)
- Facility C: Now predicts! (learned from A)

New attack on Facility B:
- Attacker performs T0846
- GNN predicts T0843 next (67% probability)
- Proactive defense: Block program downloads
- Attack prevented!
```

### Detection Latency

**Time to Predict:**
- Graph query: < 100 ms
- GNN inference: < 500 ms
- **Total: < 1 second** (real-time)

**When It Runs:**
- After each alert detected
- Extracts MITRE technique from alert
- Generates prediction immediately

### Strengths

✅ Proactive defense (predicts next move)
✅ Learns from all facilities (federated)
✅ Based on MITRE ATT&CK (industry standard)
✅ Provides actionable intelligence
✅ Improves over time
✅ Explainable (shows attack path)

### Weaknesses

❌ Requires labeled attack data
❌ Complex to train and deploy
❌ Depends on MITRE technique mapping
❌ May not predict novel attack chains
❌ Requires graph database (Neo4j)

### Best For

- Multi-stage attacks
- Attack prediction
- Proactive defense
- Threat intelligence
- Attack path analysis

---

## Model Comparison

### Detection Speed

```
Physics Rules:    < 1 second   ████
Isolation Forest: < 5 seconds  ████████
LSTM:            < 30 seconds  ████████████████████████████
GNN:             < 1 second    ████ (prediction, not detection)
```

### Accuracy (False Positive Rate)

```
Physics Rules:    Low (if rules correct)
Isolation Forest: Medium (may flag normal outliers)
LSTM:            Low (learns normal patterns)
GNN:             Medium (depends on training data)
```

### Computational Cost

```
Physics Rules:    Very Low   ██
Isolation Forest: Low        ████
LSTM:            High        ████████████████
GNN:             High        ████████████████
```

### Training Data Required

```
Physics Rules:    None (domain knowledge)
Isolation Forest: 10,000+ samples
LSTM:            7+ days of data
GNN:             Attack sequences (100+)
```

---

## How They Work Together

### Multi-Layered Defense

```
Attack Occurs
       ↓
┌─────────────────────────────────────┐
│  Layer 1: Physics Rules (< 1 sec)  │ ← Immediate safety check
└─────────────────────────────────────┘
       ↓
┌─────────────────────────────────────┐
│  Layer 2: Isolation Forest (< 5s)  │ ← Outlier detection
└─────────────────────────────────────┘
       ↓
┌─────────────────────────────────────┐
│  Layer 3: LSTM (< 30 sec)          │ ← Behavioral analysis
└─────────────────────────────────────┘
       ↓
┌─────────────────────────────────────┐
│  Layer 4: GNN (< 1 sec)            │ ← Predict next step
└─────────────────────────────────────┘
       ↓
┌─────────────────────────────────────┐
│     Correlation Engine              │ ← Combine all signals
└─────────────────────────────────────┘
       ↓
High Confidence Alert
```

### Correlation Logic

```python
def correlate_detections(detections):
    sources = set([d.source for d in detections])
    num_sources = len(sources)
    
    # Calculate confidence
    base_confidence = max([d.confidence for d in detections])
    confidence = min(base_confidence * (1 + 0.2 * num_sources), 1.0)
    
    # Determine severity
    if num_sources >= 3:
        severity = "CRITICAL"  # All methods agree
    elif num_sources >= 2:
        severity = "HIGH"      # Two methods agree
    else:
        severity = detections[0].severity
    
    return Alert(confidence=confidence, severity=severity, sources=sources)
```

### Example Attack Detection

**Scenario: Gradual Drift Attack**

```
Time: 0:00 - Attack starts (300°C → 420°C over 60 seconds)

Time: 0:01 - Physics Rules: No alert (still within range)
Time: 0:05 - Isolation Forest: No alert (gradual change)
Time: 0:30 - LSTM: ALERT! (behavioral anomaly detected)
Time: 0:30 - GNN: Predicts next step (T0843 - 67% probability)
Time: 0:45 - Physics Rules: ALERT! (temperature > 350°C)
Time: 0:45 - Isolation Forest: ALERT! (now an outlier)

Correlation Engine:
- 3 sources detected: LSTM, Physics, IF
- Confidence: 0.95 (very high)
- Severity: CRITICAL
- GNN prediction: T0843 next (proactive defense)
```

---

## Federated Learning Details

### Which Models Are Federated?

**Federated (2 models):**
1. ✅ LSTM Autoencoder
2. ✅ GNN (Attack Prediction)

**Local (2 models):**
1. ❌ Isolation Forest
2. ❌ Physics Rules

### FL Process

```
Step 1: Local Training
┌──────────┐  ┌──────────┐  ┌──────────┐
│Facility A│  │Facility B│  │Facility C│
│          │  │          │  │          │
│ Train    │  │ Train    │  │ Train    │
│ LSTM+GNN │  │ LSTM+GNN │  │ LSTM+GNN │
│ 5 epochs │  │ 5 epochs │  │ 5 epochs │
└──────────┘  └──────────┘  └──────────┘
     ↓              ↓              ↓
     └──────────────┴──────────────┘
                    ↓
Step 2: Send Weights to FL Server
            ┌──────────────┐
            │  FL Server   │
            │              │
            │  Aggregate   │
            │  (FedMedian) │
            └──────────────┘
                    ↓
Step 3: Distribute Updated Model
     ┌──────────────┴──────────────┐
     ↓              ↓              ↓
┌──────────┐  ┌──────────┐  ┌──────────┐
│Facility A│  │Facility B│  │Facility C│
│          │  │          │  │          │
│ Update   │  │ Update   │  │ Update   │
│ LSTM+GNN │  │ LSTM+GNN │  │ LSTM+GNN │
└──────────┘  └──────────┘  └──────────┘
```

### Privacy Preservation

**Differential Privacy (Opacus):**
- Adds noise to gradients during training
- Epsilon (ε): 2.0
- Delta (δ): 10⁻⁵
- Prevents reverse-engineering of training data

**What's Shared:**
- Model weights (~10 MB)
- Aggregated gradients

**What's NOT Shared:**
- Raw sensor data
- Facility-specific information
- Attack details
- Equipment configurations

---

## Summary

### Model Roles

**LSTM Autoencoder:**
- Primary detection model
- Learns behavioral patterns
- Federated learning
- Detects gradual attacks

**Isolation Forest:**
- Fast outlier detection
- Immediate response
- Local model
- Detects sudden spikes

**Physics Rules:**
- Safety compliance
- Deterministic detection
- Domain knowledge
- Regulatory requirements

**GNN (Attack Prediction):**
- Proactive defense
- Predicts next steps
- Federated learning
- Threat intelligence

### Why This Combination?

**Defense in Depth:**
- Multiple detection layers
- Different detection methods
- Complementary strengths
- Low false negative rate

**Federated + Local:**
- Best of both worlds
- Collaborative learning (LSTM, GNN)
- Facility-specific (IF, Physics)
- Optimal performance

**Speed + Accuracy:**
- Fast detection (IF, Physics)
- Accurate detection (LSTM)
- Proactive defense (GNN)
- High confidence (correlation)

---

**Document Version:** 1.0  
**Last Updated:** October 14, 2025  
**Status:** Ready for Implementation
