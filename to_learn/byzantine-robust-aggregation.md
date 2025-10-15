# Byzantine-Robust Aggregation

## What is Byzantine-Robust Aggregation?

A technique to aggregate updates from multiple participants while detecting and excluding malicious or corrupted contributions that could poison the system.

## The Byzantine Generals Problem

### Origin Story

Ancient problem in distributed computing:
- Multiple generals must coordinate an attack
- Some generals might be traitors
- Need to reach consensus despite traitors
- Cannot trust all participants

### In Federated Learning

```
10 facilities participate:
- 9 send legitimate weight updates
- 1 sends malicious updates (compromised or attacking)

Challenge: Aggregate updates without being poisoned
```

## The Threat: Model Poisoning

### Attack Scenarios

**1. Targeted Poisoning**
```
Goal: Make model blind to specific attack type
Attacker sends: Weights that reduce detection of technique T0843
Result: All facilities deploy compromised model
Impact: Industry-wide vulnerability
```

**2. Performance Degradation**
```
Goal: Reduce overall model accuracy
Attacker sends: Random garbage weights
Result: Model becomes less effective
Impact: Loss of trust in system
```

**3. Backdoor Injection**
```
Goal: Create hidden vulnerability
Attacker sends: Weights with specific trigger pattern
Result: Model fails only under specific conditions
Impact: Exploitable weakness
```

**4. Data Extraction**
```
Goal: Learn about other facilities
Attacker sends: Crafted weights designed to leak information
Result: Privacy breach
Impact: Competitive/security risk
```

## How Byzantine-Robust Aggregation Works

### Problem with Simple Average

```
Facility A: [0.52, 0.48, 0.61]
Facility B: [0.51, 0.49, 0.60]
Facility C: [0.53, 0.47, 0.62]
Facility D: [9.99, -8.45, 15.2] ← MALICIOUS!

Simple Average: [2.89, -1.63, 4.26] ← POISONED!
```

The malicious update dominates the average.

### Solution: Robust Aggregation

Detect and exclude outliers before aggregating.

## Byzantine-Robust Algorithms

### 1. Krum Algorithm

**Idea:** Select the update closest to the majority

**Steps:**
1. For each update, compute distance to all other updates
2. For each update, sum distances to k nearest neighbors
3. Select update with smallest sum (most "central")
4. Use only that update (or average of top-m)

**Example:**
```
Updates: A, B, C, D
Distances:
- A to others: 0.1 + 0.15 + 5.0 = 5.25
- B to others: 0.1 + 0.12 + 5.1 = 5.32
- C to others: 0.15 + 0.12 + 5.2 = 5.47
- D to others: 5.0 + 5.1 + 5.2 = 15.3 ← OUTLIER

Select A (smallest sum) or average A, B, C
```

**Properties:**
- ✅ Robust to f < n/2 malicious participants
- ✅ Provable guarantees
- ❌ Computationally expensive (O(n²))

### 2. Trimmed Mean

**Idea:** Remove extreme values, average the rest

**Steps:**
1. Sort all updates for each parameter
2. Remove top β% and bottom β%
3. Average the remaining middle values

**Example:**
```
Parameter 1 updates: [0.51, 0.52, 0.53, 9.99]
Sort: [0.51, 0.52, 0.53, 9.99]
Trim 25%: Remove 9.99
Average: (0.51 + 0.52 + 0.53) / 3 = 0.52
```

**Properties:**
- ✅ Simple and efficient
- ✅ Robust to β fraction of malicious participants
- ✅ Works well in practice
- ❌ Requires choosing β parameter

### 3. Median

**Idea:** Use median instead of mean

**Steps:**
1. For each parameter, collect all updates
2. Compute median (middle value)
3. Median is naturally resistant to outliers

**Example:**
```
Updates: [0.51, 0.52, 0.53, 9.99]
Median: 0.525 (average of 0.52 and 0.53)

Outlier 9.99 has no effect!
```

**Properties:**
- ✅ Very simple
- ✅ Naturally robust
- ✅ No parameters to tune
- ❌ Less efficient than mean when no attacks

### 4. Coordinate-wise Median (Project Choice)

**Idea:** Apply median to each weight independently

**Steps:**
```python
for each weight parameter w:
    collect all facility updates for w
    aggregated_w = median(updates)
```

**Example:**
```
Weight 1: [0.51, 0.52, 0.53, 9.99] → median = 0.525
Weight 2: [0.48, 0.49, 0.47, -8.45] → median = 0.48
Weight 3: [0.61, 0.60, 0.62, 15.2] → median = 0.61

Result: [0.525, 0.48, 0.61] ← Clean!
```

### 5. Multi-Krum

**Idea:** Select m closest updates and average them

**Steps:**
1. Run Krum algorithm
2. Select top-m updates (not just 1)
3. Average the selected updates

**Properties:**
- ✅ More robust than single Krum
- ✅ Better accuracy than single update
- ✅ Configurable (choose m)

## Detection Techniques

### Statistical Outlier Detection

**Z-Score Method:**
```python
mean = average(all_updates)
std = standard_deviation(all_updates)

for update in updates:
    z_score = (update - mean) / std
    if abs(z_score) > threshold:  # e.g., 3.0
        flag as outlier
```

**Interquartile Range (IQR):**
```python
Q1 = 25th percentile
Q3 = 75th percentile
IQR = Q3 - Q1

lower_bound = Q1 - 1.5 * IQR
upper_bound = Q3 + 1.5 * IQR

if update < lower_bound or update > upper_bound:
    flag as outlier
```

### Distance-Based Detection

**Euclidean Distance:**
```python
for each update:
    distances = [distance(update, other) for other in updates]
    avg_distance = mean(distances)
    
    if avg_distance > threshold:
        flag as outlier
```

### Reputation Systems

**Track facility behavior over time:**
```python
reputation[facility] = initial_value

for each round:
    if facility_update is outlier:
        reputation[facility] -= penalty
    else:
        reputation[facility] += reward
    
    if reputation[facility] < threshold:
        exclude facility
```

## Implementation in the Project

### Multi-Layer Defense

**Layer 1: Gradient Clipping**
```
Limit magnitude of individual updates
Max change per parameter: ±1.0
```

**Layer 2: Statistical Detection**
```
Compute z-scores for all updates
Flag updates with |z| > 3.0
```

**Layer 3: Robust Aggregation**
```
Apply coordinate-wise median or trimmed mean
Exclude flagged outliers
```

**Layer 4: Reputation Tracking**
```
Monitor facility behavior over time
Reduce weight of suspicious facilities
```

### Example Code Structure

```python
def byzantine_robust_aggregate(updates, method='median'):
    """
    Aggregate updates while excluding malicious ones.
    
    Args:
        updates: List of weight updates from facilities
        method: 'median', 'trimmed_mean', 'krum'
    
    Returns:
        Aggregated weights
    """
    # Step 1: Detect outliers
    outliers = detect_outliers(updates)
    
    # Step 2: Filter
    clean_updates = [u for u in updates if u not in outliers]
    
    # Step 3: Aggregate
    if method == 'median':
        return coordinate_wise_median(clean_updates)
    elif method == 'trimmed_mean':
        return trimmed_mean(clean_updates, beta=0.1)
    elif method == 'krum':
        return krum(clean_updates, m=5)
    
    # Step 4: Log suspicious activity
    if outliers:
        log_security_event(outliers)
```

## Guarantees and Limitations

### Theoretical Guarantees

**Krum:**
- Robust to f < n/2 Byzantine participants
- Provably converges to correct model

**Trimmed Mean:**
- Robust to β fraction of Byzantine participants
- Maintains statistical properties

**Median:**
- Robust to up to 50% Byzantine participants
- Breakdown point = 50%

### Practical Limitations

**Cannot Defend Against:**
- Majority of participants being malicious (>50%)
- Sophisticated attacks that mimic legitimate updates
- Collusion between multiple malicious participants

**Requires:**
- Sufficient number of honest participants
- Diversity in updates (not all identical)
- Proper parameter tuning

## Why It Matters for ICS

### 1. Critical Infrastructure Protection
- Cannot assume all facilities are trustworthy
- Insider threats are real
- Equipment malfunction can send bad data

### 2. Trust in Collaboration
- Facilities willing to participate
- Know they're protected from bad actors
- System remains reliable

### 3. Resilience
- System continues working despite attacks
- Graceful degradation
- No single point of failure

### 4. Compliance
- Demonstrates security measures
- Auditable protection
- Risk mitigation

## Key Takeaways

1. **Byzantine-robust aggregation protects against malicious participants**
2. **Multiple algorithms available** (Krum, Trimmed Mean, Median)
3. **Detects and excludes outlier updates**
4. **Essential for federated learning security**
5. **Project uses coordinate-wise median + statistical detection**
6. **Provides provable robustness guarantees**
7. **Enables safe collaboration in adversarial environments**

## Further Reading

- Paper: "Byzantine-Robust Distributed Learning" (Blanchard et al., 2017)
- Paper: "Machine Learning with Adversaries: Byzantine Tolerant Gradient Descent" (Krum)
- Survey: "Byzantine-Robust Learning on Heterogeneous Datasets"
- Implementation: https://github.com/bladesteam/blades
