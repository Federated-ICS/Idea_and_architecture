# Practical Federated Learning Parameters

## Overview

This guide covers the practical parameters and configurations for federated learning in the ICS Threat Correlation Engine project.

## Training Schedule

### Rounds Per Day

**Configuration: 4 rounds per day**

```
Schedule:
- Round 1: 00:00 (midnight)
- Round 2: 06:00 (morning)
- Round 3: 12:00 (noon)
- Round 4: 18:00 (evening)

Interval: Every 6 hours
```

**Why 4 rounds?**

```
✓ Fast threat propagation (6-24 hour window)
✓ Enough time to collect meaningful data
✓ Not too frequent (network/compute overhead)
✓ Balances real-time protection with efficiency
```

**Alternatives:**

```
Too Frequent (24 rounds/day = hourly):
❌ Not enough new data between rounds
❌ Excessive network traffic
❌ Computational overhead
❌ Diminishing returns

Too Infrequent (1 round/week):
❌ Slow threat propagation
❌ Defeats the purpose
❌ Facilities vulnerable for days

Just Right (4 rounds/day):
✓ Optimal balance
```

### Local Training Epochs

**Configuration: 5 epochs per round**

```
Each facility:
- Trains model on local data
- 5 complete passes through data
- Takes ~20-30 minutes
```

**Why 5 epochs?**

```
✓ Enough learning without overfitting
✓ Fast enough to complete within 6-hour window
✓ Good quality weight updates
✓ Industry standard for federated learning
```

**Alternatives:**

```
Too Few (1 epoch):
❌ Model doesn't learn enough
❌ Poor quality updates
❌ Wasted opportunity

Too Many (50 epochs):
❌ Overfitting to local data
❌ Takes too long
❌ Diminishing returns
❌ May miss the 6-hour window

Just Right (5 epochs):
✓ Optimal learning
```

## Model Parameters

### LSTM Autoencoder

**Architecture:**
```
Encoder:
- Input: (batch_size, sequence_length, n_features)
- LSTM Layer 1: 128 units
- LSTM Layer 2: 64 units
- Output: Latent vector (64 dimensions)

Decoder:
- Input: Latent vector (64 dimensions)
- LSTM Layer 1: 64 units
- LSTM Layer 2: 128 units
- Output: (batch_size, sequence_length, n_features)
```

**Hyperparameters:**
```
sequence_length: 60 (60 seconds of data)
n_features: 10-50 (sensor readings, protocol features)
batch_size: 32
learning_rate: 0.001
dropout: 0.2
```

**Why these values?**

```
128/64 units:
✓ Enough capacity for complex patterns
✓ Not too large (overfitting, slow)
✓ Standard for time-series

60-second sequences:
✓ Captures attack sequences
✓ Not too long (memory, speed)
✓ Typical attack step duration

Batch size 32:
✓ Stable gradients
✓ Reasonable memory usage
✓ Good convergence
```

### Graph Neural Network (GAT)

**Architecture:**
```
Input: Node features (num_techniques, feature_dim)

GAT Layer 1:
- Hidden dim: 64
- Attention heads: 4
- Activation: ELU
- Dropout: 0.6

GAT Layer 2:
- Hidden dim: 32
- Attention heads: 4
- Activation: ELU
- Dropout: 0.6

Output:
- Dense layer
- Softmax activation
- Output: (num_techniques,) probabilities
```

**Hyperparameters:**
```
hidden_dim_1: 64
hidden_dim_2: 32
num_heads: 4
dropout: 0.6
learning_rate: 0.005
```

**Why these values?**

```
4 attention heads:
✓ Multiple perspectives on relationships
✓ Not too many (overfitting)
✓ Standard for GAT

64/32 hidden dimensions:
✓ Sufficient capacity
✓ Reasonable size
✓ Good generalization

Dropout 0.6:
✓ Prevents overfitting on graph
✓ Higher than typical (graphs need more regularization)
```

### Isolation Forest

**Hyperparameters:**
```
n_estimators: 100 (number of trees)
max_samples: 256 (subsample size)
contamination: 0.01 (1% expected anomalies)
max_features: 1.0 (use all features)
bootstrap: False
```

**Why these values?**

```
100 trees:
✓ Good ensemble diversity
✓ Stable predictions
✓ Not too slow

256 samples per tree:
✓ Enough data to learn patterns
✓ Fast training
✓ Good generalization

1% contamination:
✓ Realistic for ICS (attacks are rare)
✓ Strict threshold
✓ Low false positives
```

## Privacy Parameters

### Differential Privacy

**Configuration:**
```
epsilon (ε): 2.0
delta (δ): 10⁻⁵ (0.00001)
```

**Noise Mechanism:**
```
Gaussian noise:
- Mean: 0
- Std: Calibrated to ε and δ
- Applied to gradients before sending
```

**Why ε=2.0?**

```
✓ Moderate privacy (industry standard)
✓ Good model accuracy
✓ Meets regulatory requirements
✓ Balances privacy and utility
```

**Privacy Budget Management:**
```
Per round: ε = 0.5
Per day (4 rounds): ε = 2.0
Per week: ε = 14.0

Need to reset or manage budget over time
```

### Gradient Clipping

**Configuration:**
```
max_norm: 1.0 (L2 norm clipping)
```

**Why 1.0?**

```
✓ Prevents extreme updates
✓ Allows normal learning
✓ Protects against poisoning
✓ Standard value for FL
```

**Alternatives:**
```
Conservative (0.5):
- More protection
- Slower learning

Permissive (2.0):
- Faster learning
- Less protection

Balanced (1.0):
- Optimal tradeoff
```

## Communication Parameters

### Weight Update Size

**Configuration:**
```
Model weights: ~10 MB per facility per round
```

**Breakdown:**
```
LSTM Autoencoder: ~5 MB
GNN: ~3 MB
Isolation Forest: ~2 MB
Total: ~10 MB
```

**Network Requirements:**
```
Per facility per day:
- 4 rounds × 10 MB = 40 MB transmitted
- Minimal bandwidth requirement
- Works on standard industrial networks
```

### Compression

**Optional:**
```
Gradient compression:
- Quantization (32-bit → 16-bit)
- Sparsification (send only top-k)
- Can reduce size by 50-75%

Trade-off:
- Smaller size
- Slight accuracy loss
- More computation
```

## Aggregation Parameters

### Byzantine-Robust Aggregation

**Configuration:**
```
Method: Coordinate-wise median
Outlier detection: Z-score > 3.0
Minimum participants: 3 facilities
```

**Why median?**

```
✓ Naturally robust to outliers
✓ Simple and efficient
✓ No parameters to tune
✓ Works well in practice
```

**Outlier threshold:**
```
Z-score > 3.0:
✓ Catches extreme outliers
✓ Not too strict (false positives)
✓ Standard statistical threshold
```

### Minimum Participants

**Configuration:**
```
Minimum: 3 facilities
Optimal: 10+ facilities
```

**Why minimum 3?**

```
2 facilities:
❌ No privacy (can infer other's data)
❌ No robustness (one bad = 50% influence)

3 facilities:
✓ Basic privacy
✓ Median works
✓ Some robustness

10+ facilities:
✓ Strong privacy
✓ Excellent robustness
✓ Better model accuracy
```

## Performance Targets

### Detection Latency

**Target: <30 seconds from event to alert**

```
Breakdown:
- Data ingestion: <5 seconds
- Detection (LSTM, IF): <10 seconds
- Correlation: <5 seconds
- Alert generation: <5 seconds
- Buffer: 5 seconds

Total: 30 seconds
```

### Detection Accuracy

**Targets:**
```
Single facility: 92% accuracy, 8% false positives
Federated (10 facilities): 96% accuracy, 4% false positives
Federated (50 facilities): 97% accuracy, 3% false positives

Goal: >95% accuracy, <5% false positives
```

### Attack Prediction

**Target: 67% accuracy for next-step prediction**

```
Interpretation:
- Given current technique detected
- Predict next technique with 67% accuracy
- Better than random (would be ~5% for 20 techniques)
- Actionable for defenders
```

### MITRE ATT&CK Coverage

**Target: 87% of ICS techniques**

```
Total ICS techniques: ~80
Covered: ~70 techniques
Gaps: ~10 techniques (rare or new)

Validated by red team simulator
```

## Timing Parameters

### FL Round Timeline

**Total: ~40 minutes per round**

```
00:00 - Round starts
00:00-00:05 (5 min): Server distributes model
00:05-00:30 (25 min): Facilities train locally (5 epochs)
00:30-00:35 (5 min): Facilities send weight updates
00:35-00:40 (5 min): Server aggregates
00:40-06:00: Improved model deployed and used

Next round: 06:00
```

**Buffer time: 5 hours 20 minutes**

```
Allows for:
- Network delays
- Facility downtime
- Retries
- Maintenance windows
```

### Attack Propagation Time

**Target: 6-24 hours**

```
Best case: 10 minutes
- Attack happens just before round
- Next round starts immediately
- All facilities protected

Average case: 3 hours
- Attack happens mid-interval
- Half-way to next round
- Reasonable protection window

Worst case: 6 hours
- Attack happens just after round
- Must wait for next round
- Still much better than weeks
```

## Resource Requirements

### Per Facility

**Compute:**
```
CPU: 4-8 cores
RAM: 16-32 GB
GPU: Optional (speeds up training 5-10x)
Storage: 500 GB - 1 TB
```

**Network:**
```
Bandwidth: 10 Mbps minimum
Latency: <100 ms to FL server
Reliability: 99% uptime
```

### FL Server

**Compute:**
```
CPU: 8-16 cores
RAM: 32-64 GB
GPU: Not required (aggregation is CPU-bound)
Storage: 1-2 TB
```

**Network:**
```
Bandwidth: 100 Mbps minimum
Latency: <50 ms to facilities
Reliability: 99.9% uptime
```

## Scaling Parameters

### Small Deployment (3-5 facilities)

```
Rounds per day: 4
Epochs per round: 5
Aggregation: Simple median
Privacy: ε=2.0
Expected accuracy: 94%
```

### Medium Deployment (10-20 facilities)

```
Rounds per day: 4
Epochs per round: 5
Aggregation: Trimmed mean + median
Privacy: ε=1.5
Expected accuracy: 96%
```

### Large Deployment (50+ facilities)

```
Rounds per day: 6 (every 4 hours)
Epochs per round: 3 (faster rounds)
Aggregation: Multi-Krum + median
Privacy: ε=1.0
Expected accuracy: 97%
```

## Monitoring Parameters

### Health Metrics

**Track per round:**
```
- Number of participants
- Average training time
- Gradient norms
- Model accuracy
- False positive rate
- Detection latency
```

**Alerts:**
```
- Participant count < minimum
- Training time > threshold
- Gradient norms > 10.0
- Accuracy drop > 5%
- False positives > 10%
- Latency > 30 seconds
```

## Configuration File Example

```yaml
federated_learning:
  rounds_per_day: 4
  schedule: ["00:00", "06:00", "12:00", "18:00"]
  local_epochs: 5
  min_participants: 3
  
privacy:
  differential_privacy:
    epsilon: 2.0
    delta: 0.00001
  gradient_clipping:
    max_norm: 1.0
  secure_aggregation:
    enabled: true

models:
  lstm_autoencoder:
    encoder_units: [128, 64]
    decoder_units: [64, 128]
    sequence_length: 60
    batch_size: 32
    learning_rate: 0.001
    dropout: 0.2
  
  gnn:
    hidden_dims: [64, 32]
    num_heads: 4
    dropout: 0.6
    learning_rate: 0.005
  
  isolation_forest:
    n_estimators: 100
    max_samples: 256
    contamination: 0.01

aggregation:
  method: "median"
  outlier_threshold: 3.0
  byzantine_robust: true

performance:
  detection_latency_target: 30  # seconds
  accuracy_target: 0.95
  false_positive_target: 0.05
  prediction_accuracy_target: 0.67

resources:
  facility:
    cpu_cores: 8
    ram_gb: 32
    storage_gb: 1000
  server:
    cpu_cores: 16
    ram_gb: 64
    storage_gb: 2000
```

## Key Takeaways

1. **4 rounds per day, 5 epochs per round** - Optimal schedule
2. **ε=2.0, δ=10⁻⁵** - Balanced privacy
3. **Gradient clipping at 1.0** - Stability and security
4. **~10 MB per update** - Minimal bandwidth
5. **<30 second detection** - Real-time protection
6. **>95% accuracy, <5% FP** - Production-ready
7. **67% prediction accuracy** - Actionable intelligence
8. **87% ATT&CK coverage** - Comprehensive protection

## Further Reading

- Paper: "Federated Learning: Strategies for Improving Communication Efficiency"
- Paper: "Adaptive Federated Optimization"
- Tutorial: "Hyperparameter Tuning for Federated Learning"
- Best practices: "Production Federated Learning Systems"
