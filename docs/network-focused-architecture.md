# Federated Network-Based ICS Threat Detection System

## Architecture & Design Document

**Project Focus:** Cyber Threat Detection for Industrial Control Systems  
**Data Source:** Network Traffic Analysis  
**Key Innovation:** Federated Learning for Collaborative Defense  
**Timeline:** October 18 - November 30, 2025 (6 weeks)

---

## Table of Contents
1. [Executive Summary](#executive-summary)
2. [System Architecture](#system-architecture)
3. [Network Data Collection](#network-data-collection)
4. [Detection Models](#detection-models)
5. [Correlation Engine](#correlation-engine)
6. [Attack Prediction](#attack-prediction)
7. [Federated Learning](#federated-learning)
8. [Technology Stack](#technology-stack)
9. [Attack Scenarios](#attack-scenarios)
10. [Implementation Plan](#implementation-plan)

---

## Executive Summary

### Project Overview

A federated learning-based network threat detection system for Industrial Control Systems (ICS) that enables multiple facilities to collaboratively defend against cyber attacks while preserving data privacy.

### Core Value Proposition

**"When one facility detects a new attack, all facilities learn to detect it - without sharing raw data"**

### Key Features

1. **Network-Based Detection** - Analyzes network traffic patterns to detect cyber threats
2. **Multi-Model Detection** - Uses 2 complementary AI models for comprehensive coverage
3. **Federated Learning** - Collaborative defense across facilities with privacy preservation
4. **Attack Prediction** - Predicts attacker's next move using Graph Neural Networks
5. **Real-Time Correlation** - Combines multiple detection signals for high-confidence alerts

### Why Network-Only Focus?

- ✅ **Clearer scope** - Cyber threat detection is well-defined
- ✅ **Simpler implementation** - 8 network features vs 18 total features
- ✅ **Better datasets** - Can use existing network security datasets
- ✅ **Faster development** - More time for polish and testing
- ✅ **Still shows innovation** - Federated learning is the unique value

---

## System Architecture

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Dashboard (React)                         │
│         Alerts | FL Status | Attack Graph | Metrics         │
└─────────────────────────────────────────────────────────────┘
                              ↕ WebSocket + REST
┌─────────────────────────────────────────────────────────────┐
│                    API Gateway (FastAPI)                     │
└─────────────────────────────────────────────────────────────┘
                              ↕
┌──────────────────┬──────────────────┬──────────────────────┐
│   FL Server      │  Correlation     │  Attack Prediction   │
│   (Flower)       │  Engine          │  (GNN + Neo4j)       │
└──────────────────┴──────────────────┴──────────────────────┘
         ↕                  ↕                    ↕
┌──────────────────┬──────────────────┬──────────────────────┐
│  Detection       │  Detection       │  Kafka Message Bus   │
│  Model 1:        │  Model 2:        │                      │
│  Isolation       │  LSTM            │                      │
│  Forest          │  Autoencoder     │                      │
└──────────────────┴──────────────────┴──────────────────────┘
         ↕                  ↕                    ↕
┌──────────────────┬──────────────────┬──────────────────────┐
│  FL Client A     │  FL Client B     │  FL Client C         │
│  (Facility A)    │  (Facility B)    │  (Facility C)        │
└──────────────────┴──────────────────┴──────────────────────┘
         ↕                  ↕                    ↕
┌──────────────────┬──────────────────┬──────────────────────┐
│   PostgreSQL     │  Network Data    │  Network Data        │
│   (Alerts)       │  Collector       │  Simulator           │
└──────────────────┴──────────────────┴──────────────────────┘
```

### Component Breakdown

#### 1. **Network Data Layer**
- Network traffic capture/simulation
- Feature extraction (8 network metrics)
- Real-time streaming to detection models

#### 2. **Detection Layer**
- Isolation Forest (fast anomaly detection)
- LSTM Autoencoder (behavioral analysis)
- Independent operation, publish to Kafka

#### 3. **Correlation Layer**
- Receives alerts from both models
- Groups by asset and time window
- Calculates confidence and severity
- Publishes correlated alerts

#### 4. **Prediction Layer**
- GNN analyzes detected attacks
- Queries MITRE ATT&CK graph
- Predicts next attack techniques
- Provides actionable recommendations

#### 5. **Federated Learning Layer**
- FL Server coordinates training rounds
- FL Clients train on local data
- Differential privacy applied
- Model aggregation and distribution

#### 6. **Presentation Layer**
- Real-time dashboard
- Alert management
- FL status monitoring
- Attack graph visualization

---

## Network Data Collection

### The 8 Network Features

We extract 8 key metrics from network traffic every second:

```python
network_features = {
    # Traffic Volume Metrics
    "packets_per_sec": int,        # Number of packets in last second
    "bytes_per_sec": int,          # Total bytes transferred
    
    # Connection Pattern Metrics
    "unique_src_ips": int,         # Number of unique source IPs
    "unique_dest_ips": int,        # Number of unique destination IPs
    
    # Protocol Metrics
    "protocol_distribution": float, # Ratio of industrial protocols (Modbus, DNP3, OPC-UA)
    
    # Connection Quality Metrics
    "failed_connections": int,      # Number of failed connection attempts
    
    # Packet Characteristic Metrics
    "avg_packet_size": float,      # Average packet size in bytes
    "inter_arrival_time": float    # Average time between packets (ms)
}
```

### Feature Extraction Process

```
Raw Network Packets
       ↓
┌──────────────────────────────────┐
│   Packet Capture                 │
│   - Source IP, Dest IP           │
│   - Protocol, Port               │
│   - Packet size, Timestamp       │
│   - Connection status            │
└──────────────────────────────────┘
       ↓
┌──────────────────────────────────┐
│   Aggregation (1-second window)  │
│   - Count packets                │
│   - Sum bytes                    │
│   - Count unique IPs             │
│   - Calculate averages           │
└──────────────────────────────────┘
       ↓
┌──────────────────────────────────┐
│   Feature Vector (8 values)      │
│   [45, 23000, 5, 5, 0.8,        │
│    2, 512, 20]                   │
└──────────────────────────────────┘
       ↓
   Detection Models
```

### Normal vs Attack Traffic

**Normal Operation:**
```python
{
    "packets_per_sec": 45,
    "bytes_per_sec": 23000,
    "unique_src_ips": 5,
    "unique_dest_ips": 5,
    "protocol_distribution": 0.80,  # 80% Modbus
    "failed_connections": 2,
    "avg_packet_size": 512,
    "inter_arrival_time": 20
}
```

**Port Scan Attack:**
```python
{
    "packets_per_sec": 3000,        # ← Spike!
    "bytes_per_sec": 150000,        # ← Spike!
    "unique_src_ips": 1,            # ← Single attacker
    "unique_dest_ips": 250,         # ← Scanning many targets!
    "protocol_distribution": 0.0,   # ← Raw TCP, not Modbus
    "failed_connections": 2500,     # ← Most connections fail!
    "avg_packet_size": 64,          # ← Small packets
    "inter_arrival_time": 1         # ← Very fast!
}
```

---

## Detection Models

### Model 1: Isolation Forest

**Purpose:** Fast detection of sudden network anomalies

**Type:** Machine Learning - Ensemble Method (Unsupervised)

**How It Works:**
- Builds 100 random decision trees
- Anomalies are "easy to isolate" (short path in trees)
- Normal points are "hard to isolate" (long path in trees)
- Calculates anomaly score based on average path length

**Input:**
- Single data point (current second)
- 8 network features

**Output:**
- Anomaly score (0-1)
- Alert if score > 0.6

**Detection Speed:** < 5 seconds

**Training:**
- Train on 7 days of normal network traffic
- No labeled data needed (unsupervised)
- Retrains weekly or when network patterns change

**What It Detects:**
- ✅ Port scans (sudden spike in dest IPs)
- ✅ DDoS attacks (massive packet flood)
- ✅ Traffic spikes (unusual volume)
- ✅ Network reconnaissance (scanning patterns)

**What It Misses:**
- ❌ Slow scans (gradual changes)
- ❌ Data exfiltration (subtle patterns)
- ❌ Stealthy attacks (below threshold)

**Federated:** ❌ No (local per facility)

---

### Model 2: LSTM Autoencoder

**Purpose:** Behavioral analysis of network patterns over time

**Type:** Deep Learning - Recurrent Neural Network (Unsupervised)

**How It Works:**
- Learns normal network behavior patterns
- Encoder compresses 60-second window to latent representation
- Decoder tries to reconstruct original input
- High reconstruction error = abnormal behavior

**Architecture:**
```
Input: (60 timesteps, 8 features)
       ↓
Encoder: LSTM(128) → LSTM(64)
       ↓
Latent: 64-dimensional representation
       ↓
Decoder: LSTM(64) → LSTM(128) → Dense(8)
       ↓
Output: (60 timesteps, 8 features)
       ↓
Reconstruction Error (MSE)
```

**Input:**
- 60-second sliding window
- 8 network features per timestep
- Shape: (60, 8)

**Output:**
- Reconstruction error (MSE)
- Alert if error > threshold (95th percentile)

**Detection Speed:** < 30 seconds

**Training:**
- Train on 7 days of normal network traffic
- 60-second windows
- 5 epochs per FL round
- Differential privacy applied

**What It Detects:**
- ✅ Slow network scans (gradual increase)
- ✅ Data exfiltration (unusual outbound patterns)
- ✅ Malware spreading (infection patterns)
- ✅ Behavioral anomalies (pattern changes)

**What It Misses:**
- ❌ Instant attacks (needs 30+ seconds of data)
- ❌ Very subtle changes (below threshold)

**Federated:** ✅ Yes (learns collaboratively)

---

### Why These 2 Models?

**Complementary Strengths:**

| Aspect | Isolation Forest | LSTM Autoencoder |
|--------|-----------------|------------------|
| **Speed** | < 5 seconds | < 30 seconds |
| **Detection Type** | Sudden anomalies | Behavioral patterns |
| **Time Window** | Single point | 60 seconds |
| **Learning** | Unsupervised | Unsupervised |
| **Federated** | No | Yes |
| **Best For** | Port scans, DDoS | Slow scans, exfiltration |

**Together:** 85-90% attack coverage

---

## Correlation Engine

### Purpose

Combines alerts from both detection models to create high-confidence alerts.

### How It Works

```
Step 1: Receive alerts from Kafka
       ↓
Step 2: Group by facility + asset + time window (60 seconds)
       ↓
Step 3: Calculate confidence based on number of sources
       ↓
Step 4: Escalate severity if multiple sources agree
       ↓
Step 5: Publish correlated alert
```

### Correlation Logic

```python
def correlate_alerts(alerts):
    # Group alerts by asset and time
    grouped = group_by_asset_and_time(alerts, window=60)
    
    for asset, alert_group in grouped:
        sources = set([a['source'] for a in alert_group])
        num_sources = len(sources)
        
        # Calculate confidence
        base_confidence = max([a['confidence'] for a in alert_group])
        
        if num_sources >= 2:
            # Both models agree!
            confidence = min(base_confidence * 1.4, 1.0)
            severity = "CRITICAL"
        else:
            # Only one model
            confidence = base_confidence
            severity = alert_group[0]['severity']
        
        return CorrelatedAlert(
            sources=sources,
            confidence=confidence,
            severity=severity,
            num_sources=num_sources
        )
```

### Progressive Correlation

**Don't wait - alert immediately, then update!**

```
10:02:01 - Isolation Forest detects
         → IMMEDIATE ALERT (1 source, WARNING)
         
10:02:30 - LSTM detects
         → UPDATE ALERT (2 sources, CRITICAL)
         → Both models agree!
```

### Benefits

- ✅ **Fast initial response** (< 5 seconds from Isolation Forest)
- ✅ **Progressive confidence** (gets stronger as more models detect)
- ✅ **Reduced false positives** (both models agreeing = high confidence)
- ✅ **Real-time updates** (dashboard shows live correlation)

---

## Attack Prediction

### Purpose

Predict attacker's next move after detecting an attack.

### Technology: Graph Neural Network (GAT)

**Type:** Deep Learning - Graph Attention Network (Supervised)

**How It Works:**
- Uses MITRE ATT&CK for ICS as knowledge base
- Techniques are nodes, relationships are edges
- GNN learns which techniques typically follow others
- Predicts top-3 most likely next steps

**Architecture:**
```
Input: Current technique (e.g., T0846 - Port Scan)
       ↓
Query: Neo4j for MITRE ATT&CK graph
       ↓
GNN: GATConv(64, heads=4) → Dropout(0.6) → GATConv(32, heads=4)
       ↓
Output: Probability distribution over next techniques
       ↓
Top-3 Predictions with probabilities
```

**Example:**

```
Current Attack: T0846 (Remote System Discovery - Port Scan)
       ↓
GNN Predictions:
1. T0800 (Lateral Movement) - 72% probability
2. T0843 (Program Download) - 67% probability
3. T0858 (Change Operating Mode) - 10% probability
       ↓
Recommended Actions:
- Block lateral movement attempts
- Monitor for program downloads to PLCs
- Increase logging on critical assets
```

**Federated:** ✅ Yes (learns attack patterns collaboratively)

---

## Federated Learning

### Purpose

Enable facilities to learn from each other's attack experiences without sharing raw data.

### How It Works

```
┌─────────────────────────────────────────────────────────┐
│                    FL Server                            │
│  - Coordinates training rounds                          │
│  - Aggregates model updates                             │
│  - Distributes global model                             │
└─────────────────────────────────────────────────────────┘
         ↕                ↕                ↕
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│ FL Client A  │  │ FL Client B  │  │ FL Client C  │
│ Facility A   │  │ Facility B   │  │ Facility C   │
│              │  │              │  │              │
│ Train on     │  │ Train on     │  │ Train on     │
│ local data   │  │ local data   │  │ local data   │
│              │  │              │  │              │
│ Send weights │  │ Send weights │  │ Send weights │
└──────────────┘  └──────────────┘  └──────────────┘
```

### FL Round Process

**Step 1: Local Training (Each Facility)**
```python
# Each facility trains on its own network data
for epoch in range(5):
    for batch in local_network_data:
        loss = train_step(model, batch)
        loss.backward()
        optimizer.step()

# Apply differential privacy
private_weights = add_differential_privacy(model.weights)
```

**Step 2: Send to FL Server**
```python
# Send only model weights (not raw data!)
fl_client.send_weights(private_weights)
```

**Step 3: Aggregation (FL Server)**
```python
# Collect weights from all clients
weights_list = [client_a_weights, client_b_weights, client_c_weights]

# Aggregate using FedMedian (Byzantine-robust)
global_weights = fedmedian_aggregate(weights_list)

# Update global model
global_model.set_weights(global_weights)
```

**Step 4: Distribution**
```python
# Send updated global model back to all clients
for client in fl_clients:
    client.update_model(global_weights)
```

### Privacy Preservation

**Differential Privacy (Opacus):**
- Adds calibrated noise to gradients
- Privacy budget: ε=2.0, δ=10⁻⁵
- Prevents reverse-engineering of training data

**What's Shared:**
- ✅ Model weights (~10 MB)
- ✅ Aggregated gradients

**What's NOT Shared:**
- ❌ Raw network traffic data
- ❌ Facility-specific information
- ❌ Attack details
- ❌ Network configurations

### Benefits

**Before FL:**
```
Facility A: Detects new attack type
Facility B: Vulnerable (never seen this attack)
Facility C: Vulnerable (never seen this attack)
```

**After FL Round (6 hours later):**
```
Facility A: Still detects (original knowledge)
Facility B: Now detects! (learned from A)
Facility C: Now detects! (learned from A)
```

**Result:** Collaborative defense without data sharing

---

## Technology Stack

### Backend

| Component | Technology | Version | Purpose |
|-----------|-----------|---------|---------|
| **API Framework** | FastAPI | 0.104+ | REST API + WebSocket |
| **ML Framework** | PyTorch | 2.1+ | LSTM, GNN models |
| **ML Library** | scikit-learn | 1.3+ | Isolation Forest |
| **FL Framework** | Flower | 1.6+ | Federated learning |
| **Privacy** | Opacus | 1.4+ | Differential privacy |
| **Message Bus** | Kafka | 3.6+ | Event streaming |
| **Database** | PostgreSQL | 15+ | Alerts, FL rounds |
| **Graph DB** | Neo4j | 5.x | MITRE ATT&CK |
| **Language** | Python | 3.11+ | All backend code |

### Frontend

| Component | Technology | Version | Purpose |
|-----------|-----------|---------|---------|
| **Framework** | React | 18+ | UI framework |
| **Language** | TypeScript | 5+ | Type safety |
| **UI Library** | Material-UI | 5+ | Components |
| **Charts** | Recharts | 2+ | Time-series charts |
| **Graphs** | D3.js | 7+ | Attack graph viz |
| **State** | React Query | 4+ | Data fetching |
| **WebSocket** | Socket.io | 4+ | Real-time updates |

### Infrastructure

| Component | Technology | Purpose |
|-----------|-----------|---------|
| **Containerization** | Docker | Service isolation |
| **Orchestration** | Docker Compose | Multi-container apps |
| **CI/CD** | GitHub Actions | Automated testing |

---

## Attack Scenarios

### Scenario 1: Port Scan Attack

**Attack Type:** Network Reconnaissance (MITRE ATT&CK: T0846)

**Timeline:**
```
10:02:00 - Normal operation (45 packets/sec)
10:02:15 - Attacker starts port scan (3000 packets/sec, 250 dest IPs)
10:02:16 - Isolation Forest detects (< 1 second)
10:02:45 - LSTM detects (30 seconds)
10:02:45 - Correlation: CRITICAL (both models agree)
10:02:46 - GNN predicts: T0800 (Lateral Movement) next
```

**Detection:**
- Isolation Forest: ✅ Immediate (extreme outlier)
- LSTM: ✅ Confirms (abnormal pattern)
- Correlation: ✅ CRITICAL (100% confidence)
- GNN: ✅ Predicts next step

---

### Scenario 2: Slow Network Scan (Stealthy)

**Attack Type:** Stealthy Reconnaissance

**Timeline:**
```
10:05:00 - Normal operation (45 packets/sec, 5 dest IPs)
10:05:00 - Attacker starts slow scan (60 packets/sec, 2 ports/sec)
10:05:05 - Isolation Forest: No alert (not extreme enough)
10:05:30 - LSTM detects (gradual pattern change)
10:05:30 - Correlation: WARNING (1 source)
```

**Detection:**
- Isolation Forest: ❌ Misses (too subtle)
- LSTM: ✅ Detects (abnormal pattern over time)
- Correlation: ⚠️ WARNING (lower confidence)

**This shows why you need both models!**

---

### Scenario 3: DDoS Attack

**Attack Type:** Denial of Service

**Timeline:**
```
10:10:00 - Normal operation (45 packets/sec)
10:10:15 - DDoS begins (500,000 packets/sec)
10:10:16 - Isolation Forest detects (< 1 second)
10:10:45 - LSTM detects (30 seconds)
10:10:45 - Correlation: CRITICAL (both models agree)
```

**Detection:**
- Isolation Forest: ✅ Immediate
- LSTM: ✅ Confirms
- Correlation: ✅ CRITICAL

---

### Scenario 4: Data Exfiltration

**Attack Type:** Data Theft

**Timeline:**
```
10:15:00 - Normal operation
10:15:00 - Attacker exfiltrates data (consistent 500-byte packets to external IP)
10:15:05 - Isolation Forest: No alert (volume not extreme)
10:15:30 - LSTM detects (unusual outbound pattern)
10:15:30 - Correlation: WARNING (1 source)
10:15:31 - GNN identifies as data theft
```

**Detection:**
- Isolation Forest: ❌ Misses (subtle)
- LSTM: ✅ Detects (abnormal pattern)
- GNN: ✅ Classifies attack type

---

## Implementation Plan

### Week 1-2: Foundation (Oct 18 - Nov 1)

**Infrastructure:**
- ✅ Docker Compose setup
- ✅ Kafka + PostgreSQL + Neo4j
- ✅ Network data simulator
- ✅ Basic data pipeline

**Deliverables:**
- All services running
- Simulator generates realistic network traffic
- Data flows to Kafka

---

### Week 3: Detection Models (Nov 2 - Nov 8)

**Isolation Forest:**
- ✅ Train on normal network data
- ✅ Real-time detection service
- ✅ Alert generation

**LSTM Autoencoder:**
- ✅ Model architecture
- ✅ Training pipeline
- ✅ Real-time detection service

**Deliverables:**
- Both models detecting attacks
- Alerts published to Kafka

---

### Week 4: Federated Learning (Nov 9 - Nov 15)

**FL Server:**
- ✅ Flower server setup
- ✅ Model aggregation (FedMedian)
- ✅ Round management

**FL Clients:**
- ✅ 3 clients (one per facility)
- ✅ Local training
- ✅ Differential privacy (Opacus)

**Deliverables:**
- FL round completes successfully
- Model improves after FL

---

### Week 5: Correlation + Prediction (Nov 16 - Nov 22)

**Correlation Engine:**
- ✅ Alert grouping logic
- ✅ Confidence calculation
- ✅ Progressive correlation

**GNN:**
- ✅ MITRE ATT&CK graph in Neo4j
- ✅ GNN model training
- ✅ Prediction service

**Deliverables:**
- Correlated alerts working
- Attack prediction working

---

### Week 6: Dashboard + Demo (Nov 23 - Nov 30)

**Dashboard:**
- ✅ Alerts page
- ✅ FL status page
- ✅ Attack graph visualization
- ✅ Real-time WebSocket updates

**Demo:**
- ✅ Demo scenarios implemented
- ✅ Testing and polish
- ✅ Documentation
- ✅ Video recording

**Deliverables:**
- Working demo
- Documentation complete
- Video walkthrough

---

## Success Metrics

### Technical Metrics

- **Detection Latency:** < 30 seconds (from attack to alert)
- **FL Round Duration:** < 6 minutes
- **Detection Accuracy:** > 85% on test scenarios
- **False Positive Rate:** < 10%
- **System Throughput:** 1000 messages/second

### Demo Metrics

- **Scenario Success Rate:** > 95% (10 runs)
- **Dashboard Responsiveness:** < 1 second update latency
- **FL Demonstration:** Shows collaborative learning clearly
- **Attack Prediction:** Accurate predictions for demo scenarios

---

## Conclusion

This network-focused architecture provides:

1. ✅ **Clear scope** - Cyber threat detection for ICS networks
2. ✅ **Innovative approach** - Federated learning for collaborative defense
3. ✅ **Comprehensive detection** - 85-90% attack coverage with 2 models
4. ✅ **Proactive defense** - Attack prediction with GNN
5. ✅ **Privacy preservation** - Differential privacy guarantees
6. ✅ **Achievable timeline** - 6 weeks with time for polish

**Key Differentiator:** Federated learning enables facilities to learn from each other's attack experiences without sharing sensitive data - faster than traditional threat intelligence sharing.

---

**Document Version:** 1.0  
**Last Updated:** October 18, 2025  
**Status:** Ready for Implementation  
**Author:** Project Architecture Team
