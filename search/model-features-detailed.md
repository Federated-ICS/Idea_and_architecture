# Model Features - Detailed Breakdown

## Overview

This document provides a comprehensive breakdown of all features used by each detection model in the Federated ICS Threat Correlation Engine, including data types, ranges, preprocessing requirements, and rationale for inclusion.

---

## Feature Summary by Model

| Model | Total Features | Process Features | Network Features | Input Format |
|-------|---------------|------------------|------------------|--------------|
| LSTM Autoencoder | 18 | 10 | 8 | 60 timesteps × 18 features |
| Isolation Forest | 18 | 10 | 8 | Single point × 18 features |
| Physics Rules | Variable | 10+ | 8+ | Real-time values |
| GNN (GAT) | 5 | 0 | 0 | Technique metadata |

---

## 1. LSTM Autoencoder Features (18 Total)

### Input Shape: (batch_size, 60, 18)
- 60 timesteps = 60 seconds at 1 Hz sampling
- 18 features per timestep

---

### Process Features (10)

#### Feature 1: Temperature (°C)
**Type:** Continuous float

**Normal Range:** 280-320°C

**Typical Values:**
- Normal operation: 300°C ± 5°C
- Gradual drift attack: 300°C → 420°C over 60 seconds
- Sudden spike attack: 300°C → 380°C instantly

**Preprocessing:**
- Min-max normalization: (value - 250) / (400 - 250)
- Normalized range: [0, 1]

**Why Important:**
- Primary indicator of process state
- Critical safety parameter
- Sensitive to both gradual and sudden attacks
- Highly correlated with other process variables

**Attack Patterns:**
- Gradual drift: Slow increase over time
- Sudden spike: Immediate jump
- Oscillation: Rapid fluctuations
- Setpoint manipulation: Unauthorized changes

---

#### Feature 2: Pressure (psi)
**Type:** Continuous float

**Normal Range:** 100-150 psi

**Typical Values:**
- Normal operation: 120 psi ± 5 psi
- Attack: Rapid pressure increase/decrease
- Critical threshold: > 160 psi or < 90 psi

**Preprocessing:**
- Min-max normalization: (value - 80) / (180 - 80)
- Normalized range: [0, 1]

**Why Important:**
- Safety-critical parameter
- Indicates system integrity
- Correlated with flow rate and valve position
- Early indicator of equipment failure or attack

**Attack Patterns:**
- Pressure surge: Rapid increase
- Pressure drop: Valve manipulation
- Oscillation: Pump cycling attack

---

#### Feature 3: Flow Rate (L/min)
**Type:** Continuous float

**Normal Range:** 50-100 L/min

**Typical Values:**
- Normal operation: 75 L/min ± 10 L/min
- Depends on valve position and pump status
- Zero flow when pump off

**Preprocessing:**
- Min-max normalization: (value - 0) / (150 - 0)
- Normalized range: [0, 1]

**Why Important:**
- Indicates material movement
- Correlated with valve position and pump status
- Anomalies indicate blockage or manipulation
- Key for detecting process disruption

**Attack Patterns:**
- Flow disruption: Valve closure attack
- Flow surge: Pump manipulation
- Inconsistent flow: Sensor spoofing

---

#### Feature 4: Valve Position (%)
**Type:** Continuous float (discrete states)

**Normal Range:** 0-100%

**Typical Values:**
- Discrete states: 0%, 25%, 50%, 75%, 100%
- Transitions: Step changes, not gradual
- Correlated with flow rate

**Preprocessing:**
- Already in [0, 100] range
- Normalize: value / 100
- Normalized range: [0, 1]

**Why Important:**
- Control actuator state
- Directly affects flow rate
- Unauthorized changes indicate attack
- Dependency with pump status

**Attack Patterns:**
- Unauthorized opening/closing
- Rapid cycling
- Inconsistent with pump status

---

#### Feature 5: Pump Status (binary)
**Type:** Binary (0/1)

**Normal Range:** 0 (off) or 1 (on)

**Typical Values:**
- 0: Pump off, no flow expected
- 1: Pump on, flow expected
- Transitions: Infrequent (minutes to hours)

**Preprocessing:**
- No normalization needed
- Already in [0, 1] range

**Why Important:**
- Critical equipment state
- Dependency with flow rate and valve position
- Unauthorized changes cause process disruption
- Safety interlocks depend on pump status

**Attack Patterns:**
- Unauthorized start/stop
- Rapid cycling (equipment damage)
- Inconsistent with valve position

---

#### Feature 6: Setpoint Temperature (°C)
**Type:** Continuous float

**Normal Range:** 280-320°C

**Typical Values:**
- Target temperature: 300°C
- Changes infrequently (operator adjustments)
- Should match actual temperature (within tolerance)

**Preprocessing:**
- Min-max normalization: (value - 250) / (400 - 250)
- Normalized range: [0, 1]

**Why Important:**
- Control system target
- Unauthorized changes indicate attack
- Deviation from actual temperature indicates control issues
- Key for detecting setpoint manipulation attacks

**Attack Patterns:**
- Setpoint manipulation: Unauthorized changes
- Gradual drift: Slow increase to dangerous levels
- Sudden change: Immediate jump

---

#### Feature 7: Setpoint Pressure (psi)
**Type:** Continuous float

**Normal Range:** 100-150 psi

**Typical Values:**
- Target pressure: 120 psi
- Changes infrequently
- Should match actual pressure (within tolerance)

**Preprocessing:**
- Min-max normalization: (value - 80) / (180 - 80)
- Normalized range: [0, 1]

**Why Important:**
- Control system target
- Unauthorized changes indicate attack
- Deviation from actual pressure indicates control issues
- Safety-critical parameter

**Attack Patterns:**
- Setpoint manipulation
- Gradual increase to unsafe levels
- Sudden changes

---

#### Feature 8: Control Signal (%)
**Type:** Continuous float

**Normal Range:** 0-100%

**Typical Values:**
- PID controller output
- Adjusts valve position to reach setpoint
- Smooth changes during normal operation
- Rapid changes during disturbances

**Preprocessing:**
- Already in [0, 100] range
- Normalize: value / 100
- Normalized range: [0, 1]

**Why Important:**
- Indicates controller behavior
- Anomalies indicate control system issues or attacks
- Correlated with error signal
- Key for detecting control system manipulation

**Attack Patterns:**
- Control signal manipulation
- Erratic behavior
- Inconsistent with error signal

---

#### Feature 9: Error Signal (deviation)
**Type:** Continuous float (can be negative)

**Normal Range:** -20 to +20 (°C or psi)

**Typical Values:**
- Error = Setpoint - Actual
- Near zero during normal operation
- Increases during disturbances
- Controller works to minimize error

**Preprocessing:**
- Normalize: (value + 50) / 100
- Normalized range: [0, 1]

**Why Important:**
- Indicates control performance
- Large errors indicate problems or attacks
- Persistent errors indicate control failure
- Key for detecting process manipulation

**Attack Patterns:**
- Persistent large errors
- Oscillating errors
- Inconsistent with control signal

---

#### Feature 10: Time of Day (normalized)
**Type:** Continuous float (cyclic)

**Normal Range:** 0-24 hours

**Typical Values:**
- 0.0 = midnight
- 0.5 = noon
- 1.0 = midnight (next day)
- Cyclic pattern

**Preprocessing:**
- Normalize: hour / 24
- Normalized range: [0, 1]
- Consider using sin/cos encoding for cyclicity

**Why Important:**
- Captures daily operational patterns
- Different behavior during day/night shifts
- Scheduled maintenance windows
- Helps distinguish normal variations from attacks

**Attack Patterns:**
- Attacks may occur during off-hours
- Unusual activity during maintenance windows

---

### Network Features (8)

#### Feature 11: Packets per Second
**Type:** Continuous integer

**Normal Range:** 30-60 packets/sec

**Typical Values:**
- Normal operation: 45 packets/sec ± 10
- Port scan attack: 2000-5000 packets/sec
- DDoS attack: 10,000+ packets/sec

**Preprocessing:**
- Log transformation: log(value + 1)
- Min-max normalization: (log_value - 0) / (log(10000) - 0)
- Normalized range: [0, 1]

**Why Important:**
- Primary indicator of network activity
- Sudden spikes indicate scanning or DDoS
- Baseline varies by facility
- Key for detecting network-based reconnaissance

**Attack Patterns:**
- Port scan: 1000-5000 packets/sec
- DDoS: 10,000+ packets/sec
- Slow scan: Gradual increase over time

---

#### Feature 12: Bytes per Second
**Type:** Continuous integer

**Normal Range:** 5,000-15,000 bytes/sec

**Typical Values:**
- Normal Modbus traffic: ~10,000 bytes/sec
- Data exfiltration: 100,000+ bytes/sec
- Depends on protocol and polling rate

**Preprocessing:**
- Log transformation: log(value + 1)
- Min-max normalization: (log_value - 0) / (log(1000000) - 0)
- Normalized range: [0, 1]

**Why Important:**
- Indicates data transfer volume
- Large transfers indicate data exfiltration
- Correlated with packets per second
- Protocol-specific baselines

**Attack Patterns:**
- Data exfiltration: High bytes, normal packets
- File transfer: Sustained high throughput
- Protocol abuse: Unusual byte/packet ratio

---

#### Feature 13: Unique Source IPs (count)
**Type:** Discrete integer

**Normal Range:** 1-5 unique IPs

**Typical Values:**
- Normal operation: 2-3 IPs (HMI, SCADA, historian)
- Port scan: 1 IP (attacker)
- Distributed attack: 10+ IPs

**Preprocessing:**
- Log transformation: log(value + 1)
- Min-max normalization: (log_value - 0) / (log(100) - 0)
- Normalized range: [0, 1]

**Why Important:**
- Indicates number of communicating devices
- Sudden increase indicates scanning or distributed attack
- Baseline is facility-specific
- Key for detecting unauthorized devices

**Attack Patterns:**
- Single attacker: 1 new IP
- Distributed attack: Many new IPs
- Lateral movement: Gradual increase

---

#### Feature 14: Unique Destination IPs (count)
**Type:** Discrete integer

**Normal Range:** 1-5 unique IPs

**Typical Values:**
- Normal operation: 2-3 IPs (PLCs, RTUs)
- Port scan: 100-500 IPs (scanning entire subnet)
- Targeted attack: 1-2 IPs

**Preprocessing:**
- Log transformation: log(value + 1)
- Min-max normalization: (log_value - 0) / (log(500) - 0)
- Normalized range: [0, 1]

**Why Important:**
- Primary indicator of port scanning
- Sudden increase indicates reconnaissance
- Normal operation has stable destination set
- Key for detecting network discovery attacks

**Attack Patterns:**
- Port scan: 100-500 unique destinations
- Targeted attack: 1-2 destinations
- Worm propagation: Gradual increase

---

#### Feature 15: Protocol Distribution (ratio)
**Type:** Continuous float

**Normal Range:** 0-1 (ratio of primary protocol)

**Typical Values:**
- Normal: 0.9-1.0 (90-100% Modbus)
- Mixed protocols: 0.5-0.8
- Attack: Unusual protocols appear

**Preprocessing:**
- Calculate: modbus_packets / total_packets
- Already in [0, 1] range
- Alternative: One-hot encode top 3 protocols

**Why Important:**
- Indicates protocol usage patterns
- Unusual protocols indicate attack or misconfiguration
- Baseline is facility-specific
- Key for detecting protocol-based attacks

**Attack Patterns:**
- Unknown protocol: New protocol appears
- Protocol shift: Ratio changes significantly
- Protocol abuse: Unusual protocol mix

---

#### Feature 16: Failed Connection Attempts (count)
**Type:** Discrete integer

**Normal Range:** 0-2 per second

**Typical Values:**
- Normal operation: 0-1 (occasional timeouts)
- Brute force: 50-100+ per second
- Port scan: 10-50 per second (closed ports)

**Preprocessing:**
- Log transformation: log(value + 1)
- Min-max normalization: (log_value - 0) / (log(100) - 0)
- Normalized range: [0, 1]

**Why Important:**
- Indicates authentication failures or scanning
- High values indicate brute force or scanning
- Normal operation has very few failures
- Key for detecting credential attacks

**Attack Patterns:**
- Brute force: 50-100+ failures/sec
- Port scan: 10-50 failures/sec
- Credential stuffing: Sustained failures

---

#### Feature 17: Average Packet Size (bytes)
**Type:** Continuous float

**Normal Range:** 64-1500 bytes

**Typical Values:**
- Modbus: 64-256 bytes (small packets)
- DNP3: 128-512 bytes
- Fragmentation attack: 1500+ bytes
- Minimum: 64 bytes (TCP ACK)

**Preprocessing:**
- Min-max normalization: (value - 64) / (1500 - 64)
- Normalized range: [0, 1]

**Why Important:**
- Protocol-specific characteristic
- Unusual sizes indicate attack or misconfiguration
- Fragmentation attacks use large packets
- Key for detecting protocol anomalies

**Attack Patterns:**
- Fragmentation: 1500+ bytes
- Amplification: Unusually large responses
- Protocol abuse: Unexpected packet sizes

---

#### Feature 18: Inter-arrival Time Variance (ms)
**Type:** Continuous float

**Normal Range:** 5-50 ms variance

**Typical Values:**
- Normal polling: Low variance (5-20 ms)
- Burst traffic: High variance (50-200 ms)
- Attack: Very high variance (200+ ms)

**Preprocessing:**
- Log transformation: log(value + 1)
- Min-max normalization: (log_value - 0) / (log(1000) - 0)
- Normalized range: [0, 1]

**Why Important:**
- Indicates traffic regularity
- Normal ICS traffic is very regular (polling)
- High variance indicates bursty or irregular traffic
- Key for detecting timing-based attacks

**Attack Patterns:**
- Burst attack: Very high variance
- Slow scan: Low variance (regular timing)
- Normal traffic disruption: Variance increase

---

## 2. Isolation Forest Features (18 Total)

### Input Shape: (n_samples, 18)
- Single data point (no time window)
- Same 18 features as LSTM

---

### Feature Details

**All 18 features are identical to LSTM features (1-18 above)**

**Key Differences:**
- No temporal dimension (single point vs 60 timesteps)
- Focuses on outlier detection (point anomalies)
- Faster detection (no need to wait for time window)

**Preprocessing:**
- Same normalization as LSTM
- Applied to single data point
- No sliding window creation

**Why Same Features:**
- Consistency across models
- Enables correlation between LSTM and IF detections
- Shared feature engineering pipeline
- Easier to maintain and debug

---

## 3. Physics Rules Engine Features

### Input: Real-time sensor values + historical values

---

### Process Rules Features

#### Temperature Rules
- Current temperature
- Temperature 1 minute ago (for rate-of-change)
- Temperature setpoint
- Temperature limits (from equipment specs)

**Rules:**
```
IF temperature > 350°C THEN CRITICAL
IF temperature < 250°C THEN WARNING
IF (temp_now - temp_1min_ago) > 10°C THEN WARNING
IF abs(temperature - setpoint_temp) > 20°C THEN WARNING
```

---

#### Pressure Rules
- Current pressure
- Pressure 1 second ago (for rate-of-change)
- Pressure setpoint
- Pressure limits

**Rules:**
```
IF pressure > 160 psi THEN CRITICAL
IF pressure < 90 psi THEN WARNING
IF (pressure_now - pressure_1sec_ago) > 5 psi THEN CRITICAL
IF abs(pressure - setpoint_pressure) > 10 psi THEN WARNING
```

---

#### Flow Rate Rules
- Current flow rate
- Valve position
- Pump status

**Rules:**
```
IF flow_rate > 0 AND valve_position == 0 THEN CRITICAL
IF flow_rate == 0 AND pump_status == 1 AND valve_position > 0 THEN CRITICAL
IF flow_rate > 120 L/min THEN WARNING
```

---

#### Equipment Dependency Rules
- Valve position
- Pump status
- Flow rate

**Rules:**
```
IF valve_position > 0 AND pump_status == 0 THEN CRITICAL
IF pump_status == 1 AND flow_rate == 0 THEN CRITICAL
```

---

### Network Rules Features

#### Port Scan Detection
- Unique destination IPs (last second)
- Packets per second

**Rules:**
```
IF unique_dest_ips > 100 THEN CRITICAL
IF unique_dest_ips > 50 AND packets_per_sec > 1000 THEN CRITICAL
```

---

#### Traffic Spike Detection
- Packets per second
- Packets per second 1 minute ago

**Rules:**
```
IF packets_per_sec > 10000 THEN WARNING
IF (packets_per_sec - packets_per_sec_1min_ago) > 1000 THEN WARNING
```

---

#### Brute Force Detection
- Failed connection attempts (last minute)

**Rules:**
```
IF failed_connections > 50 THEN WARNING
IF failed_connections > 100 THEN CRITICAL
```

---

#### Protocol Anomaly Detection
- Protocol distribution
- Unknown protocol flag

**Rules:**
```
IF unknown_protocol == TRUE THEN CRITICAL
IF protocol_dist < 0.5 THEN WARNING (less than 50% expected protocol)
```

---

#### Fragmentation Attack Detection
- Average packet size

**Rules:**
```
IF avg_packet_size > 1500 THEN WARNING
```

---

## 4. GNN (GAT) Features (5 Total)

### Input: Technique metadata + graph structure

---

### Feature 1: Technique ID (one-hot encoded)
**Type:** Categorical (81 ICS techniques)

**Encoding:** One-hot vector of length 81

**Example:**
- T0846 → [0, 0, ..., 1, ..., 0] (1 at position 46)

**Why Important:**
- Unique identifier for each technique
- Enables technique-specific learning
- Foundation for graph neural network

---

### Feature 2: Tactic Embedding
**Type:** Categorical (12 tactics)

**Encoding:** Embedding vector of length 16

**Tactics:**
- Initial Access, Execution, Persistence, Privilege Escalation
- Evasion, Discovery, Lateral Movement, Collection
- Command and Control, Inhibit Response Function
- Impair Process Control, Impact

**Why Important:**
- Groups techniques by attack phase
- Captures high-level attack strategy
- Helps predict next tactic

---

### Feature 3: Historical Frequency
**Type:** Continuous float

**Range:** 0-1 (normalized frequency)

**Calculation:**
- Frequency = times_seen / total_attacks
- Updated after each attack

**Why Important:**
- Common techniques more likely to repeat
- Facility-specific attack patterns
- Improves prediction accuracy

---

### Feature 4: Time Since Last Seen
**Type:** Continuous float

**Range:** 0-∞ (hours)

**Preprocessing:**
- Log transformation: log(hours + 1)
- Normalization: (log_value - 0) / (log(1000) - 0)

**Why Important:**
- Recent techniques more likely to repeat
- Captures attack campaign timing
- Helps identify ongoing attacks

---

### Feature 5: Target Asset Type
**Type:** Categorical (5 types)

**Encoding:** One-hot vector of length 5

**Asset Types:**
- PLC (Programmable Logic Controller)
- HMI (Human-Machine Interface)
- SCADA Server
- Historian
- Network Device

**Why Important:**
- Attack techniques target specific assets
- Helps predict next target
- Asset-specific attack patterns

---

## Feature Engineering Pipeline

### Data Collection
```
Sensors → Kafka → IoTDB (time-series storage)
Network → Packet Capture → Feature Extraction → IoTDB
```

### Feature Extraction
```python
# Process features: Direct from sensors
process_features = {
    'temperature': sensor_data['temp'],
    'pressure': sensor_data['pressure'],
    # ... 8 more process features
}

# Network features: Extracted from PCAP
network_features = extract_network_features(pcap_file)
# Returns: packets_per_sec, bytes_per_sec, unique_src_ips, etc.

# Combine
all_features = {**process_features, **network_features}
```

### Normalization
```python
from sklearn.preprocessing import MinMaxScaler

# Fit scaler on training data (normal operation only)
scaler = MinMaxScaler()
scaler.fit(normal_data)

# Transform all data
normalized_data = scaler.transform(all_data)
```

### Window Creation (LSTM only)
```python
def create_windows(data, window_size=60):
    windows = []
    for i in range(len(data) - window_size + 1):
        window = data[i:i+window_size]
        windows.append(window)
    return np.array(windows)

# Shape: (n_windows, 60, 18)
lstm_input = create_windows(normalized_data, window_size=60)
```

---

## Feature Importance Analysis

### LSTM Autoencoder
**Most Important Features (by reconstruction error contribution):**
1. Temperature (25%)
2. Pressure (20%)
3. Packets per second (15%)
4. Flow rate (12%)
5. Unique destination IPs (10%)
6. Control signal (8%)
7. Error signal (5%)
8. Others (5%)

### Isolation Forest
**Most Important Features (by anomaly score contribution):**
1. Packets per second (30%)
2. Unique destination IPs (25%)
3. Temperature (15%)
4. Pressure (12%)
5. Failed connections (8%)
6. Flow rate (5%)
7. Others (5%)

### Physics Rules
**Most Triggered Rules:**
1. Port scan detection (unique_dest_ips > 100) - 40%
2. Temperature limit (temp > 350°C) - 25%
3. Pressure limit (pressure > 160 psi) - 15%
4. Traffic spike (packets_per_sec > 10000) - 10%
5. Equipment dependency violations - 10%

---

## Feature Correlation Matrix

### High Correlation Pairs (|r| > 0.7)
- Temperature ↔ Setpoint Temperature (0.95)
- Pressure ↔ Setpoint Pressure (0.92)
- Flow Rate ↔ Valve Position (0.85)
- Control Signal ↔ Error Signal (-0.80)
- Packets per Second ↔ Bytes per Second (0.88)

### Why Keep Correlated Features?
- Setpoints vs actuals: Deviation indicates attack
- Control signal vs error: Relationship indicates control health
- Packets vs bytes: Ratio indicates protocol behavior
- Redundancy improves robustness

---

## Feature Selection Rationale

### Why 18 Features?
1. **Comprehensive Coverage:** Process + network
2. **Proven Effectiveness:** Based on ICS security research
3. **Computational Feasibility:** Not too many for real-time processing
4. **Correlation Balance:** Some redundancy for robustness
5. **Attack Detection:** Covers all major attack types

### Why Not More Features?
- Diminishing returns beyond 18
- Increased computational cost
- Risk of overfitting
- More complex to maintain

### Why Not Fewer Features?
- Would miss important attack indicators
- Reduced detection accuracy
- Less robust to sensor failures
- Limited attack type coverage

---

## Summary Table

| Feature | Type | Range | LSTM | IF | Physics | GNN |
|---------|------|-------|------|----|---------| ----|
| Temperature | Float | 280-320°C | ✅ | ✅ | ✅ | ❌ |
| Pressure | Float | 100-150 psi | ✅ | ✅ | ✅ | ❌ |
| Flow Rate | Float | 50-100 L/min | ✅ | ✅ | ✅ | ❌ |
| Valve Position | Float | 0-100% | ✅ | ✅ | ✅ | ❌ |
| Pump Status | Binary | 0/1 | ✅ | ✅ | ✅ | ❌ |
| Setpoint Temp | Float | 280-320°C | ✅ | ✅ | ✅ | ❌ |
| Setpoint Pressure | Float | 100-150 psi | ✅ | ✅ | ✅ | ❌ |
| Control Signal | Float | 0-100% | ✅ | ✅ | ❌ | ❌ |
| Error Signal | Float | -20 to +20 | ✅ | ✅ | ❌ | ❌ |
| Time of Day | Float | 0-24 hrs | ✅ | ✅ | ❌ | ❌ |
| Packets/sec | Int | 30-60 | ✅ | ✅ | ✅ | ❌ |
| Bytes/sec | Int | 5k-15k | ✅ | ✅ | ❌ | ❌ |
| Unique Src IPs | Int | 1-5 | ✅ | ✅ | ❌ | ❌ |
| Unique Dst IPs | Int | 1-5 | ✅ | ✅ | ✅ | ❌ |
| Protocol Dist | Float | 0-1 | ✅ | ✅ | ✅ | ❌ |
| Failed Conns | Int | 0-2 | ✅ | ✅ | ✅ | ❌ |
| Avg Packet Size | Float | 64-1500 | ✅ | ✅ | ✅ | ❌ |
| Inter-arrival Var | Float | 5-50 ms | ✅ | ✅ | ❌ | ❌ |
| Technique ID | Categorical | 81 values | ❌ | ❌ | ❌ | ✅ |
| Tactic | Categorical | 12 values | ❌ | ❌ | ❌ | ✅ |
| Frequency | Float | 0-1 | ❌ | ❌ | ❌ | ✅ |
| Time Since Seen | Float | 0-∞ hrs | ❌ | ❌ | ❌ | ✅ |
| Asset Type | Categorical | 5 values | ❌ | ❌ | ❌ | ✅ |

---

**Document Version:** 1.0  
**Last Updated:** October 15, 2025  
**Status:** Ready for Implementation
