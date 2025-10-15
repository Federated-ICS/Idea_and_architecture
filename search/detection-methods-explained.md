# Three Detection Methods Explained

## The Defense-in-Depth Strategy

Think of it like having three different security guards, each with unique skills:

### 1. LSTM Autoencoder - The Pattern Expert 🧠

**What it does:**
- Learns the "normal rhythm" of your ICS over time
- Trained on sequences of sensor readings (temperature, pressure, flow rates)
- Reconstructs what it expects to see based on historical patterns

**How it detects attacks:**
- Compares actual readings vs. what it predicted
- High reconstruction error = something's off
- Example: "Temperature usually rises gradually over 10 minutes, but this jumped instantly"

**What it catches:**
- Subtle behavioral changes over time
- Coordinated multi-sensor attacks
- Slow drift attacks (boiling frog scenario)
- Temporal anomalies that look fine in isolation

**What it misses:**
- Sudden spikes (needs time to see the pattern break)
- Attacks that happen faster than its time window
- Single-point outliers without temporal context

**Real example:**
```
Normal: temp=[100, 102, 104, 106, 108] over 5 minutes
Attack: temp=[100, 102, 250, 106, 108] - sudden spike at minute 3
LSTM might miss this if it's too sudden, but will catch the recovery pattern
```

---

### 2. Isolation Forest - The Outlier Hunter 🎯

**What it does:**
- Statistical anomaly detection using decision trees
- Isolates data points that are "easy to separate" from the crowd
- Works on individual data points, not sequences

**How it detects attacks:**
- Builds a forest of random decision trees
- Anomalies require fewer splits to isolate
- Assigns anomaly score: closer to 1 = more anomalous

**What it catches:**
- Sudden spikes and drops
- Values way outside normal range
- Point anomalies that happen instantly
- Attacks the LSTM might need time to recognize

**What it misses:**
- Gradual changes that stay within "reasonable" bounds
- Attacks that manipulate values slowly
- Context about whether the spike makes sense (physics)

**Real example:**
```
Normal range: pressure = 50-60 PSI
Attack: pressure suddenly = 250 PSI
Isolation Forest: "This is 4 standard deviations out - ALERT!"
Catches it in <5 seconds, no temporal context needed
```

---

### 3. Physics Rules Engine - The Domain Expert ⚙️

**What it does:**
- Encodes known physical laws and safety constraints
- Hard-coded rules based on engineering specifications
- Validates that the system obeys physics

**How it detects attacks:**
- Checks every reading against safety bounds
- Validates relationships between sensors (if A then B)
- Enforces process logic (can't heat and cool simultaneously)

**What it catches:**
- Safety violations (temp > max rated)
- Impossible physical states (negative pressure)
- Logic violations (pump on but no flow)
- Attacks that violate known constraints

**What it misses:**
- Novel attacks within "legal" bounds
- Sophisticated attacks that respect physics
- Zero-day exploits that manipulate valid ranges

**Real example:**
```
Rule: "Reactor temperature must be 50-350°C"
Attack: Modbus write sets temp to 380°C
Physics Engine: "CRITICAL - Exceeds maximum safe temperature"
Instant detection, no learning needed
```

---

## Why You Need All Three Together

### The Correlation Magic ✨

When all three methods agree, you have **high confidence** it's a real attack, not a false positive:

```
Scenario: Attacker manipulates reactor temperature

Isolation Forest: "Anomaly score 0.85 - sudden spike detected"
LSTM: "Reconstruction error 0.92 - pattern deviation"
Physics Rules: "CRITICAL - Temperature 380°C exceeds max 350°C"

Correlation Engine: "3/3 sources agree → Confidence: 0.95 → CRITICAL ALERT"
```

### Coverage Matrix

| Attack Type | LSTM | Isolation Forest | Physics Rules |
|-------------|------|------------------|---------------|
| Sudden spike | ❌ (too fast) | ✅ (catches instantly) | ✅ (if violates bounds) |
| Gradual drift | ✅ (pattern change) | ❌ (stays in range) | ✅ (eventually violates) |
| Coordinated multi-sensor | ✅ (sees correlation) | ⚠️ (might miss) | ⚠️ (if within bounds) |
| Impossible state | ⚠️ (learns it's wrong) | ⚠️ (might be in range) | ✅ (violates physics) |
| Novel zero-day | ✅ (pattern deviation) | ✅ (if outlier) | ❌ (if within bounds) |

**Key insight:** Each method covers the others' blind spots.

---

## Real-World Attack Examples

### Example 1: Stuxnet-Style Attack
**Attack:** Slowly increase centrifuge speed over weeks

- **Isolation Forest:** ❌ Each individual reading looks normal
- **LSTM:** ✅ Detects the gradual upward trend over time
- **Physics Rules:** ✅ Eventually exceeds safe RPM threshold

**Result:** 2/3 detection → Medium confidence alert

---

### Example 2: Modbus Write Attack
**Attack:** Instantly write pressure value to 500 PSI (normal: 50-60)

- **Isolation Forest:** ✅ Immediate outlier detection (<5 sec)
- **LSTM:** ⚠️ Takes 30 seconds to recognize pattern break
- **Physics Rules:** ✅ Exceeds maximum safe pressure instantly

**Result:** 3/3 detection → High confidence CRITICAL alert

---

### Example 3: Sophisticated APT
**Attack:** Manipulate multiple sensors in coordinated way, staying within individual safe ranges

- **Isolation Forest:** ❌ Each sensor reading looks normal
- **LSTM:** ✅ Detects unusual correlation between sensors
- **Physics Rules:** ⚠️ Individual values OK, but combination is impossible

**Result:** 2/3 detection → Medium-high confidence alert

---

## Implementation Architecture

```
Sensor Data Stream (Kafka)
         ↓
    ┌────┴────┬────────────┬──────────────┐
    ↓         ↓            ↓              ↓
  LSTM    Isolation    Physics      (Other)
  Model    Forest      Rules
    ↓         ↓            ↓              ↓
    └────┬────┴────────────┴──────────────┘
         ↓
  Correlation Engine
  (Combines signals, assigns confidence)
         ↓
    Alert System
```

### Correlation Logic

```python
def correlate_detections(lstm_score, if_score, physics_violated):
    detections = []
    
    if lstm_score > 0.8:
        detections.append(("LSTM", lstm_score))
    
    if if_score > 0.7:
        detections.append(("IsolationForest", if_score))
    
    if physics_violated:
        detections.append(("PhysicsRules", 1.0))
    
    # Calculate confidence based on agreement
    confidence = len(detections) / 3.0
    
    if confidence >= 0.66:  # 2 or more agree
        severity = "CRITICAL" if confidence > 0.9 else "HIGH"
        return Alert(
            confidence=confidence,
            severity=severity,
            sources=detections
        )
    
    return None  # Not enough agreement
```

---

## Why This Reduces False Positives

**Single method:**
- LSTM alone: Might flag legitimate maintenance as anomaly
- Isolation Forest alone: Might flag sensor noise as attack
- Physics Rules alone: Might flag edge cases as violations

**Combined (2-3 methods agree):**
- Legitimate maintenance: Only LSTM flags it (1/3) → No alert
- Sensor noise: Only IF flags it (1/3) → No alert
- Real attack: All 3 flag it (3/3) → HIGH CONFIDENCE alert

**Result:** Dramatically lower false positive rate while maintaining high detection rate.

---

## Demo Impact

When you show all three working together:

1. **Proves robustness** - Not relying on single method
2. **Shows sophistication** - Defense-in-depth approach
3. **Builds confidence** - Multiple independent validators
4. **Differentiates you** - Most competitors use 1-2 methods max
5. **Reduces skepticism** - "What about false positives?" → "We correlate 3 sources"

The correlation engine is what makes this powerful - it's not just three separate detectors, it's an intelligent system that weighs evidence from multiple sources.

---

**Document Version:** 1.0  
**Last Updated:** October 14, 2025  
**Related:** demo-priorities.md
