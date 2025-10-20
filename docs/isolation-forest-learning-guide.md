# Isolation Forest - Complete Learning Guide

## Table of Contents
1. [Core Concept](#core-concept)
2. [How It Works](#how-it-works)
3. [Building Isolation Trees](#building-isolation-trees)
4. [Anomaly Scoring](#anomaly-scoring)
5. [Training Process](#training-process)
6. [Implementation Guide](#implementation-guide)
7. [Use Cases](#use-cases)
8. [Quick Reference](#quick-reference)

---

## Core Concept

### The Big Insight 💡

**"Anomalies are few and different, so they're easier to isolate than normal points."**

### Visual Example

```
Normal readings (most of the data):
●●●●●●●●●●
●●●●●●●●●●  ← Clustered together around 300°C
●●●●●●●●●●

Anomaly (rare and different):
              ★  ← Way out at 450°C
```

### The Key Question

**"How many random splits does it take to isolate a point?"**

#### For the anomaly (★):
```
Split 1: Is temperature > 350°C?
         YES → Isolated! ★ is alone
```
**Result: 1 split needed**

#### For a normal point (●):
```
Split 1: Is temperature > 350°C?
         NO → Still with many other points ●●●●●●

Split 2: Is temperature > 310°C?
         NO → Still with some points ●●●●

Split 3: Is temperature > 305°C?
         YES → Still with a few points ●●

Split 4: Is temperature > 302°C?
         NO → Finally isolated one point ●
```
**Result: 4 splits needed**

### The Principle

- **Anomalies** = Few splits to isolate (short path)
- **Normal points** = Many splits to isolate (long path)

---

## How It Works

### Isolation Forest vs Normal Decision Trees

#### Normal Decision Tree (for classification):
- Looks at the data carefully
- Finds the BEST split (using metrics like Gini or entropy)
- Tries to separate classes optimally
- Goal: Accurate classification

#### Isolation Tree (for anomaly detection):
- Makes RANDOM splits
- Picks a random feature
- Picks a random split value (between min and max)
- Goal: Isolate points quickly

### Why Random Splits?

Random splits are brilliant because:

1. **Faster to compute** ⚡
   - No need to calculate information gain, Gini index, etc.
   - Just pick random feature + random value
   - Much faster than traditional decision trees

2. **Works better for outliers** 🎯
   - Anomalies are already "easy to isolate"
   - Random splits will naturally isolate them quickly
   - We don't need "optimal" splits - random works great!

3. **Prevents overfitting** 🛡️
   - Random splits don't memorize the training data
   - Each tree is different (diversity)
   - More robust to variations in data

---

## Building Isolation Trees

### Example: Building One Isolation Tree

**Your data (temperature readings):**
```
Point 1: 300°C
Point 2: 305°C
Point 3: 298°C
Point 4: 450°C  ← anomaly
Point 5: 302°C
```

**Tree building process:**

```
Root: All 5 points [300, 305, 298, 450, 302]
  ↓
Random split: "temperature > 320°C?" (random value between 298-450)
  ↓
├─ NO: [300, 305, 298, 302]  (4 points)
│   ↓
│   Random split: "temperature > 301°C?"
│   ↓
│   ├─ NO: [300, 298]
│   └─ YES: [305, 302]
│
└─ YES: [450]  ← Isolated in just 1 split! ★
```

### The Ensemble Approach

Isolation Forest doesn't build just ONE tree - it builds MANY trees (typically 100).

**Why multiple trees?**

One random tree might get unlucky with its splits. But if you build 100 trees:
- Anomalies will have short paths in MOST trees
- Normal points will have long paths in MOST trees
- Average them out = reliable detection!

**Example:**

```
Anomaly across 100 trees:
Tree 1: Anomaly isolated in 2 splits
Tree 2: Anomaly isolated in 1 split
Tree 3: Anomaly isolated in 2 splits
...
Tree 100: Anomaly isolated in 1 split

Average path length: ~1.5 splits (SHORT!)
```

```
Normal point across 100 trees:
Tree 1: Normal point isolated in 8 splits
Tree 2: Normal point isolated in 7 splits
Tree 3: Normal point isolated in 9 splits
...
Tree 100: Normal point isolated in 8 splits

Average path length: ~8 splits (LONG!)
```

---

## Anomaly Scoring

### The Formula

Isolation Forest calculates a score between **0 and 1**:

```
Anomaly Score = 2^(-average_path_length / c)

where c = average path length of a normal point
```

### Score Interpretation

#### Score close to 1 → Anomaly
- Short path length
- Easy to isolate
- **Different from the rest**

#### Score close to 0.5 → Normal
- Average path length
- Not particularly easy or hard to isolate
- **Similar to most points**

#### Score close to 0 → Very normal
- Long path length
- Hard to isolate (buried in the cluster)
- **Very typical point**

### Example Scores

```
Your data points and their scores:

Point A: 300°C  → Average path: 8 splits → Score: 0.45 (normal)
Point B: 305°C  → Average path: 7 splits → Score: 0.48 (normal)
Point C: 298°C  → Average path: 9 splits → Score: 0.42 (normal)
Point D: 450°C  → Average path: 2 splits → Score: 0.92 (ANOMALY!)
Point E: 302°C  → Average path: 8 splits → Score: 0.45 (normal)
```

### Setting the Threshold

Common threshold: **0.6**

```python
if anomaly_score > 0.6:
    alert("Anomaly detected!")
```

This means:
- Score 0.92 → ALERT! ✅
- Score 0.45 → Normal ✅
- Score 0.65 → ALERT! ✅
- Score 0.55 → Normal ✅

### Multi-Feature Detection

**Important:** Even if only 1-2 features out of 18 are anomalous, Isolation Forest can still detect it!

**Example - Port Scan Attack:**
- Temperature: 300°C (normal)
- Pressure: 120 psi (normal)
- Packets/sec: 3000 (VERY HIGH! Normal is 45)
- Unique dest IPs: 250 (VERY HIGH! Normal is 5)

**Result:** Short path length because the network features are extreme outliers!

```
Tree 1: "packets/sec > 1000?" → YES! Isolated immediately ★
Tree 2: "unique_dest_ips > 100?" → YES! Isolated immediately ★
Tree 3: "temperature > 310?" → NO, "packets/sec > 500?" → YES! ★
```

---

## Training Process

### Step 1: Collect Normal Data

```python
# You need examples of "normal" operation
normal_data = [
    [300, 120, 75, 45, 5, ...],  # Normal day 1
    [305, 118, 73, 47, 4, ...],  # Normal day 2
    [298, 122, 76, 46, 5, ...],  # Normal day 3
    ...
    # 10,000+ samples of normal operation
]
```

### Step 2: Build the Forest

```python
from sklearn.ensemble import IsolationForest

model = IsolationForest(
    n_estimators=100,      # Build 100 trees
    contamination=0.01,    # Expect 1% anomalies
    max_samples=256,       # Use 256 samples per tree
    random_state=42
)

model.fit(normal_data)  # Train on normal data only!
```

### Step 3: Make Predictions

```python
new_data = [[450, 120, 75, 3000, 250, ...]]  # Port scan attack!

score = model.decision_function(new_data)  # Returns anomaly score
prediction = model.predict(new_data)        # Returns -1 (anomaly) or 1 (normal)

if score > 0.6:
    print("ANOMALY DETECTED!")
```

### Key Training Insight

**You only train on NORMAL data!**

Why?
- You have lots of normal operation data
- You might not have attack examples
- The model learns what "normal" looks like
- Anything different = anomaly

### ⚠️ Critical Warning: Data Contamination

**What happens if you accidentally include attacks in your training data?**

The model will think those attacks are "normal" and won't detect them!

**Example:**

```python
# Your "normal" training data (but oops, day 5 had an attack!)
training_data = [
    [300, 120, 75, 45, 5],   # Normal day 1
    [305, 118, 73, 47, 4],   # Normal day 2
    [298, 122, 76, 46, 5],   # Normal day 3
    [302, 119, 74, 44, 6],   # Normal day 4
    [450, 120, 75, 3000, 250], # Attack day 5! ← CONTAMINATED!
    [301, 121, 75, 45, 5],   # Normal day 6
    ...
]

model.fit(training_data)  # Model learns this is "normal"
```

**Result:** When a similar attack happens later, it won't flag it!

**Solution:** Use the `contamination` parameter to be tolerant of small amounts of noise:

```python
IsolationForest(contamination=0.01)  # Expect about 1% might be anomalies
```

But ideally, you want **clean normal data** for training!

---

## Implementation Guide

### Complete Workflow for ICS System

```python
# ============================================
# 1. TRAINING PHASE (one-time setup)
# ============================================

# Collect 7 days of normal operation
normal_data = query_iotdb("SELECT * FROM facility_a WHERE timestamp BETWEEN ...")
# Shape: (10000, 18) - 10,000 samples, 18 features
# Features: 10 process + 8 network

# Train Isolation Forest
model = IsolationForest(
    n_estimators=100,
    contamination=0.01,
    max_samples=256
)
model.fit(normal_data)

# Save model
import joblib
joblib.dump(model, "isolation_forest_facility_a.pkl")

# ============================================
# 2. DETECTION PHASE (real-time)
# ============================================

# Load model
model = joblib.load("isolation_forest_facility_a.pkl")

while True:
    # Get latest sensor + network data
    current_data = get_latest_data()  # Shape: (1, 18)
    
    # Run detection
    score = model.decision_function(current_data)[0]
    
    if score > 0.6:
        alert = {
            "source": "IsolationForest",
            "confidence": score,
            "severity": "WARNING",
            "message": "Outlier detected",
            "timestamp": datetime.now(),
            "features": current_data.tolist()
        }
        
        # Publish to Kafka
        publish_to_kafka(alert)
        
        # Store in PostgreSQL
        store_in_postgres(alert)
        
        # Log
        logger.warning(f"Anomaly detected! Score: {score:.2f}")
    
    time.sleep(1)  # Check every second
```

### Key Parameters Explained

```python
IsolationForest(
    n_estimators=100,      # Number of trees (more = more stable, but slower)
    contamination=0.01,    # Expected % of anomalies in training data
    max_samples=256,       # Samples per tree (smaller = faster, less accurate)
    max_features=1.0,      # Fraction of features to use (1.0 = all features)
    random_state=42        # For reproducibility
)
```

**Tuning Tips:**
- **n_estimators:** 100 is usually good. Increase to 200 for more stability.
- **contamination:** Set to expected anomaly rate (0.01 = 1%)
- **max_samples:** 256 is default. Decrease for faster training on large datasets.

---

## Use Cases

### When to Use Isolation Forest

**✅ GOOD FOR:**
- Sudden spikes/drops
- Point anomalies
- Fast detection needed (< 5 seconds)
- Unlabeled data (unsupervised)
- High-dimensional data (many features)
- Real-time systems
- Outlier detection

**❌ NOT GOOD FOR:**
- Gradual changes over time
- Temporal patterns
- Anomalies that need context
- When you need to understand "why"
- Collective anomalies (groups of points)

### Attack Types - What Isolation Forest Detects Best

**✅ EXCELLENT FOR:**
- Sudden temperature spike (300°C → 450°C)
- Port scan (packets/sec: 45 → 3000)
- Network traffic spike (bytes/sec: 1000 → 50000)
- Unauthorized access attempts (failed_connections: 2 → 150)
- Sudden pressure drop (120 psi → 50 psi)

**❌ STRUGGLES WITH:**
- Gradual temperature drift (300°C → 310°C → 320°C → ... over 60 seconds)
- Slow data exfiltration
- Coordinated multi-stage attacks
- Attacks that mimic normal patterns

### Comparison with Other Methods

| Method | Detection Time | Best For | Federated? |
|--------|---------------|----------|------------|
| **Isolation Forest** | < 5 seconds | Sudden outliers | ❌ No |
| **LSTM Autoencoder** | < 30 seconds | Behavioral anomalies | ✅ Yes |
| **Physics Rules** | < 1 second | Known violations | ❌ No |

---

## Quick Reference

### Cheat Sheet

```python
# Training
from sklearn.ensemble import IsolationForest
model = IsolationForest(n_estimators=100, contamination=0.01)
model.fit(normal_data)

# Detection
score = model.decision_function(new_data)[0]
is_anomaly = score > 0.6

# Prediction
prediction = model.predict(new_data)  # -1 = anomaly, 1 = normal
```

### For Your ICS System

| Feature | Value |
|---------|-------|
| **Input** | 18 features (10 process + 8 network) |
| **Training data** | 10,000+ normal samples (7 days) |
| **Detection time** | < 5 seconds |
| **Threshold** | 0.6 |
| **Best for** | Sudden spikes, port scans, outliers |
| **Federated?** | ❌ No (local per facility) |
| **Training frequency** | Weekly or when normal patterns change |

### System Integration

```
Your 3-Layer Detection:

Layer 1: Physics Rules (< 1 sec)
         ↓ Catches obvious violations

Layer 2: Isolation Forest (< 5 sec) ← Fast outlier detection
         ↓ Catches sudden spikes/outliers

Layer 3: LSTM (< 30 sec)
         ↓ Catches behavioral anomalies

         ↓
   Correlation Engine
   (combines all 3 for high-confidence alerts)
```

**Isolation Forest's Role:**
- Fast outlier detection
- Catches sudden attacks (spikes, port scans)
- Complements LSTM (which catches gradual changes)
- Provides second opinion for correlation
- No time window needed (instant detection)

### Common Pitfalls

1. **Training on contaminated data** → Model learns attacks as normal
2. **Threshold too low** → Too many false positives
3. **Threshold too high** → Misses real attacks
4. **Not retraining** → Model becomes stale as normal patterns change
5. **Too few training samples** → Poor baseline of "normal"

### Best Practices

1. ✅ Train on at least 7 days of clean normal data
2. ✅ Retrain weekly or when operations change
3. ✅ Monitor false positive rate and adjust threshold
4. ✅ Use with other detection methods (defense in depth)
5. ✅ Log all detections for analysis
6. ✅ Validate training data is truly "normal"

---

## Real-World Example

### Scenario: Port Scan Detection

**Normal Operation:**
```
Time 10:00 - packets/sec: 45, unique_dest_ips: 5, score: 0.45 (normal)
Time 10:01 - packets/sec: 47, unique_dest_ips: 4, score: 0.48 (normal)
```

**Attack Begins:**
```
Time 10:02 - packets/sec: 3000, unique_dest_ips: 250, score: 0.95 (ALERT!)
Time 10:03 - packets/sec: 2847, unique_dest_ips: 245, score: 0.93 (ALERT!)
```

**Attack Stops:**
```
Time 10:04 - packets/sec: 46, unique_dest_ips: 5, score: 0.48 (normal)
```

**What Happened:**
- Brief port scan attack occurred at 10:02-10:03
- Isolation Forest detected it immediately (< 5 seconds)
- Attack stopped, system returned to normal
- This is exactly what Isolation Forest excels at!

---

## Summary

### Key Takeaways

1. **Core Principle:** Anomalies are easier to isolate than normal points
2. **Method:** Build 100 random trees, average path lengths
3. **Training:** Unsupervised, train on normal data only
4. **Detection:** Fast (< 5 seconds), works on single data points
5. **Best For:** Sudden outliers, spikes, port scans
6. **Limitation:** Doesn't catch gradual changes (use LSTM for that)

### Why It's Perfect for Your System

- ✅ Fast detection (< 5 seconds)
- ✅ Works with 18 features (process + network)
- ✅ Catches sudden attacks (spikes, port scans)
- ✅ Complements LSTM (different detection types)
- ✅ Easy to implement (sklearn)
- ✅ Low computational cost
- ✅ No labeled data needed

---

**Document Version:** 1.0  
**Last Updated:** October 18, 2025  
**Author:** Learning Session Notes
