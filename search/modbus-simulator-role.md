# Modbus Simulator - Role and Specifications

## Overview

The Modbus Simulator is a critical component that replaces real industrial equipment (PLCs, sensors, actuators) for development and demonstration purposes. It generates realistic ICS data without needing actual hardware.

---

## Primary Purpose

**Replace Real Industrial Equipment:**
- Simulates PLCs (Programmable Logic Controllers)
- Simulates sensors (temperature, pressure, flow)
- Simulates actuators (valves, pumps)
- Implements Modbus/TCP protocol
- Generates realistic industrial process data

---

## Key Responsibilities

### 1. Generate Sensor Data

**Temperature Sensors:**
- Normal range: 280-320°C
- Pattern: Sine wave with Gaussian noise
- Sampling rate: 1 Hz
- Noise: σ = 2°C

**Pressure Sensors:**
- Normal range: 100-150 psi
- Pattern: Steady state with small variations
- Sampling rate: 1 Hz
- Noise: σ = 1 psi

**Flow Rate Sensors:**
- Normal range: 50-100 L/min
- Pattern: Correlated with valve position
- Sampling rate: 1 Hz
- Noise: σ = 2 L/min

**Valve Position:**
- Range: 0-100%
- Pattern: Step changes (discrete states)
- States: 0%, 25%, 50%, 75%, 100%
- Sampling rate: 1 Hz

### 2. Simulate 3 Facilities

**Facility A:**
- Baseline: Temperature oscillates 290-310°C (sine wave)
- Pressure: 120 psi ± 5 psi
- Flow rate: 75 L/min ± 10 L/min
- Unique operational pattern

**Facility B:**
- Baseline: Temperature steady at 300°C ± 5°C
- Pressure: 130 psi ± 3 psi
- Flow rate: 80 L/min ± 8 L/min
- Different from Facility A

**Facility C:**
- Baseline: Temperature follows production schedule
- Pressure: 125 psi ± 4 psi
- Flow rate: 70 L/min ± 12 L/min
- Third unique pattern

**Purpose:**
- Represents distributed ICS environments
- Each facility has independent sensor readings
- Demonstrates federated learning across facilities
- Shows data sovereignty (data stays local)

### 3. Inject Attack Scenarios

**Attack 1: Sudden Spike (Isolation Forest Detection)**
- Type: Modbus write attack
- Action: Temperature jumps from 300°C to 380°C instantly
- Duration: Single timestep
- Detection time: < 5 seconds
- Detector: Isolation Forest

**Attack 2: Gradual Drift (LSTM Detection)**
- Type: Stealthy manipulation
- Action: Temperature increases 2°C per second
- Duration: 60 seconds (300°C → 420°C)
- Detection time: ~30 seconds
- Detector: LSTM Autoencoder

**Attack 3: Oscillation Attack**
- Type: Rapid cycling
- Action: Pump/valve on/off every 2 seconds
- Duration: 5 minutes
- Detection time: ~30 seconds
- Detector: LSTM + Physics Rules

**Attack 4: Multi-Stage Attack (GNN Prediction)**
- Stage 1: Discovery (T0846) - Network scan
- Stage 2: Lateral Movement (T0800) - Connect to PLC
- Stage 3: Program Download (T0843) - Upload malicious logic
- Purpose: Demonstrate attack prediction
- Detector: GNN predicts each next step

**Attack 5: Setpoint Manipulation**
- Type: Unauthorized control
- Action: Change temperature setpoint
- May stay within normal ranges
- Detection: Behavioral anomaly

### 4. Dual-Write Data

**Write to Kafka:**
- Topic: `sensor.data`
- Format: JSON messages
- Purpose: Real-time processing
- Consumers: Detection services

**Write to IoTDB:**
- Database: `root.facility_a`, `root.facility_b`, `root.facility_c`
- Timeseries: temperature, pressure, flow_rate, valve_position
- Purpose: Historical queries for LSTM
- Query pattern: Last 60 seconds

**Why Both?**
- Kafka: Stream processing, real-time alerts
- IoTDB: Time-series queries, LSTM training
- Different use cases, complementary

### 5. Modbus Protocol Compliance

**Protocol:** Modbus/TCP

**Port:** 502 (standard Modbus port)

**Registers:**
- Address 1000: Temperature (holding register)
- Address 1001: Pressure (holding register)
- Address 1002: Flow rate (holding register)
- Address 1003: Valve position (holding register)
- Addresses 1004-1009: Additional sensors

**Function Codes:**
- 0x03: Read Holding Registers
- 0x06: Write Single Register
- 0x10: Write Multiple Registers

**Attack Simulation:**
- Unauthorized writes to registers
- Malformed Modbus packets
- Timing attacks
- Protocol violations

---

## Data Flow

```
┌─────────────────────────────────────────────────────────┐
│              Modbus Simulator                           │
│                                                         │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐ │
│  │  Facility A  │  │  Facility B  │  │  Facility C  │ │
│  │              │  │              │  │              │ │
│  │ PLC-REACT-01 │  │ PLC-REACT-02 │  │ PLC-REACT-03 │ │
│  │              │  │              │  │              │ │
│  │ Temp: 300°C  │  │ Temp: 305°C  │  │ Temp: 295°C  │ │
│  │ Press: 120psi│  │ Press: 130psi│  │ Press: 125psi│ │
│  └──────────────┘  └──────────────┘  └──────────────┘ │
└─────────────────────────────────────────────────────────┘
         │                    │                    │
         ├────────────────────┴────────────────────┤
         │                                         │
         ↓                                         ↓
┌──────────────────┐                    ┌──────────────────┐
│  Kafka Topic     │                    │     IoTDB        │
│  sensor.data     │                    │  Time-Series DB  │
│                  │                    │                  │
│  Real-time       │                    │  Historical      │
│  Processing      │                    │  Queries         │
└──────────────────┘                    └──────────────────┘
         │                                         │
         ↓                                         ↓
┌─────────────────────────────────────────────────────────┐
│              Detection Services                         │
│                                                         │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐ │
│  │ Isolation    │  │    LSTM      │  │   Physics    │ │
│  │   Forest     │  │ Autoencoder  │  │    Rules     │ │
│  │              │  │              │  │              │ │
│  │ Latest value │  │ 60-sec window│  │ Real-time    │ │
│  │ from IoTDB   │  │ from IoTDB   │  │ from Kafka   │ │
│  └──────────────┘  └──────────────┘  └──────────────┘ │
└─────────────────────────────────────────────────────────┘
         │                    │                    │
         └────────────────────┴────────────────────┘
                              │
                              ↓
                    ┌──────────────────┐
                    │  Correlation     │
                    │     Engine       │
                    └──────────────────┘
                              │
                              ↓
                    ┌──────────────────┐
                    │     Alerts       │
                    │   PostgreSQL     │
                    └──────────────────┘
```

---

## Why It's Essential for Your Demo

### 1. No Hardware Required
**Cost Savings:**
- Real PLC: $5,000 - $50,000 each
- Sensors: $500 - $5,000 each
- Network equipment: $2,000 - $10,000
- **Total saved: $20,000 - $200,000**

**Time Savings:**
- No hardware procurement (weeks)
- No physical setup (days)
- No network configuration (days)
- **Immediate availability**

### 2. Controlled Demonstrations
**Repeatability:**
- Trigger attacks on command
- Exact same scenario every time
- Perfect timing for 3-minute demo
- No unexpected behavior

**Flexibility:**
- Modify attack parameters instantly
- Test different scenarios
- Adjust timing for demo flow
- Easy to reset and retry

### 3. Safety
**No Risk:**
- No damage to real equipment
- No safety hazards
- Can simulate dangerous scenarios
- Test extreme attack conditions

**Examples of Safe Testing:**
- Overpressure scenarios (would damage real equipment)
- Overtemperature scenarios (fire risk in real systems)
- Rapid cycling (mechanical wear in real systems)
- Simultaneous failures (catastrophic in real systems)

### 4. Development Speed
**Rapid Iteration:**
- Immediate data availability
- No hardware setup time
- Easy to modify scenarios
- Fast debugging

**Parallel Development:**
- Multiple developers can work simultaneously
- Each can run their own simulator
- No shared hardware conflicts
- Independent testing

### 5. Federated Learning Demo
**Multi-Facility Simulation:**
- Simulates 3 independent facilities
- Shows data staying local
- Demonstrates collaborative learning
- Proves federated learning concept

**Key Demo Points:**
- Facility A learns from attack
- FL round distributes knowledge
- Facilities B & C now protected
- No data sharing required

---

## Output Data Formats

### Kafka Message Format

```json
{
  "timestamp": "2025-10-14T14:23:45.123Z",
  "facility_id": "facility_a",
  "device": "PLC-REACTOR-01",
  "sensors": {
    "temperature": 305.2,
    "pressure": 122.5,
    "flow_rate": 78.3,
    "valve_position": 75
  },
  "metadata": {
    "unit_id": 1,
    "register_base": 1000
  }
}
```

### IoTDB Insert Format

```sql
INSERT INTO root.facility_a.reactor(timestamp, temperature, pressure)
VALUES (1697289825123, 305.2, 122.5);

INSERT INTO root.facility_a.pump(timestamp, flow_rate)
VALUES (1697289825123, 78.3);

INSERT INTO root.facility_a.valve(timestamp, position)
VALUES (1697289825123, 75);
```

### Modbus Protocol Format

**Read Holding Registers (Function Code 0x03):**
```
Request:
[Transaction ID][Protocol ID][Length][Unit ID][Function Code][Start Address][Quantity]
[0x0001][0x0000][0x0006][0x01][0x03][0x03E8][0x000A]

Response:
[Transaction ID][Protocol ID][Length][Unit ID][Function Code][Byte Count][Register Values...]
[0x0001][0x0000][0x0017][0x01][0x03][0x14][0x0131][0x007A][0x004E][...]
```

**Write Single Register (Function Code 0x06):**
```
Request (Attack):
[Transaction ID][Protocol ID][Length][Unit ID][Function Code][Register Address][Register Value]
[0x0002][0x0000][0x0006][0x01][0x06][0x03E8][0x017C]  # Write 380 to temp register

Response:
[Transaction ID][Protocol ID][Length][Unit ID][Function Code][Register Address][Register Value]
[0x0002][0x0000][0x0006][0x01][0x06][0x03E8][0x017C]  # Echo back
```

---

## Configuration

### Simulator Configuration File

```yaml
facilities:
  - id: facility_a
    name: "Power Plant A"
    devices:
      - name: PLC-REACTOR-01
        modbus_address: "192.168.1.10"
        modbus_port: 502
        unit_id: 1
        registers:
          - address: 1000
            name: temperature
            normal_range: [280, 320]
            unit: celsius
            pattern: sine_wave
            frequency: 0.01  # Hz
            noise: 2.0
          
          - address: 1001
            name: pressure
            normal_range: [100, 150]
            unit: psi
            pattern: steady_state
            noise: 1.0
          
          - address: 1002
            name: flow_rate
            normal_range: [50, 100]
            unit: liters_per_minute
            pattern: correlated
            correlation_with: valve_position
            noise: 2.0
          
          - address: 1003
            name: valve_position
            normal_range: [0, 100]
            unit: percent
            pattern: step_changes
            states: [0, 25, 50, 75, 100]
    
    scenarios:
      - name: normal_operation
        duration: 300  # seconds
        description: "Normal operation baseline"
      
      - name: sudden_spike_attack
        trigger: manual
        attack_type: write_register
        target_register: 1000
        attack_value: 380
        description: "Sudden temperature spike to 380°C"
      
      - name: gradual_drift_attack
        trigger: manual
        attack_type: gradual_change
        target_register: 1000
        start_value: 300
        end_value: 420
        duration: 60
        description: "Gradual temperature increase over 60 seconds"
      
      - name: multi_stage_attack
        trigger: manual
        stages:
          - technique: T0846
            duration: 30
            description: "Network discovery scan"
          - technique: T0800
            duration: 30
            description: "Lateral movement to PLC"
          - technique: T0843
            duration: 30
            description: "Malicious program download"

  - id: facility_b
    # Similar structure for Facility B
    
  - id: facility_c
    # Similar structure for Facility C

kafka:
  bootstrap_servers: "kafka:9092"
  topic: "sensor.data"
  
iotdb:
  host: "iotdb"
  port: 6667
  username: "root"
  password: "root"
```

---

## Demo Scenarios

### Scenario 1: Multi-Layered Detection + FL

**Simulator Actions:**
1. Run normal operation for 30 seconds (all 3 facilities)
2. Inject sudden spike attack on Facility A
   - Temperature: 300°C → 380°C
3. Wait for detections (30 seconds)
4. Trigger FL round (via API call)
5. Continue normal operation during FL training (5 minutes)
6. Inject same attack on Facility B
7. Show immediate detection

**Expected Output:**
- Facility A: 3 alerts (LSTM, IF, Physics)
- FL round: Model updated and distributed
- Facility B: Immediate detection (< 5 seconds)

### Scenario 2: Attack Prediction

**Simulator Actions:**
1. Run normal operation
2. Inject multi-stage attack on Facility A
   - Stage 1: Discovery (T0846) - 30 seconds
   - Wait for prediction
   - Stage 2: Lateral Movement (T0800) - 30 seconds
   - Wait for prediction
   - Stage 3: Program Download (T0843) - 30 seconds
3. Show prediction accuracy

**Expected Output:**
- After T0846: GNN predicts T0800 (67% probability)
- After T0800: GNN predicts T0843 (72% probability)
- Predictions validated as attack proceeds

---

## Alternative: Real Data Playback

### Instead of Simulation

**Option 1: PCAP Replay**
- Play back Morris & Gao dataset
- Use tcpreplay for network traffic
- Less control, more realistic

**Option 2: CSV Playback**
- Read sensor data from CSV files
- Replay at original timestamps
- Good for validation

**Option 3: Hybrid Approach**
- Use real data for normal operation
- Use simulator for attacks
- Best of both worlds

### Why Simulation is Better for Demo

**Advantages:**
- ✅ Perfect timing control
- ✅ Repeatable scenarios
- ✅ On-demand attacks
- ✅ Easy to modify
- ✅ No external dependencies

**Disadvantages:**
- ❌ May not capture all real-world complexity
- ❌ Requires domain knowledge
- ❌ Need validation with real data

---

## Implementation Technologies

### Recommended Libraries

**Python:**
- `pymodbus` - Modbus protocol implementation
- `kafka-python` - Kafka producer
- `iotdb-session` - IoTDB client
- `numpy` - Data generation
- `scipy` - Signal processing

**Alternative:**
- `OpenPLC` - Open-source PLC simulator
- `ModbusPal` - Java-based Modbus simulator
- `pycomm3` - For other protocols (future)

---

## Summary

### The Modbus Simulator is Essential Because:

1. ✅ **Replaces expensive hardware** ($20K-$200K saved)
2. ✅ **Generates realistic ICS data** (temperature, pressure, flow)
3. ✅ **Enables controlled attack scenarios** (sudden spike, gradual drift, multi-stage)
4. ✅ **Supports 3-facility federated learning demo** (distributed simulation)
5. ✅ **Writes to both Kafka and IoTDB** (dual-write for different use cases)
6. ✅ **Makes demo reproducible and safe** (no risk to real equipment)
7. ✅ **Implements Modbus/TCP protocol** (realistic protocol-level attacks)
8. ✅ **Provides immediate data availability** (no procurement delays)

### Without It:

- ❌ Need real PLCs and sensors
- ❌ Expensive hardware investment
- ❌ Time-consuming setup
- ❌ Safety risks during attacks
- ❌ Limited attack scenarios
- ❌ Difficult to repeat demos
- ❌ Hardware maintenance required

---

**Document Version:** 1.0  
**Last Updated:** October 14, 2025  
**Status:** Ready for Implementation
