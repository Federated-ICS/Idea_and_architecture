# Isolation Forest for Point Anomalies

## What is Isolation Forest?

An unsupervised machine learning algorithm that detects anomalies by "isolating" them. Based on the principle that anomalies are rare and different, making them easy to isolate from normal data.

## Core Concept

**Key Insight:** Anomalies are easier to isolate than normal points

```
Normal points (clustered):
- Require many splits to isolate
- Deep in the tree

Anomalies (isolated):
- Require few splits to isolate
- Shallow in the tree
```

## The Analogy

### Finding Someone in a Crowd

**Normal person (in a group):**
```
Q1: Are you in the north half? → Yes
Q2: Are you near the tree? → Yes
Q3: Are you wearing red? → No
Q4: Are you tall? → Yes
Q5: Are you male? → Yes
... many more questions needed
```

**Unusual person (standing alone):**
```
Q1: Are you standing alone? → Yes, found!
... only 1 question needed
```

## How It Works

### Step 1: Build Isolation Trees

```
Start with all data points

Random split: Temperature > 50°C?
├─ Yes (10 points)
│  └─ Random split: Pressure > 100 PSI?
│     ├─ Yes (8 points) ← Normal cluster
│     └─ No (2 points)
│
└─ No (1 point) ← ANOMALY! Isolated quickly
```

### Step 2: Measure Path Length

```
Path length = Number of splits to isolate a point

Normal point: Path length = 8 (many splits)
Anomaly: Path length = 2 (few splits)
```

### Step 3: Compute Anomaly Score

```
Anomaly Score = 2^(-path_length / c)

Where c = average path length for normal data

Short path → High score → ANOMALY
Long path → Low score → Normal
```

### Step 4: Build Forest

```
Build many trees (100-200)
Average scores across all trees
More robust than single tree
```

## Visual Example

### Normal Data Distribution

```
Scatter plot:

Pressure
  ↑
  |     ●●●●●
  |     ●●●●●
  |     ●●●●●  ← Normal cluster
  |     ●●●●●
  |
  └──────────→ Temperature

To isolate one ● from cluster:
- Many splits needed
- Deep in tree
```

### Anomaly Detection

```
Pressure
  ↑
  |     ●●●●●
  |     ●●●●●
  |     ●●●●●  ← Normal cluster
  |     ●●●●●
  |
  |              ○  ← Anomaly
  |
  └──────────→ Temperature

To isolate ○:
- Split 1: Temperature > 60? → Found!
- Only 1 split needed
- Shallow in tree
```

## ICS Example

### Normal Operation

```
Temperature: 45-55°C
Pressure: 95-105 PSI
Flow: 480-520 L/min

Data points cluster together:
[50, 100, 500]
[51, 102, 505]
[49, 98, 495]
[52, 103, 510]

Hard to isolate → Normal
```

### Attack/Anomaly

```
Sudden command injection:
Temperature: 95°C ← Way outside normal
Pressure: 103 PSI
Flow: 500 L/min

Data point: [95, 103, 500]

Easy to isolate:
Split 1: Temperature > 60? → Yes
Split 2: Temperature > 80? → Yes
Found in 2 splits → ANOMALY
```

## Algorithm Details

### Building an Isolation Tree

```python
def build_tree(data, current_height, max_height):
    if current_height >= max_height or len(data) <= 1:
        return Leaf(size=len(data))
    
    # Random feature
    feature = random.choice(features)
    
    # Random split value
    min_val = data[feature].min()
    max_val = data[feature].max()
    split_value = random.uniform(min_val, max_val)
    
    # Split data
    left_data = data[data[feature] < split_value]
    right_data = data[data[feature] >= split_value]
    
    # Recursively build subtrees
    left_tree = build_tree(left_data, current_height + 1, max_height)
    right_tree = build_tree(right_data, current_height + 1, max_height)
    
    return Node(feature, split_value, left_tree, right_tree)
```

### Computing Anomaly Score

```python
def anomaly_score(path_length, n):
    """
    path_length: Average path length for the point
    n: Number of training samples
    """
    c = 2 * (np.log(n - 1) + 0.5772) - 2 * (n - 1) / n
    score = 2 ** (-path_length / c)
    return score

# Interpretation:
# score ≈ 1.0 → Anomaly
# score ≈ 0.5 → Normal
# score ≈ 0.0 → Very normal
```

## Why "Forest"?

### Single Tree Problem

```
One tree might get unlucky:
- Random split happens to isolate normal point
- False positive

Solution: Build many trees
```

### Ensemble Approach

```
Build 100 trees with different random splits
For each point:
- Compute path length in each tree
- Average path lengths
- Compute final anomaly score

Result: More robust, fewer false positives
```

## Isolation Forest vs LSTM Autoencoder

### Different Types of Anomalies

**Point Anomalies (Isolation Forest):**
```
Single weird value
Example: Temperature suddenly jumps to 999°C

Isolation Forest: ✓ Detects
LSTM: Maybe (depends on sequence)
```

**Sequential Anomalies (LSTM):**
```
Weird pattern over time
Example: Gradual drift from 50°C to 90°C over 20 steps

Isolation Forest: ✗ Misses (each point looks normal)
LSTM: ✓ Detects
```

### Complementary Strengths

```
Attack Type 1: Sudden Command Injection
Value: 9999 (way outside normal range)
Isolation Forest: ✓ Fast detection
LSTM: ✓ Also detects

Attack Type 2: Slow Parameter Drift
Values: 50 → 52 → 54 → ... → 90
Isolation Forest: ✗ Each value looks normal
LSTM: ✓ Sequence is abnormal

Attack Type 3: Unusual Command Sequence
Values: All normal, but sequence is wrong
Isolation Forest: ✗ Values are normal
LSTM: ✓ Sequence pattern is abnormal
```

## Advantages

✅ **Fast:** O(n log n) training, O(log n) prediction
✅ **Unsupervised:** No labeled data needed
✅ **Handles high dimensions:** Works with many features
✅ **Robust:** Ensemble approach reduces false positives
✅ **Interpretable:** Can explain via path length
✅ **No assumptions:** Doesn't assume data distribution

## Limitations

❌ **Point anomalies only:** Misses sequential anomalies
❌ **No temporal context:** Doesn't consider time
❌ **Threshold tuning:** Need to set anomaly score threshold
❌ **High-dimensional curse:** Performance degrades with too many features
❌ **No causality:** Doesn't explain why it's anomalous

## Hyperparameters

### Number of Trees
```
Too few (e.g., 10):
- Unstable predictions
- High variance

Too many (e.g., 1000):
- Diminishing returns
- Slower training

Optimal: 100-200 trees
```

### Subsample Size
```
Too small (e.g., 32):
- May miss patterns
- Overfitting

Too large (e.g., all data):
- Slow training
- Memory issues

Optimal: 256 samples per tree
```

### Contamination
```
Expected proportion of anomalies in data

Low (e.g., 0.01 = 1%):
- Strict threshold
- Fewer false positives
- May miss some anomalies

High (e.g., 0.1 = 10%):
- Loose threshold
- More detections
- More false positives

Typical: 0.01-0.05 for ICS
```

## Implementation

```python
from sklearn.ensemble import IsolationForest

# Create model
model = IsolationForest(
    n_estimators=100,      # Number of trees
    max_samples=256,       # Subsample size
    contamination=0.01,    # Expected anomaly rate
    random_state=42
)

# Train on normal data
model.fit(normal_data)

# Predict
predictions = model.predict(new_data)
# Returns: 1 for normal, -1 for anomaly

# Get anomaly scores
scores = model.score_samples(new_data)
# Returns: Lower score = more anomalous
```

## Federated Learning

### Why Federate Isolation Forest?

```
Benefits from diverse training data:
- Facility A: Power plant normal patterns
- Facility B: Water utility normal patterns
- Facility C: Manufacturing normal patterns

Combined model:
- Broader baseline of "normal"
- Fewer false positives
- Better generalization
```

### How to Federate

```
Challenge: Isolation Forest is tree-based, not gradient-based

Solution 1: Federate the training data distribution
- Each facility sends summary statistics
- Central model trains on aggregated statistics

Solution 2: Ensemble of local models
- Each facility trains local Isolation Forest
- Aggregate predictions (voting or averaging)

Solution 3: Convert to neural network
- Train neural network to mimic Isolation Forest
- Federate the neural network weights
```

## Use in the Project

### Integration with Other Components

```
Data Flow:
1. Sensor reading arrives
2. Isolation Forest checks for point anomalies
3. LSTM checks for sequential anomalies
4. Physics rules check for process violations
5. Correlation engine combines all signals
6. Alert if multiple signals agree
```

### Example Scenario

```
08:00:00 - Normal reading: [50, 100, 500]
Isolation Forest: Score = 0.45 (normal)
LSTM: Reconstruction error = 0.2 (normal)
Physics: Within bounds ✓
Result: No alert

08:00:01 - Attack: [999, 100, 500]
Isolation Forest: Score = 0.95 (ANOMALY!) ✓
LSTM: Reconstruction error = 35.0 (ANOMALY!) ✓
Physics: Temperature > max (VIOLATION!) ✓
Result: HIGH CONFIDENCE ALERT
```

## Key Takeaways

1. **Isolation Forest detects point anomalies** - single weird values
2. **Based on path length** - anomalies are easy to isolate
3. **Fast and efficient** - suitable for real-time detection
4. **Unsupervised** - no labeled data needed
5. **Complements LSTM** - different anomaly types
6. **Federated** - benefits from multi-facility data
7. **Project uses it for outlier detection** - first line of defense

## Further Reading

- Paper: "Isolation Forest" (Liu et al., 2008)
- Paper: "Isolation-Based Anomaly Detection" (Liu et al., 2012)
- Implementation: scikit-learn documentation
- Tutorial: "Anomaly Detection with Isolation Forest"
