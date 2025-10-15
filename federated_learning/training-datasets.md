# Training Datasets for ICS Threat Detection System

## Overview

This document provides a comprehensive list of datasets for training the Federated ICS Threat Correlation Engine, including sources for LSTM Autoencoder, Isolation Forest, Physics Rules, and GNN attack prediction models.

---

## 1. ICS/SCADA Datasets

### Morris & Gao ICS Dataset (Recommended for MVP)

**Source:** Mississippi State University

**Content:**
- Network traffic from gas pipeline and water storage tank systems
- Real industrial control system operations
- Both normal and attack scenarios

**Attacks Included:**
- 36 different attack scenarios
- MITM (Man-in-the-Middle)
- DoS (Denial of Service)
- Reconnaissance attacks
- Command injection
- Response injection

**Protocols:**
- Modbus/TCP

**Size:** ~2GB

**Access:** Free, direct download

**Link:** https://sites.google.com/a/uah.edu/tommy-morris-uah/ics-data-sets

**Best For:**
- LSTM training (temporal patterns)
- Isolation Forest validation
- Modbus protocol analysis

---

### SWaT (Secure Water Treatment) ⭐ NETWORK DATA AVAILABLE

**Source:** iTrust, Singapore University of Technology and Design

**Content:**
- 11 days of continuous operation
- 7 days normal operation
- 4 days under attack
- Real water treatment testbed
- **Network traffic logs** ✅

**Sensors:**
- 51 sensors and actuators
- Temperature, pressure, flow, level sensors
- Pump and valve actuators

**Network Features Available:**
- Network logs
- Modbus/CIP traffic
- Connection patterns
- Protocol anomalies

**Attacks Included:**
- 36 attack scenarios
- Single-stage and multi-stage attacks
- Physical process attacks
- Network attacks (reconnaissance, DoS)

**Size:** ~40GB

**Access:** Free, requires registration and data use agreement

**Link:** https://itrust.sutd.edu.sg/itrust-labs_datasets/dataset_info/

**Best For:**
- LSTM training (large dataset)
- Multi-stage attack sequences
- Physics rules validation
- Network + process correlation
- Long-term behavioral analysis

---

### WADI (Water Distribution) ⭐ NETWORK DATA AVAILABLE

**Source:** iTrust, Singapore University of Technology and Design

**Content:**
- 16 days of operation
- 14 days normal operation
- 2 days under attack
- Water distribution network
- **Network traffic logs** ✅

**Sensors:**
- 123 sensors and actuators
- More complex than SWaT
- Multiple interconnected processes

**Network Features Available:**
- Network logs
- Protocol traffic
- Connection patterns
- Multi-segment network data

**Attacks Included:**
- 15 attack scenarios
- Coordinated attacks
- Stealthy attacks
- Network-based reconnaissance

**Size:** ~50GB

**Access:** Free, requires registration

**Link:** https://itrust.sutd.edu.sg/itrust-labs_datasets/dataset_info/

**Best For:**
- Advanced LSTM training
- Complex attack patterns
- Federated learning scenarios (multiple processes)
- Network + process correlation
- Multi-segment network analysis

---

### HAI (Hardware-in-the-Loop Augmented ICS) ⭐ RECOMMENDED FOR NETWORK DATA

**Source:** iTrust, Singapore University of Technology and Design

**Content:**
- 4 industrial processes (boiler, turbine, pump, water treatment)
- **Network traffic (PCAP files)** ✅
- **Process sensor data** ✅
- 38 attack scenarios
- Both normal and attack operations

**Network Features Available:**
- Packet captures (PCAP format)
- Protocol data (Modbus, CIP/EtherNet/IP)
- Packet size, inter-arrival time
- Source/destination IPs
- Protocol distribution
- Connection attempts

**Process Features Available:**
- Temperature, pressure, flow sensors
- Valve positions, pump status
- Setpoint values
- Control signals

**Attacks Included:**
- 38 attack scenarios
- Network-based attacks (port scans, DoS)
- Process manipulation attacks
- Multi-stage attacks
- Cyber-physical attacks

**Protocols:**
- Modbus/TCP
- EtherNet/IP (CIP)

**Size:** ~15GB

**Access:** Free, GitHub repository

**Link:** https://github.com/icsdataset/hai

**Best For:**
- **Network + process correlation** ⭐
- Training LSTM with network features
- Training Isolation Forest with network features
- Multi-protocol training
- Cyber-physical attack detection
- Diverse attack scenarios

**Why Best for Your Project:**
- Contains BOTH network traffic AND sensor data
- 38 different attacks (more than most datasets)
- Well-documented and maintained
- Easy to access (GitHub)
- Recent dataset (2020)
- Perfect for 18-feature model (10 process + 8 network)

---

## 2. Network-Specific ICS Datasets

### TON_IoT Dataset

**Source:** UNSW Canberra Cyber

**Content:**
- IoT and ICS network traffic
- **Full packet captures (PCAP)** ✅
- Telemetry data
- 9 attack types

**Network Features Available:**
- Packet size
- Inter-arrival time
- Protocol distribution
- Source/destination IPs and ports
- Bytes per second
- Packets per second
- Connection attempts
- Failed connections

**Attacks Included:**
- DDoS attacks
- Ransomware
- Backdoor
- Injection attacks
- Scanning
- XSS
- Password attacks
- MITM

**Size:** ~30GB

**Access:** Free, requires registration

**Link:** https://research.unsw.edu.au/projects/toniot-datasets

**Best For:**
- Network anomaly detection
- Training Isolation Forest with network features
- Training LSTM with network features
- IoT/ICS hybrid environments
- Multiple attack types

---

### CICIDS2017 (with ICS Extension)

**Source:** Canadian Institute for Cybersecurity

**Content:**
- Network traffic (PCAP)
- **80+ pre-extracted network features** ✅
- Multiple attack types
- Labeled data

**Network Features Available (Pre-extracted):**
- Flow duration
- Packet length (mean, std, min, max)
- Inter-arrival time (mean, std, min, max)
- Packets per second
- Bytes per second
- Protocol flags
- Active/idle time
- Subflow features

**Attacks Included:**
- Brute Force
- DoS/DDoS
- Port Scan
- Botnet
- Web attacks
- Infiltration

**Size:** ~8GB

**Access:** Free, direct download

**Link:** https://www.unb.ca/cic/datasets/ids-2017.html

**Best For:**
- Pre-extracted network features (saves preprocessing time)
- Network anomaly detection
- Fast prototyping
- Benchmark comparisons

---

## 3. Network Traffic Datasets (Protocol-Specific)

### 4SICS ICS Lab Dataset

**Source:** Swedish ICS Security Lab (Netresec)

**Content:**
- Real ICS network traffic captures
- Multiple protocols
- Normal operations and attacks

**Protocols:**
- Modbus
- Siemens S7
- DNP3
- Various ICS protocols

**Format:** PCAP files

**Access:** Free, direct download

**Link:** https://www.netresec.com/?page=PCAP4SICS

**Best For:**
- Protocol-specific analysis
- Network anomaly detection
- Multi-protocol support

---

### ICS-PCAP Dataset

**Source:** Community contributors (GitHub)

**Content:**
- Collection of ICS protocol PCAP files
- Various industrial scenarios
- Protocol examples and attacks

**Protocols:**
- Modbus
- DNP3
- S7
- OPC-UA
- BACnet

**Access:** Free, GitHub repository

**Link:** https://github.com/automayt/ICS-pcap

**Best For:**
- Protocol learning
- Parser development
- Quick testing

---

## 3. MITRE ATT&CK for ICS

### MITRE ATT&CK ICS Matrix

**Source:** MITRE Corporation

**Content:**
- 81 techniques across 12 tactics
- ICS-specific attack techniques
- Technique relationships
- Real-world attack examples

**Tactics:**
- Initial Access
- Execution
- Persistence
- Privilege Escalation
- Evasion
- Discovery
- Lateral Movement
- Collection
- Command and Control
- Inhibit Response Function
- Impair Process Control
- Impact

**Format:**
- JSON
- STIX 2.0
- CSV

**Access:** Free, public

**Web Interface:** https://attack.mitre.org/matrices/ics/

**Direct JSON Download:**
```
https://raw.githubusercontent.com/mitre-attack/attack-stix-data/master/ics-attack/ics-attack.json
```

**Best For:**
- GNN training (attack prediction)
- Technique mapping
- Attack sequence learning
- Neo4j graph population

---

## 4. Synthetic Data Generation

### Why Synthetic Data?

**Advantages:**
- Full control over attack timing
- Perfect for demo scenarios
- No licensing issues
- Privacy-preserving
- Customizable to your needs

**Disadvantages:**
- May not capture all real-world complexity
- Requires domain knowledge
- Need validation with real data

### Normal Operation Patterns

**Temperature Sensor:**
- Range: 280-320°C
- Pattern: Sine wave with Gaussian noise
- Frequency: 1 Hz sampling
- Noise: σ = 2°C

**Pressure Sensor:**
- Range: 100-150 psi
- Pattern: Steady state with small variations
- Frequency: 1 Hz sampling
- Noise: σ = 1 psi

**Flow Rate Sensor:**
- Range: 50-100 L/min
- Pattern: Correlated with valve position
- Frequency: 1 Hz sampling
- Noise: σ = 2 L/min

**Valve Position:**
- Range: 0-100%
- Pattern: Step changes (discrete states)
- Frequency: 1 Hz sampling
- States: 0%, 25%, 50%, 75%, 100%

### Attack Patterns

**1. Sudden Spike Attack (Isolation Forest Detection)**
- Temperature jumps from 300°C to 380°C instantly
- Duration: Single timestep
- Detection: < 5 seconds

**2. Gradual Drift Attack (LSTM Detection)**
- Temperature increases 2°C per second
- Duration: 60 seconds (300°C → 420°C)
- Detection: ~30 seconds

**3. Oscillation Attack**
- Rapid on/off cycling of pumps/valves
- Frequency: 0.5 Hz (2-second cycles)
- Duration: 5 minutes

**4. Setpoint Manipulation**
- Unauthorized changes to control setpoints
- Gradual or sudden changes
- May stay within normal ranges

**5. Sensor Spoofing**
- False sensor readings
- Can be constant or varying
- Designed to hide real process state

---

## 5. Recommended Dataset Strategy for MVP

### Phase 1: Synthetic Data (Week 1-2)

**Purpose:** Initial development and demo

**Tasks:**
- Generate realistic sensor data
- Create controlled attack scenarios
- Test detection algorithms
- Develop demo scenarios

**Advantages:**
- Immediate availability
- Perfect control
- No access restrictions

### Phase 2: Real Data Validation (Week 3-4)

**Purpose:** Validate with real ICS data

**Recommended Dataset:** Morris & Gao

**Tasks:**
- Download and preprocess data
- Train LSTM on real patterns
- Validate Isolation Forest
- Test on real Modbus traffic

**Why Morris & Gao:**
- Easy to access
- Modbus protocol (matches your design)
- Reasonable size (~2GB)
- Well-documented attacks

### Phase 3: MITRE Data Integration (Week 4)

**Purpose:** Attack prediction training

**Tasks:**
- Download MITRE ATT&CK ICS JSON
- Parse and load into Neo4j
- Extract technique sequences
- Train GNN on attack chains

**Attack Sequences Example:**
```
T0846 (Discovery) → T0800 (Lateral Movement) → T0843 (Program Download)
T0846 (Discovery) → T0858 (Change Operating Mode)
T0800 (Lateral Movement) → T0843 (Program Download) → T0836 (Modify Parameter)
```

### Phase 4: Advanced Datasets (Optional)

**If Time Permits:**
- SWaT for larger-scale training
- HAI for multi-protocol support
- WADI for complex scenarios

---

## 6. Data Preprocessing Requirements

### For LSTM Autoencoder Training

**Input Format:**
- Window size: 60 timesteps (60 seconds at 1 Hz)
- Features: 10 (temperature, pressure, flow, etc.)
- Shape: (batch_size, 60, 10)

**Preprocessing Steps:**
1. Remove missing values (interpolation)
2. Normalize features (min-max scaling to 0-1)
3. Create sliding windows
4. Train/validation/test split: 70/15/15

**Training Data:**
- Use only normal operation data
- No attack data in training
- Minimum: 24 hours of data
- Recommended: 7 days of data

### For Isolation Forest Training

**Input Format:**
- Single data points
- Features: 10 (same as LSTM)
- Shape: (n_samples, 10)

**Preprocessing Steps:**
1. Remove missing values
2. Normalize features (min-max scaling)
3. Fit on normal data only

**Training Data:**
- Normal operation only
- Minimum: 10,000 samples
- Contamination parameter: 0.01 (1%)

### For Physics Rules Engine

**Input Format:**
- Real-time sensor values
- No windowing required

**Rule Definition:**
- Based on domain knowledge
- Safety limits from equipment specs
- Rate-of-change limits
- Dependency rules

**Example Rules:**
```
Temperature: 250°C < T < 350°C
Pressure: 90 psi < P < 160 psi
Rate: dT/dt < 10°C/min
Dependency: IF valve_open THEN pump_running
```

### For GNN Attack Prediction

**Input Format:**
- Attack technique sequences
- Graph structure (techniques as nodes)
- Edge weights (transition probabilities)

**Preprocessing Steps:**
1. Parse MITRE ATT&CK JSON
2. Extract technique relationships
3. Create adjacency matrix
4. Generate training sequences

**Training Data:**
- Historical attack sequences
- MITRE documented attack chains
- Synthetic attack paths

---

## 7. Quick Start Data Sources

### Immediate Use (No Registration)

**1. MITRE ATT&CK ICS JSON**
- Direct download
- No registration required
- Updated regularly

**2. 4SICS PCAP Files**
- Direct download
- Multiple protocols
- Good for testing

**3. Synthetic Data Generation**
- Python script
- Full control
- Immediate availability

### Requires Registration (Free)

**1. SWaT Dataset**
- Email request to iTrust
- Data use agreement
- 1-2 week approval time

**2. WADI Dataset**
- Same process as SWaT
- Larger and more complex

**3. Morris & Gao Dataset**
- Simple web form
- Immediate download after submission

---

## 8. Data Storage and Management

### Storage Requirements

**Development:**
- Synthetic data: ~1 GB
- Morris & Gao: ~2 GB
- MITRE ATT&CK: ~50 MB
- **Total: ~3 GB**

**Production (Optional):**
- SWaT: ~40 GB
- WADI: ~50 GB
- HAI: ~15 GB
- **Total: ~105 GB**

### Data Organization

```
data/
├── synthetic/
│   ├── facility_a/
│   │   ├── normal_operation.csv
│   │   └── attack_scenarios.csv
│   ├── facility_b/
│   └── facility_c/
├── morris_gao/
│   ├── gas_pipeline/
│   └── water_storage/
├── mitre_attack/
│   ├── ics-attack.json
│   └── technique_sequences.csv
└── models/
    ├── lstm_weights/
    ├── isolation_forest/
    └── gnn_weights/
```

### Version Control

**Recommended:**
- Use DVC (Data Version Control) for large datasets
- Git LFS for smaller files
- Document data versions in README

---

## 9. Dataset Licensing and Usage

### Open Source / Public Domain
- MITRE ATT&CK: Public domain
- 4SICS: Free for research
- ICS-PCAP: Various licenses (check individual files)

### Academic / Research Use
- Morris & Gao: Free for research
- SWaT: Research use only, requires agreement
- WADI: Research use only, requires agreement
- HAI: MIT License (permissive)

### Commercial Use
- Check individual dataset licenses
- Some require permission for commercial use
- MITRE ATT&CK: Can be used commercially

---

## 10. Recommended Approach for Your Demo

### Best Strategy for November 30 Deadline

**Week 1-2: Synthetic Data**
- Generate 3 facilities worth of data
- Create 3 attack scenarios
- Perfect for controlled demo
- No dependencies on external data

**Week 3: Add Morris & Gao**
- Download and integrate
- Validate detection algorithms
- Show real-world applicability
- Backup if synthetic data issues

**Week 4: MITRE Integration**
- Download MITRE ATT&CK JSON
- Load into Neo4j
- Train GNN
- Essential for attack prediction feature

**Week 5-6: Polish**
- Fine-tune with real data
- Optimize performance
- Document data sources
- Prepare demo scenarios

### Minimum Viable Data

**For Demo to Work:**
1. ✅ Synthetic data (3 facilities)
2. ✅ MITRE ATT&CK (attack prediction)

**For Credibility:**
3. ✅ Morris & Gao (real-world validation)

**Nice to Have:**
4. ❌ SWaT/WADI (time-consuming to obtain)
5. ❌ HAI (additional protocols)

---

## 11. Data Generation Script (Recommended)

### Create Your Own Synthetic Data

**Advantages:**
- Immediate availability
- Perfect for demo
- Full control over scenarios
- No licensing issues

**What to Generate:**
- 7 days of normal operation per facility
- 10 attack scenarios per facility
- 1 Hz sampling rate
- 10 sensor features

**Estimated Time:**
- Script development: 4-8 hours
- Data generation: 1 hour
- Validation: 2 hours

---

## 12. Additional Resources

### Research Papers with Datasets

**"A Dataset to Support Research in the Design of Secure Water Treatment Systems"**
- SWaT dataset paper
- Describes attack scenarios
- Link: https://ieeexplore.ieee.org/document/7469060

**"HAI 1.0: HIL-based Augmented ICS Security Dataset"**
- HAI dataset paper
- Attack methodology
- Link: https://github.com/icsdataset/hai

### ICS Security Communities

**ICS-CERT**
- https://www.cisa.gov/ics
- Incident reports
- Attack patterns

**SANS ICS**
- https://www.sans.org/industrial-control-systems/
- Training materials
- Case studies

---

## Summary

### For Your MVP Demo (Priority Order)

1. **Synthetic Data** (Week 1-2)
   - Generate immediately
   - Full control
   - Perfect for demo

2. **MITRE ATT&CK** (Week 4)
   - Download JSON
   - Essential for GNN
   - Attack prediction feature

3. **Morris & Gao** (Week 3)
   - Easy to obtain
   - Real-world validation
   - Modbus protocol

4. **SWaT/WADI** (Optional)
   - If time permits
   - Larger scale
   - More credibility

### Total Data Needed

**Minimum (MVP):**
- Synthetic: ~1 GB
- MITRE: ~50 MB
- **Total: ~1 GB**

**Recommended:**
- Synthetic: ~1 GB
- Morris & Gao: ~2 GB
- MITRE: ~50 MB
- **Total: ~3 GB**

---

## 13. Network Feature Extraction Guide

### Extracting Network Features from PCAP Files

**Tools:**
- **Scapy** (Python) - Packet manipulation
- **PyShark** (Python) - Wireshark wrapper
- **tshark** (CLI) - Command-line Wireshark
- **CICFlowMeter** - Pre-built feature extractor

**Example: Extract 8 Network Features**

```python
import pyshark
import pandas as pd
from collections import defaultdict

def extract_network_features(pcap_file, window_seconds=1):
    """Extract 8 network features per second"""
    
    cap = pyshark.FileCapture(pcap_file)
    
    features = defaultdict(lambda: {
        'packets': 0,
        'bytes': 0,
        'src_ips': set(),
        'dst_ips': set(),
        'protocols': defaultdict(int),
        'packet_sizes': [],
        'inter_arrival_times': [],
        'failed_connections': 0
    })
    
    prev_time = None
    
    for packet in cap:
        timestamp = float(packet.sniff_timestamp)
        window = int(timestamp)  # 1-second windows
        
        # Packets per second
        features[window]['packets'] += 1
        
        # Bytes per second
        features[window]['bytes'] += int(packet.length)
        
        # Unique IPs
        if hasattr(packet, 'ip'):
            features[window]['src_ips'].add(packet.ip.src)
            features[window]['dst_ips'].add(packet.ip.dst)
        
        # Protocol distribution
        if hasattr(packet, 'highest_layer'):
            features[window]['protocols'][packet.highest_layer] += 1
        
        # Packet size
        features[window]['packet_sizes'].append(int(packet.length))
        
        # Inter-arrival time
        if prev_time:
            iat = (timestamp - prev_time) * 1000  # ms
            features[window]['inter_arrival_times'].append(iat)
        prev_time = timestamp
        
        # Failed connections (TCP RST, ICMP unreachable)
        if hasattr(packet, 'tcp') and packet.tcp.flags_reset == '1':
            features[window]['failed_connections'] += 1
    
    # Convert to DataFrame
    rows = []
    for window, data in features.items():
        rows.append({
            'timestamp': window,
            'packets_per_sec': data['packets'],
            'bytes_per_sec': data['bytes'],
            'unique_src_ips': len(data['src_ips']),
            'unique_dst_ips': len(data['dst_ips']),
            'protocol_dist': len(data['protocols']),  # or ratio
            'failed_connections': data['failed_connections'],
            'avg_packet_size': sum(data['packet_sizes']) / len(data['packet_sizes']) if data['packet_sizes'] else 0,
            'inter_arrival_variance': pd.Series(data['inter_arrival_times']).var() if data['inter_arrival_times'] else 0
        })
    
    return pd.DataFrame(rows)

# Usage
network_features = extract_network_features('hai_attack.pcap')
network_features.to_csv('network_features.csv', index=False)
```

### Recommended Dataset for Network Features

**Priority 1: HAI Dataset** ⭐
- Has both PCAP and sensor data
- 38 attacks
- Easy to access
- Well-documented

**Priority 2: TON_IoT**
- Comprehensive network features
- Multiple attack types
- Good for network-only training

**Priority 3: SWaT**
- Large-scale
- Network logs available
- Requires registration

---

**Document Version:** 1.1  
**Last Updated:** October 15, 2025  
**Status:** Ready for Use  
**Changes:** Added network-specific datasets (HAI, TON_IoT, CICIDS2017) with detailed network feature information and extraction guide
