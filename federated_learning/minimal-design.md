# MVP Design Document - November 30 Demo

## Overview

This design document focuses exclusively on the MUST HAVE features identified in demo-priorities.md. It provides a streamlined architecture and implementation plan for delivering a working demonstration by November 30, 2025.

**Design Philosophy:**
- Demo-first: Every component must contribute to the 3-minute demo
- Minimal viable: No features beyond the 5 core requirements
- Proven and simple: Use the simplest technology that works
- Integration-focused: Components must work together seamlessly

**Core Value Proposition:**
Demonstrate that federated learning enables collaborative ICS threat detection while preserving data privacy - faster than traditional threat intelligence sharing.

---

## The 5 MUST HAVE Features

### 1. Federated Learning (THE KILLER FEATURE) 🏆
**Why:** Only unique differentiator vs competitors
**Demo Impact:** Shows collaborative defense in action

### 2. Real-Time Detection (PROOF IT WORKS) ✅
**Why:** Proves the system actually detects threats
**Demo Impact:** Shows robust multi-layered detection

### 3. Dashboard Visualization (SHOW, DON'T TELL) 📊
**Why:** Visual proof is more convincing than logs
**Demo Impact:** Makes abstract concepts concrete

### 4. Privacy Preservation (TRUST FACTOR) 🔒
**Why:** Addresses "why not just share data?" question
**Demo Impact:** Shows mathematical privacy guarantees

### 5. Attack Prediction (WOW FACTOR) 🔮
**Why:** Proactive defense - predicts attacker's next move
**Demo Impact:** Shifts from reactive to proactive

---

## Architecture

### Simplified System Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Dashboard (React)                         │
│              Alerts | FL Status | Attack Graph               │
└─────────────────────────────────────────────────────────────┘
                              ↕ WebSocket + REST
┌─────────────────────────────────────────────────────────────┐
│                    API Gateway (FastAPI)                     │
└─────────────────────────────────────────────────────────────┘
                              ↕
┌──────────────────┬──────────────────┬──────────────────────┐
│   FL Server      │  Detection       │  Attack Prediction   │
│   (Flower)       │  Services        │  (GNN + Neo4j)       │
│                  │  LSTM + IF +     │                      │
│                  │  Physics Rules   │                      │
└──────────────────┴──────────────────┴──────────────────────┘
         ↕                  ↕                    ↕
┌──────────────────┬──────────────────┬──────────────────────┐
│  FL Client A     │  FL Client B     │  FL Client C         │
│  (Facility A)    │  (Facility B)    │  (Facility C)        │
└──────────────────┴──────────────────┴──────────────────────┘
         ↕                  ↕                    ↕
┌──────────────────┬──────────────────┬──────────────────────┐
│   Kafka          │  IoTDB           │  PostgreSQL          │
│   (Messages)     │  (Sensor Data)   │  (Alerts)            │
└──────────────────┴──────────────────┴──────────────────────┘
         ↕                                       ↕
┌─────────────────────────────────────────────────────────────┐
│              Modbus Simulator + Sensor Data                  │
└─────────────────────────────────────────────────────────────┘
```


### Data Flow for Demo Scenarios

**Scenario 1: Detection + FL**
```
1. Simulator → Kafka + IoTDB (dual write)
2. Detection Services query IoTDB for sensor history
3. LSTM + IF + Physics → Correlation → Alert
4. Alert → PostgreSQL → Dashboard (WebSocket)
5. Alert triggers FL round
6. FL Server → 3 FL Clients → Train on IoTDB data → Upload weights
7. FL Server → Aggregate → Distribute updated model
8. Dashboard shows FL progress
9. Replay attack → All facilities detect
```

**Why IoTDB:**
- Optimized for time-series queries (LSTM needs 60-second windows)
- 10-100x compression vs PostgreSQL
- Fast range queries for historical data
- Handles high-frequency data (1Hz sensor readings)

**Scenario 2: Attack Prediction**
```
1. Alert detected → Extract MITRE technique
2. Query Neo4j for technique relationships
3. GNN predicts next techniques with probabilities
4. Store prediction → PostgreSQL
5. Dashboard shows attack graph + predictions
6. Attacker proceeds → Prediction validated
```

---

## Technology Stack (Minimal)

### Core Technologies

| Component | Technology | Justification |
|-----------|-----------|---------------|
| Backend | Python 3.11 + FastAPI | ML ecosystem, async support |
| Message Bus | Kafka 3.6 | Reliable messaging, proven |
| Time-Series DB | Apache IoTDB 1.3.0 | Optimized for sensor data |
| Database | PostgreSQL 15 | Alerts, FL rounds, predictions |
| Graph DB | Neo4j 5.x | MITRE ATT&CK relationships |
| ML Framework | PyTorch 2.1 | LSTM + GNN support |
| FL Framework | Flower 1.6 | Simple, framework-agnostic |
| Privacy | Opacus 1.4 | PyTorch DP integration |
| Frontend | React 18 + TypeScript | Fast development, real-time |
| Visualization | Recharts + D3.js | Charts + custom graphs |
| Container | Docker Compose | Simple orchestration |

### What We're NOT Using (Simplified)
- ❌ MongoDB - Use PostgreSQL JSONB (one less DB)
- ❌ Multiple protocols - Modbus only
- ❌ TensorFlow - PyTorch only (consistency)

---

## Component Details

### 1. Federated Learning System

#### FL Server
**Technology:** Flower (flwr) + FastAPI
**Port:** 8080

**Responsibilities:**
- Schedule FL rounds (manual trigger for demo)
- Distribute global model to 3 clients
- Collect weight updates
- Aggregate using FedMedian (Byzantine-robust)
- Track round metrics

**Key Features:**
- FedMedian aggregation strategy
- Minimum 3 clients required
- Model versioning and tracking
- Round history and metrics

#### FL Client
**Technology:** Flower (flwr) + Opacus
**Instances:** 3 (Facility A, B, C)

**Responsibilities:**
- Connect to FL server
- Receive global model
- Train on local data (5 epochs)
- Apply differential privacy
- Send weight updates

**Privacy Configuration:**
- Noise multiplier: 1.1 (calibrated for ε=2.0)
- Gradient clipping: L2 norm, max=1.0
- Privacy accounting: ε=2.0, δ=10⁻⁵

#### Privacy Metrics
**Display on Dashboard:**
- Epsilon (ε): 2.0
- Delta (δ): 10⁻⁵
- Data transmitted: ~10 MB (weights only)
- Raw data: Stays local

---

### 2. Detection Services

#### LSTM Autoencoder
**Purpose:** Detect behavioral anomalies (temporal patterns)

**Architecture:**
- Encoder: Input(60, 10) → LSTM(128) → LSTM(64)
- Decoder: LSTM(64) → LSTM(128) → Output(60, 10)

**Specifications:**
- Input: 60 timesteps × 10 features (temperature, pressure, flow, etc.)
- Output: Reconstruction error (MSE)
- Threshold: 95th percentile of training errors
- Latency: < 30 seconds
- Data Source: IoTDB (60-second time windows)

#### Isolation Forest
**Purpose:** Detect point anomalies (sudden outliers)

**Specifications:**
- Algorithm: sklearn IsolationForest
- Estimators: 100 trees
- Contamination: 0.01 (1% expected anomalies)
- Input: Single data point (10 features)
- Output: Anomaly score (-1 to 1)
- Threshold: Score > 0.6
- Latency: < 5 seconds
- Data Source: IoTDB (latest values)

#### Physics Rules Engine
**Purpose:** Validate against safety constraints

**Rule Types:**
- Range checks (temperature, pressure limits)
- Rate-of-change limits (sudden spikes)
- Dependency rules (valve/pump coordination)

**Specifications:**
- Latency: < 1 second (rule evaluation)
- Severity levels: CRITICAL, WARNING, INFO
- Configurable per facility

#### Correlation Engine
**Purpose:** Combine signals from all 3 detection methods

**Algorithm:**
- Group detections by asset and time (5-minute window)
- Calculate confidence based on source agreement
- Determine severity based on number of sources
- 3 sources = CRITICAL, 2 sources = HIGH, 1 source = original severity

**Key Insight:** All 3 methods agreeing = high confidence, low false positive

---

### 3. Attack Prediction System

#### Neo4j MITRE ATT&CK Graph
**Purpose:** Store technique relationships

**Graph Structure:**
- Nodes: MITRE ATT&CK techniques, assets
- Edges: LEADS_TO relationships with probabilities
- Properties: Technique metadata, timing information

**Key Techniques for Demo:**
- T0846: Remote System Discovery
- T0800: Lateral Movement
- T0843: Program Download
- T0858: Change Operating Mode

#### Graph Neural Network (GAT)
**Purpose:** Predict next attack techniques

**Architecture:**
- Layer 1: GATConv(64, heads=4)
- Dropout: 0.6
- Layer 2: GATConv(32, heads=4)
- Classifier: Linear layer → Softmax

**Specifications:**
- Input: Current technique + graph structure
- Output: Probability distribution over next techniques
- Training: Supervised learning on attack sequences
- Top-K predictions: 3 most likely next steps

**Federated Learning Integration:**
- GNN model is federated
- Learns attack patterns from all facilities
- Improves prediction accuracy collaboratively

---

### 4. Dashboard

#### Technology Stack
- React 18 + TypeScript
- Material-UI (MUI) for components
- Recharts for time-series charts
- D3.js for attack graph visualization
- WebSocket for real-time updates

#### Pages (3 Essential)

**1. Alerts Dashboard**
**Components:**
- Real-time alert feed (auto-scroll)
- Facility status cards (3 cards, color-coded: green/yellow/red)
- Detection method indicators (LSTM, IF, Physics)
- Alert details modal with full context

**Key Features:**
- Real-time updates via WebSocket
- Filter by severity, facility, status
- Acknowledge and resolve actions
- Alert correlation visualization

**2. FL Status Page**
**Components:**
- Round progress bar with percentage
- Client status indicators (3 cards showing training progress)
- Privacy metrics display (ε, δ, data transmitted)
- Round history table
- "Trigger FL Round" button

**Key Features:**
- Live FL round progress
- Client connection status
- Model version tracking
- Performance metrics per round

**3. Attack Graph Page**
**Components:**
- D3.js force-directed graph visualization
- Current technique highlighted (red node)
- Predicted techniques highlighted (orange nodes)
- Probability labels on edges
- Technique details panel

**Key Features:**
- Interactive graph navigation
- Zoom and pan controls
- Technique information on hover
- Attack path history

#### WebSocket Integration
**Channels:**
- alerts: Real-time alert notifications
- fl_status: FL round progress updates
- predictions: New attack predictions
- system: System status changes

---

### 5. Data Simulator

#### Modbus Simulator
**Purpose:** Generate realistic ICS traffic for demo

**Capabilities:**
- Simulate 3 facilities with PLCs
- Generate normal operation patterns (sine waves, steady states)
- Inject attack scenarios on demand
- Dual-write to Kafka and IoTDB

**Sensor Types:**
- Temperature (280-320°C normal range)
- Pressure (100-150 psi normal range)
- Flow rate (variable)
- Valve positions (binary/percentage)

**Attack Scenarios:**
1. **Sudden Spike Attack** - Isolation Forest detection
   - Immediate value jump (e.g., 300°C → 380°C)
   - Triggers within 5 seconds

2. **Gradual Drift Attack** - LSTM detection
   - Slow increase over 60 seconds
   - Behavioral pattern change
   - Triggers within 30 seconds

3. **Multi-Stage Attack** - GNN prediction
   - Discovery → Lateral Movement → Program Download
   - Demonstrates attack prediction capability

---

## Database Schema

### Apache IoTDB Schema

**Purpose:** High-performance time-series storage for sensor data

**Structure:**
- 3 databases: root.facility_a, root.facility_b, root.facility_c
- Timeseries per sensor: temperature, pressure, flow_rate, valve_position
- Encoding: GORILLA (float), RLE (int)
- Compression: SNAPPY

**Benefits:**
- Optimized compression (10-100x vs PostgreSQL)
- Fast time-range queries
- Built-in downsampling and aggregation
- Handles high-frequency sensor data (1Hz+)

**Query Patterns:**
- Recent data for LSTM: Last 60 seconds
- Latest values for Isolation Forest: Last reading
- Historical analysis: Time range queries

### PostgreSQL Tables

**Alerts Table:**
- alert_id, timestamp, facility_id
- severity, confidence, title, description
- affected_assets, techniques (MITRE IDs)
- sources (LSTM, IsolationForest, Physics)
- status (open, acknowledged, resolved)

**FL Rounds Table:**
- round_id, start_time, end_time
- num_clients, model_version
- metrics (accuracy, loss, privacy_epsilon)
- status (in_progress, completed, failed)

**Predictions Table:**
- prediction_id, timestamp
- current_technique, predicted_techniques (JSONB)
- confidence, target_assets
- timeframe, validated (boolean)

**Data Separation:**
- **IoTDB:** High-frequency sensor data (temperature, pressure, flow)
- **PostgreSQL:** Alerts, FL rounds, predictions, system metadata
- **Neo4j:** MITRE ATT&CK graph relationships

---

## API Specification

### REST Endpoints

**Alerts:**
- GET /api/alerts - List alerts with filtering
- GET /api/alerts/{id} - Get alert details
- PUT /api/alerts/{id}/status - Update status

**Federated Learning:**
- POST /api/fl/rounds/trigger - Start FL round
- GET /api/fl/rounds - List rounds
- GET /api/fl/rounds/{id} - Round details
- GET /api/fl/clients - Connected clients

**Predictions:**
- GET /api/predictions - List predictions
- GET /api/predictions/{id} - Prediction details
- POST /api/predictions/predict - Trigger prediction

**Demo:**
- POST /api/demo/scenarios/attack - Trigger attack scenario
- POST /api/demo/scenarios/fl - Trigger FL demo
- GET /api/demo/status - Demo status

**System:**
- GET /api/system/health - Health check
- GET /api/system/metrics - System metrics

### WebSocket Channels
- ws://localhost:8000/ws
- Channels: alerts, fl_status, predictions, system

---

## Demo Scenario Implementations

### Scenario 1: Multi-Layered Detection + FL

**Steps:**
1. Show normal operation (30 seconds)
2. Inject attack on Facility A (sudden spike to 380°C)
3. Wait for detections (30 seconds)
   - Isolation Forest detects (5 sec)
   - LSTM detects (30 sec)
   - Physics rules violated
4. Correlation creates high-confidence alert
5. Trigger FL round
6. Wait for FL completion (5 minutes)
7. Replay attack on Facility B
8. Verify immediate detection by all methods

**Expected Timeline:**
- 0:00 - Normal operation
- 0:30 - Attack on Facility A
- 0:35 - Isolation Forest alert
- 1:00 - LSTM + Physics alerts
- 1:05 - Correlated alert (3/3 sources)
- 1:10 - FL round triggered
- 6:10 - FL round complete
- 6:15 - Attack on Facility B
- 6:20 - Immediate detection

**Total Duration:** ~7 minutes

**Key Message:** "One facility's experience protects all others within hours"

### Scenario 2: Attack Prediction

**Steps:**
1. Inject discovery technique (T0846)
2. Wait for detection (10 seconds)
3. GNN generates prediction
   - T0843 (Program Download) - 67% probability
   - T0800 (Lateral Movement) - 23% probability
   - T0858 (Change Operating Mode) - 10% probability
4. Dashboard shows predicted attack path
5. Wait for predicted timeframe (45 seconds)
6. Attacker proceeds with T0843
7. Prediction validated

**Expected Timeline:**
- 0:00 - Discovery (T0846)
- 0:10 - Detection and alert
- 0:15 - GNN prediction displayed
- 1:00 - Attacker proceeds with T0843
- 1:10 - Prediction validated

**Total Duration:** ~2 minutes

**Key Message:** "Don't just react to attacks - predict and prevent them"

---

## Deployment

### Docker Compose Services

**Infrastructure:**
- Kafka + Zookeeper (message bus)
- IoTDB 1.3.0 (time-series database)
- PostgreSQL 15 (relational database)
- Neo4j 5.x (graph database)

**Application Services:**
- FL Server (port 8080)
- FL Clients (3 instances: facility_a, facility_b, facility_c)
- Detection Service
- Prediction Service
- API Gateway (port 8000)
- Simulator
- Dashboard (port 3000)

### System Requirements

**Minimum:**
- CPU: 4 cores
- RAM: 16 GB
- Disk: 20 GB
- OS: Ubuntu 22.04 / macOS 13+ / Windows 11 with WSL2

**Recommended:**
- CPU: 8 cores
- RAM: 32 GB
- Disk: 50 GB

### Deployment Steps

1. Clone repository
2. Start all services with docker-compose
3. Wait for services to be ready (2-3 minutes)
4. Initialize databases
5. Load MITRE ATT&CK data
6. Verify deployment
7. Access dashboard at http://localhost:3000
8. Run demo scenarios

---

## Testing Strategy

### Unit Tests
**Framework:** pytest

**Coverage:**
- Detection models (LSTM, IF, Physics)
- Correlation engine logic
- GNN prediction accuracy
- API endpoint responses
- Database operations

### Integration Tests
**Scenarios:**
- End-to-end detection flow
- FL round completion
- WebSocket real-time updates
- Demo scenario execution

### Performance Tests
**Framework:** locust

**Targets:**
- Kafka throughput: 1000 msg/sec
- Detection latency: < 30 seconds
- API response time: < 100ms (p95)
- Dashboard update latency: < 1 second

### Demo Rehearsal Tests
**Purpose:** Ensure demo scenarios work reliably

**Process:**
- Run each scenario 10 times
- Verify success rate > 95%
- Measure timing consistency
- Identify and fix failure modes

---

## Performance Targets

### Detection Latency
- Isolation Forest: < 5 seconds
- LSTM: < 30 seconds
- Physics Rules: < 1 second
- Correlation: < 5 seconds
- **Total: < 30 seconds from attack to alert**

### FL Round Duration
- Local training: 3-5 minutes
- Aggregation: < 30 seconds
- Distribution: < 30 seconds
- **Total: 5-6 minutes per round**

### Dashboard Responsiveness
- API response time: < 100ms (p95)
- WebSocket latency: < 1 second
- Page load time: < 2 seconds

### Throughput
- Kafka: 1000 messages/second
- Detection: 100 data points/second
- Database writes: 500 inserts/second

---

## Implementation Priorities

### Week 1-2: Foundation (CRITICAL PATH)
**Goal:** Infrastructure + basic detection

**Deliverables:**
- Docker Compose setup
- Kafka + PostgreSQL + IoTDB + Neo4j
- Modbus simulator
- LSTM Autoencoder (basic)
- Isolation Forest
- Physics Rules Engine
- Correlation Engine
- Alert storage

**Success Criteria:**
- All containers running
- Simulator generates data
- Detection creates alerts
- Alerts stored in database

### Week 3: Federated Learning (CRITICAL PATH)
**Goal:** FL working end-to-end

**Deliverables:**
- FL Server (Flower)
- FL Client (3 instances)
- Differential Privacy (Opacus)
- Model distribution
- Aggregation
- Round tracking

**Success Criteria:**
- FL round completes successfully
- 3 clients participate
- Model updated and distributed
- Privacy metrics tracked

### Week 4: Attack Prediction (CRITICAL PATH)
**Goal:** GNN prediction working

**Deliverables:**
- Load MITRE ATT&CK into Neo4j
- Implement GAT model
- Train on attack sequences
- Prediction API
- Integrate with detection

**Success Criteria:**
- GNN predicts next techniques
- Predictions stored
- API returns predictions

### Week 5: Dashboard (CRITICAL PATH)
**Goal:** Visual demonstration

**Deliverables:**
- React app setup
- Alerts page
- FL status page
- Attack graph visualization
- WebSocket integration
- Real-time updates

**Success Criteria:**
- Dashboard displays alerts
- FL status visible
- Attack graph interactive
- Real-time updates work

### Week 6: Integration & Demo
**Goal:** End-to-end demo scenarios

**Deliverables:**
- Implement demo scenarios
- Test reliability (10x runs)
- Performance optimization
- Documentation
- Video recording
- Bug fixes

**Success Criteria:**
- Demo scenarios work reliably
- Documentation complete
- Video recorded
- System stable

---

## Risk Mitigation

### Risk 1: FL Round Fails
**Probability:** Medium
**Impact:** Critical

**Mitigation:**
- Implement retry logic
- Fallback to manual model distribution
- Extensive testing of FL rounds

**Fallback Demo:**
- Show simulated FL with pre-recorded progress
- Explain what would happen in real scenario

### Risk 2: Detection Methods Don't Work
**Probability:** Low
**Impact:** High

**Mitigation:**
- Use proven algorithms (LSTM, IF are well-established)
- Test with synthetic data first
- Have 3 methods (redundancy)

**Fallback Demo:**
- Demonstrate with 2 out of 3 methods
- Still shows multi-layered approach

### Risk 3: Dashboard Not Ready
**Probability:** Low
**Impact:** Medium

**Mitigation:**
- Start dashboard early (Week 5)
- Use component library (MUI) for speed
- Focus on functionality over aesthetics

**Fallback Demo:**
- Use command-line demo with logs
- Show API responses
- Less impressive but proves functionality

### Risk 4: Integration Issues
**Probability:** Medium
**Impact:** High

**Mitigation:**
- Integration testing throughout
- Weekly integration checkpoints
- Docker Compose for consistency

**Fallback Demo:**
- Demonstrate components separately
- Show each capability independently

---

## Success Metrics

### Minimum Success (Must Achieve)
- ✅ FL round completes with 3 clients
- ✅ At least 2 detection methods working
- ✅ Dashboard shows alerts and FL status
- ✅ Can demonstrate collaborative defense

### Target Success (Goal)
- ✅ All 3 detection methods working
- ✅ Correlation engine combining signals
- ✅ Attack prediction (GNN) working
- ✅ Polished dashboard with real-time updates
- ✅ Privacy demonstration clear

### Stretch Success (Bonus)
- ✅ Multiple attack scenarios
- ✅ Performance metrics dashboard
- ✅ Forensics capability
- ✅ Video walkthrough polished

---

## Key Messages for Demo

### 1. Unique Value Proposition
**"First federated learning system for ICS threat detection"**
- Novel approach to collaborative defense
- Addresses data sharing barriers

### 2. Speed Advantage
**"6 hours vs weeks/months for traditional threat intelligence"**
- Automated, real-time learning
- No manual threat intelligence sharing

### 3. Privacy Guarantee
**"Mathematical privacy guarantees (ε=2.0, δ=10⁻⁵)"**
- Differential privacy
- Raw data never leaves facility

### 4. Multi-Layered Detection
**"Defense in depth: 3 detection methods working together"**
- LSTM for behavioral anomalies
- Isolation Forest for outliers
- Physics rules for safety violations
- Low false positives when all agree

### 5. Proactive Defense
**"Predict attacker's next move before they make it"**
- GNN learns attack patterns
- Shifts from reactive to proactive
- Federated learning improves predictions

---

## Documentation Deliverables

### Must Have
1. **README.md** - Quick start guide
2. **DEPLOYMENT.md** - Detailed deployment steps
3. **DEMO-GUIDE.md** - How to run demo scenarios
4. **API-DOCS** - Auto-generated from FastAPI

### Nice to Have
5. Architecture diagram (high-level)
6. Demo video (5-10 minutes)
7. Presentation slides

---

## Conclusion

This MVP design focuses exclusively on the 5 MUST HAVE features identified in demo-priorities.md:

1. ✅ **Federated Learning** - The killer feature
2. ✅ **Real-Time Detection** - Proof it works (3 methods)
3. ✅ **Dashboard** - Visual demonstration
4. ✅ **Privacy** - Trust factor
5. ✅ **Attack Prediction** - Wow factor

By eliminating non-essential components and focusing on core functionality, we reduce complexity and deliver a working demonstration that proves the unique value proposition: **collaborative ICS threat detection through federated learning, faster than traditional methods, with mathematical privacy guarantees and proactive defense capabilities.**

**Timeline:** 6 weeks
**Effort:** ~240 hours
**Team:** 4 developers
**Deadline:** November 30, 2025

---

**Document Version:** 1.0  
**Last Updated:** October 14, 2025  
**Status:** Ready for Implementation
