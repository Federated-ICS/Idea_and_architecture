# Design Document

## Overview

This design document outlines the technical architecture, technology stack, and implementation approach for delivering a demonstrable prototype of the Federated ICS Threat Correlation Engine by November 30, 2025.

**Design Philosophy:**
- **MVP First:** Focus on core features that demonstrate key capabilities
- **Modular Architecture:** Independent components that can be developed in parallel
- **Proven Technologies:** Use battle-tested open-source frameworks
- **Demo-Driven:** Design for effective demonstration and evaluation
- **Incremental Delivery:** Weekly milestones with working features

**Timeline:** 6 weeks (October 13 - November 30, 2025)

---

## Architecture

### High-Level System Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Presentation Layer                        │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │ Web Dashboard│  │   REST API   │  │  WebSocket   │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
                              │
┌─────────────────────────────────────────────────────────────┐
│              Federated Learning Coordination                 │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │  FL Server   │  │  FL Client   │  │Privacy Engine│      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
                              │
┌─────────────────────────────────────────────────────────────┐
│              Detection & Analysis Layer                      │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐       │
│  │   LSTM   │ │Isolation │ │ Physics  │ │   GNN    │       │
│  │Autoencoder│ │  Forest  │ │  Rules   │ │Prediction│       │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘       │
│  ┌────────────────────────────────────────────────┐         │
│  │         Correlation Engine                      │         │
│  └────────────────────────────────────────────────┘         │
└─────────────────────────────────────────────────────────────┘
                              │
┌─────────────────────────────────────────────────────────────┐
│                   Messaging Layer                            │
│  ┌────────────────────────────────────────────────┐         │
│  │            Apache Kafka Message Bus             │         │
│  └────────────────────────────────────────────────┘         │
└─────────────────────────────────────────────────────────────┘
                              │
┌─────────────────────────────────────────────────────────────┐
│                    Storage Layer                             │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐       │
│  │  IoTDB   │ │PostgreSQL│ │ MongoDB  │ │  Neo4j   │       │
│  │(Sensors) │ │ (Alerts) │ │  (Logs)  │ │ (Graph)  │       │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘       │
└─────────────────────────────────────────────────────────────┘
                              │
┌─────────────────────────────────────────────────────────────┐
│                  Data Sources Layer                          │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐       │
│  │  Modbus  │ │   DNP3   │ │ Network  │ │ Simulated│       │
│  │  Parser  │ │  Parser  │ │ Capture  │ │ Sensors  │       │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘       │
└─────────────────────────────────────────────────────────────┘
```


### Component Interaction Flow

**1. Data Ingestion Flow:**
```
Modbus Device → Parser → Kafka Topic → Consumer → Database
                                     ↓
                              Detection Models
```

**2. Detection Flow:**
```
Kafka → LSTM/IF/Physics → Anomaly Score → Correlation → Alert → Dashboard
                                                       ↓
                                                  PostgreSQL
```

**3. Federated Learning Flow:**
```
FL Server → Global Model → FL Client → Local Training → Weight Update
    ↑                                                         ↓
    └─────────────── Aggregation ←──── Privacy Noise ────────┘
```

**4. Attack Prediction Flow:**
```
Alert → MITRE Technique → Neo4j Graph → GNN → Next Step Prediction → Dashboard
```

---

## Technology Stack

### Core Technologies (with Justification)

#### Backend Framework
**Selected: Python 3.11 + FastAPI**
- **Why Python 3.11:** Rich ML/data science ecosystem, rapid development, ~25% faster than 3.9
- **Why FastAPI:** Modern async framework, auto-generated API docs, WebSocket support
- **Alternatives Considered:** Node.js (less ML support), Go (steeper learning curve)

#### Message Broker
**Selected: Apache Kafka 3.6**
- **Why Kafka:** Industry standard, high throughput, message persistence
- **Configuration:** 3 brokers for demo, 6 partitions per topic
- **Alternatives Considered:** RabbitMQ (lower throughput), Redis Streams (less mature)

#### Time-Series Database
**Selected: Apache IoTDB 1.2**
- **Why IoTDB:** Purpose-built for IoT/ICS, excellent compression, SQL-like queries
- **Use Case:** Store sensor readings (temperature, pressure, flow rates)
- **Alternatives Considered:** InfluxDB (commercial licensing), TimescaleDB (PostgreSQL extension)

#### Relational Database
**Selected: PostgreSQL 15**
- **Why PostgreSQL:** Robust, JSONB support, excellent performance
- **Use Case:** Alerts, incidents, user management, configurations
- **Alternatives Considered:** MySQL (less feature-rich), SQLite (not production-ready)

#### Document Database
**Selected: MongoDB 7.0**
- **Why MongoDB:** Flexible schema, good for logs and protocol messages
- **Use Case:** Protocol messages, system logs, semi-structured data
- **Alternatives Considered:** Elasticsearch (overkill for MVP), CouchDB (less popular)

#### Graph Database
**Selected: Neo4j 5.x Community Edition**
- **Why Neo4j:** Best-in-class graph database, Cypher query language, visualization
- **Use Case:** MITRE ATT&CK relationships, attack path analysis
- **Alternatives Considered:** ArangoDB (multi-model complexity), JanusGraph (harder setup)

#### Machine Learning Framework
**Selected: PyTorch 2.1 + TensorFlow 2.14**
- **Why Both:** PyTorch for GNN (PyTorch Geometric), TensorFlow for LSTM (Keras API)
- **Why PyTorch:** Dynamic graphs, better for research/prototyping
- **Why TensorFlow:** Mature ecosystem, TensorFlow Federated support
- **Alternatives Considered:** JAX (less mature ecosystem)

#### Federated Learning Framework
**Selected: Flower (flwr) 1.6**
- **Why Flower:** Framework-agnostic, easy to use, active development
- **Features:** Supports PyTorch and TensorFlow, built-in strategies
- **Alternatives Considered:** TensorFlow Federated (TF-only), PySyft (complex setup)

#### Differential Privacy
**Selected: Opacus 1.4 (PyTorch) + TensorFlow Privacy**
- **Why Opacus:** Official PyTorch privacy library, easy integration
- **Why TF Privacy:** Official TensorFlow privacy library
- **Features:** Automatic DP-SGD, privacy accounting, gradient clipping

#### Frontend Framework
**Selected: React 18 + TypeScript + Vite**
- **Why React:** Component-based, large ecosystem, fast development
- **Why TypeScript:** Type safety, better IDE support
- **Why Vite:** Fast build times, modern tooling
- **UI Library:** Material-UI (MUI) for rapid prototyping

#### Visualization
**Selected: Recharts + D3.js**
- **Why Recharts:** React-friendly, easy charts
- **Why D3.js:** Custom visualizations (attack graphs, network topology)
- **Alternatives Considered:** Chart.js (less flexible), Plotly (heavier)

#### Containerization
**Selected: Docker 24.x + Docker Compose**
- **Why Docker:** Industry standard, reproducible environments
- **Why Compose:** Simple orchestration for demo/development
- **Future:** Kubernetes for production (out of scope for MVP)

#### Protocol Libraries
- **Modbus:** pyModbus 3.5 (most mature Python Modbus library)
- **DNP3:** dnp3-python (Python bindings for OpenDNP3)
- **Network:** scapy 2.5 (packet manipulation and capture)
- **Simulation:** OpenPLC + Modbus simulators

---

## Components and Interfaces

### 1. Data Sources Layer

#### Modbus Parser Service
**Technology:** Python 3.11, pyModbus, asyncio
**Responsibilities:**
- Connect to Modbus TCP devices (port 502)
- Poll registers at configurable intervals (1-10 seconds)
- Parse function codes (Read Coils, Read Holding Registers, Write)
- Publish to Kafka topic: `protocol.modbus`

**Configuration:**
```yaml
modbus:
  devices:
    - name: "PLC-REACTOR-01"
      host: "192.168.1.10"
      port: 502
      unit_id: 1
      poll_interval: 5
      registers:
        - address: 1000
          count: 10
          type: holding
```

**Output Format:**
```json
{
  "timestamp": "2025-10-13T14:23:45Z",
  "device": "PLC-REACTOR-01",
  "function_code": 3,
  "address": 1000,
  "value": 42,
  "unit_id": 1
}
```

#### Sensor Data Simulator
**Technology:** Python 3.11, NumPy
**Responsibilities:**
- Generate realistic sensor data (temperature, pressure, flow)
- Inject anomalies on demand for demo scenarios
- Publish to Kafka topic: `sensors.data`

**Simulation Modes:**
- Normal operation (baseline patterns)
- Anomaly injection (spikes, drifts, outliers)
- Attack scenarios (coordinated anomalies)


### 2. Storage Layer

#### Apache IoTDB Schema
```sql
-- Time-series storage for sensor data
CREATE DATABASE root.facility_a;

CREATE TIMESERIES root.facility_a.reactor.temperature 
  WITH DATATYPE=FLOAT, ENCODING=GORILLA, COMPRESSION=SNAPPY;

CREATE TIMESERIES root.facility_a.reactor.pressure 
  WITH DATATYPE=FLOAT, ENCODING=GORILLA, COMPRESSION=SNAPPY;

CREATE TIMESERIES root.facility_a.pump.flow_rate 
  WITH DATATYPE=FLOAT, ENCODING=GORILLA, COMPRESSION=SNAPPY;
```

#### PostgreSQL Schema
```sql
-- Alerts table
CREATE TABLE alerts (
    alert_id VARCHAR(50) PRIMARY KEY,
    timestamp TIMESTAMP NOT NULL,
    severity VARCHAR(20) NOT NULL,
    confidence FLOAT NOT NULL,
    title TEXT NOT NULL,
    description TEXT,
    affected_assets TEXT[],
    techniques TEXT[],
    sources TEXT[],
    status VARCHAR(20) DEFAULT 'open',
    created_at TIMESTAMP DEFAULT NOW()
);

-- Incidents table
CREATE TABLE incidents (
    incident_id VARCHAR(50) PRIMARY KEY,
    start_time TIMESTAMP NOT NULL,
    end_time TIMESTAMP,
    severity VARCHAR(20) NOT NULL,
    status VARCHAR(20) DEFAULT 'investigating',
    alert_ids TEXT[],
    timeline JSONB,
    created_at TIMESTAMP DEFAULT NOW()
);

-- FL rounds tracking
CREATE TABLE fl_rounds (
    round_id SERIAL PRIMARY KEY,
    start_time TIMESTAMP NOT NULL,
    end_time TIMESTAMP,
    num_clients INTEGER,
    model_version VARCHAR(50),
    metrics JSONB,
    status VARCHAR(20) DEFAULT 'in_progress'
);
```

#### MongoDB Collections
```javascript
// Protocol messages collection
db.protocol_messages.createIndex({ "timestamp": 1 });
db.protocol_messages.createIndex({ "device": 1, "timestamp": 1 });

// System logs collection
db.system_logs.createIndex({ "timestamp": 1 });
db.system_logs.createIndex({ "level": 1, "timestamp": 1 });
```

#### Neo4j Graph Schema
```cypher
// MITRE ATT&CK for ICS techniques
CREATE (t:Technique {
  id: 'T0846',
  name: 'Remote System Discovery',
  tactic: 'Discovery'
});

// Relationships between techniques
CREATE (t1:Technique {id: 'T0846'})-[:LEADS_TO {probability: 0.67}]->(t2:Technique {id: 'T0843'});

// Asset nodes
CREATE (a:Asset {
  id: 'PLC-REACTOR-01',
  type: 'PLC',
  criticality: 'HIGH'
});
```

### 3. Detection & Analysis Layer

#### LSTM Autoencoder
**Architecture:**
```python
# Encoder
Input(60, 10) → LSTM(128) → LSTM(64) → Latent(64)

# Decoder
Latent(64) → LSTM(64) → LSTM(128) → Output(60, 10)
```

**Hyperparameters:**
- Sequence length: 60 time steps (1 minute at 1Hz)
- Features: 10 (temperature, pressure, flow, etc.)
- Batch size: 32
- Learning rate: 0.001
- Threshold: 95th percentile of reconstruction error

**Training:**
- Local training: 5 epochs per FL round
- Optimizer: Adam
- Loss: Mean Squared Error (MSE)

**Federated:** Yes - Model weights shared across facilities

#### Isolation Forest
**Configuration:**
```python
IsolationForest(
    n_estimators=100,
    max_samples=256,
    contamination=0.01,
    max_features=1.0,
    random_state=42
)
```

**Features:** Same 10 features as LSTM
**Threshold:** Anomaly score > 0.6
**Federated:** Yes - Ensemble shared

#### Physics Rules Engine
**Rule Format:**
```yaml
rules:
  - id: "RULE-001"
    name: "Reactor Temperature Range"
    type: range
    sensor: "reactor.temperature"
    min: 250
    max: 350
    unit: "celsius"
    severity: "CRITICAL"
    
  - id: "RULE-002"
    name: "Pressure Rate Limit"
    type: rate
    sensor: "reactor.pressure"
    max_rate: 10
    unit: "psi_per_minute"
    severity: "WARNING"
    
  - id: "RULE-003"
    name: "Pump Dependency"
    type: dependency
    condition: "valve.inlet == OPEN"
    requirement: "pump.status == RUNNING"
    severity: "CRITICAL"
```

**Federated:** No - Facility-specific

#### Correlation Engine
**Algorithm:**
```python
def correlate_alerts(alerts, time_window=300):
    """
    Correlate alerts within time window (5 minutes)
    """
    correlated = []
    
    for alert in alerts:
        # Temporal correlation
        nearby = find_alerts_in_window(alert, time_window)
        
        # Spatial correlation
        same_asset = [a for a in nearby if a.asset == alert.asset]
        
        # Multi-source correlation
        sources = set([a.source for a in same_asset])
        
        # Calculate confidence
        confidence = base_confidence * (1 + 0.2 * len(sources))
        
        # Determine severity
        if len(sources) >= 3:
            severity = "CRITICAL"
        elif len(sources) >= 2:
            severity = "HIGH"
        else:
            severity = alert.severity
            
        correlated.append({
            "alert_id": generate_id(),
            "confidence": min(confidence, 1.0),
            "severity": severity,
            "sources": list(sources),
            "related_alerts": [a.id for a in same_asset]
        })
    
    return correlated
```

#### Graph Neural Network (Attack Prediction)
**Architecture:**
```python
# Graph Attention Network (GAT)
Input(num_techniques, feature_dim) 
  → GATConv(64, heads=4) 
  → Dropout(0.6)
  → GATConv(32, heads=4)
  → Dense(num_techniques)
  → Softmax
```

**Features per Technique:**
- One-hot encoding of technique ID
- Tactic embedding
- Historical frequency
- Time since last seen

**Training:**
- Attack chain sequences from MITRE ATT&CK
- Supervised learning (next technique prediction)
- Loss: Cross-entropy

**Federated:** Yes - Learns attack patterns across facilities


### 4. Federated Learning Layer

#### FL Server
**Technology:** Python 3.11, Flower (flwr), FastAPI

**Responsibilities:**
- Schedule FL rounds (every 6 hours for demo)
- Distribute global model to clients
- Collect weight updates from clients
- Aggregate using coordinate-wise median
- Detect and exclude outliers (Byzantine-robust)
- Track metrics and round history

**API Endpoints:**
```python
POST /fl/start_round          # Initiate new FL round
GET  /fl/rounds               # List all rounds
GET  /fl/rounds/{id}          # Get round details
GET  /fl/rounds/{id}/metrics  # Get round metrics
POST /fl/clients/register     # Register new client
GET  /fl/clients              # List connected clients
```

**Aggregation Strategy:**
```python
from flwr.server.strategy import FedMedian

strategy = FedMedian(
    fraction_fit=1.0,           # Use all available clients
    fraction_evaluate=1.0,
    min_fit_clients=3,          # Minimum 3 clients
    min_available_clients=3,
    evaluate_fn=evaluate_global_model,
    on_fit_config_fn=fit_config,
)
```

#### FL Client
**Technology:** Python 3.11, Flower (flwr), Opacus

**Responsibilities:**
- Receive global model from server
- Load local training data
- Train for 5 epochs
- Apply differential privacy (DP-SGD)
- Compute weight updates
- Send updates to server

**Privacy Configuration:**
```python
from opacus import PrivacyEngine

privacy_engine = PrivacyEngine()

model, optimizer, data_loader = privacy_engine.make_private(
    module=model,
    optimizer=optimizer,
    data_loader=data_loader,
    noise_multiplier=1.1,      # Calibrated for ε=2.0
    max_grad_norm=1.0,         # Gradient clipping
)

# Track privacy budget
epsilon = privacy_engine.get_epsilon(delta=1e-5)
```

**Training Loop:**
```python
def train_local_model(model, data_loader, epochs=5):
    model.train()
    for epoch in range(epochs):
        for batch in data_loader:
            optimizer.zero_grad()
            loss = compute_loss(model, batch)
            loss.backward()
            optimizer.step()  # DP noise added automatically
    
    return model.state_dict()
```

#### Privacy Engine
**Differential Privacy Parameters:**
- Epsilon (ε): 2.0 (moderate privacy)
- Delta (δ): 10⁻⁵ (failure probability)
- Noise mechanism: Gaussian
- Gradient clipping: L2 norm, max=1.0

**Privacy Accounting:**
```python
from opacus.accountants import RDPAccountant

accountant = RDPAccountant()
epsilon = accountant.get_epsilon(
    delta=1e-5,
    steps=num_training_steps
)
print(f"Privacy budget: ε={epsilon:.2f}")
```

### 5. API & Presentation Layer

#### REST API
**Technology:** FastAPI, Pydantic

**Key Endpoints:**
```python
# Alerts
GET    /api/alerts                    # List alerts
GET    /api/alerts/{id}               # Get alert details
PUT    /api/alerts/{id}/status        # Update alert status
POST   /api/alerts/{id}/acknowledge   # Acknowledge alert

# Predictions
GET    /api/predictions               # List predictions
GET    /api/predictions/{id}          # Get prediction details

# System
GET    /api/system/status             # System health
GET    /api/system/metrics            # Performance metrics

# Federated Learning
GET    /api/fl/rounds                 # List FL rounds
GET    /api/fl/rounds/{id}            # Round details
POST   /api/fl/rounds/trigger         # Manually trigger round

# Demo
POST   /api/demo/scenarios/{name}     # Trigger demo scenario
GET    /api/demo/scenarios            # List available scenarios
```

#### WebSocket API
**Real-time Updates:**
```javascript
// Client connection
const ws = new WebSocket('ws://localhost:8000/ws');

// Subscribe to alerts
ws.send(JSON.stringify({
  type: 'subscribe',
  channel: 'alerts'
}));

// Receive real-time alerts
ws.onmessage = (event) => {
  const alert = JSON.parse(event.data);
  updateDashboard(alert);
};
```

#### Web Dashboard
**Technology:** React 18, TypeScript, Material-UI, Recharts, D3.js

**Pages:**
1. **Overview Dashboard**
   - Real-time alert feed
   - System status indicators
   - Key metrics (detection rate, FL status)
   - Recent predictions

2. **Alerts Page**
   - Filterable alert list
   - Alert details modal
   - Timeline visualization
   - Acknowledge/resolve actions

3. **Attack Graph**
   - Interactive MITRE ATT&CK graph
   - Current attack path highlighted
   - Predicted next steps
   - Historical attack chains

4. **Federated Learning**
   - FL round history
   - Client participation
   - Model performance metrics
   - Privacy budget tracking

5. **System Monitoring**
   - Component health
   - Resource usage
   - Message throughput
   - Database statistics

**Component Structure:**
```
src/
├── components/
│   ├── AlertCard.tsx
│   ├── AttackGraph.tsx
│   ├── MetricsChart.tsx
│   ├── FLRoundStatus.tsx
│   └── SystemHealth.tsx
├── pages/
│   ├── Dashboard.tsx
│   ├── Alerts.tsx
│   ├── AttackGraph.tsx
│   ├── FederatedLearning.tsx
│   └── System.tsx
├── services/
│   ├── api.ts
│   ├── websocket.ts
│   └── types.ts
└── App.tsx
```

---

## Data Models

### Alert Model
```typescript
interface Alert {
  alert_id: string;
  timestamp: string;
  severity: 'CRITICAL' | 'HIGH' | 'MEDIUM' | 'LOW';
  confidence: number;
  title: string;
  description: string;
  affected_assets: string[];
  techniques: string[];  // MITRE ATT&CK IDs
  sources: string[];     // LSTM, IsolationForest, Physics
  status: 'open' | 'acknowledged' | 'resolved';
  created_at: string;
}
```

### Prediction Model
```typescript
interface Prediction {
  prediction_id: string;
  timestamp: string;
  current_technique: string;
  next_techniques: {
    technique_id: string;
    name: string;
    probability: number;
    tactic: string;
  }[];
  target_assets: string[];
  timeframe: string;
  confidence: number;
}
```

### FL Round Model
```typescript
interface FLRound {
  round_id: number;
  start_time: string;
  end_time?: string;
  num_clients: number;
  model_version: string;
  metrics: {
    accuracy: number;
    loss: number;
    privacy_epsilon: number;
  };
  status: 'in_progress' | 'completed' | 'failed';
}
```

---

## Error Handling

### Detection Pipeline
```python
try:
    # Parse protocol message
    message = parse_modbus(raw_data)
except ParseError as e:
    log.error(f"Parse failed: {e}")
    metrics.increment('parse_errors')
    return None

try:
    # Run detection
    anomaly_score = lstm_model.predict(message)
except ModelError as e:
    log.error(f"Detection failed: {e}")
    metrics.increment('detection_errors')
    # Fall back to rule-based detection
    anomaly_score = rule_based_detect(message)
```

### Federated Learning
```python
try:
    # FL round execution
    results = fl_server.fit(num_rounds=1)
except ClientTimeout as e:
    log.warning(f"Client timeout: {e}")
    # Continue with available clients
    results = fl_server.fit_with_available()
except AggregationError as e:
    log.error(f"Aggregation failed: {e}")
    # Keep previous model
    return previous_model
```

### API Layer
```python
@app.exception_handler(ValidationError)
async def validation_exception_handler(request, exc):
    return JSONResponse(
        status_code=422,
        content={"detail": exc.errors()}
    )

@app.exception_handler(DatabaseError)
async def database_exception_handler(request, exc):
    log.error(f"Database error: {exc}")
    return JSONResponse(
        status_code=500,
        content={"detail": "Internal server error"}
    )
```


## Testing Strategy

### Unit Tests
**Framework:** pytest, pytest-asyncio

**Coverage:**
- Protocol parsers (Modbus, DNP3)
- Detection models (LSTM, Isolation Forest)
- Correlation engine logic
- API endpoints
- Database operations

**Example:**
```python
def test_modbus_parser():
    raw_data = b'\x00\x01\x00\x00\x00\x06\x01\x03\x00\x00\x00\x0a'
    message = parse_modbus(raw_data)
    assert message['function_code'] == 3
    assert message['address'] == 0
    assert message['count'] == 10
```

### Integration Tests
**Scenarios:**
- End-to-end data flow (parser → Kafka → detection → alert)
- FL round execution (server → client → aggregation)
- API + database interactions
- WebSocket real-time updates

### Performance Tests
**Framework:** locust

**Targets:**
- Kafka throughput: 1000 msg/sec
- Detection latency: <30 seconds
- API response time: <100ms (p95)
- Dashboard update latency: <1 second

### Demo Scenarios
**Pre-scripted Attack Scenarios:**

1. **Modbus Manipulation Attack**
   - Unauthorized write to holding registers
   - Physics rules violation
   - LSTM detects behavioral anomaly
   - Correlation creates high-severity alert

2. **Multi-Stage Attack**
   - Discovery (T0846) → Lateral Movement (T0800) → Program Download (T0843)
   - GNN predicts each next step
   - Demonstrates attack prediction capability

3. **Federated Learning Demonstration**
   - Facility A experiences novel attack
   - FL round triggered
   - Facilities B and C receive updated model
   - All facilities now detect the attack pattern

---

## Deployment Architecture

### Docker Compose Setup

**Services:**
```yaml
version: '3.8'

services:
  # Message Broker
  kafka:
    image: confluentinc/cp-kafka:7.5.0
    ports:
      - "9092:9092"
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
  
  zookeeper:
    image: confluentinc/cp-zookeeper:7.5.0
    ports:
      - "2181:2181"
  
  # Databases
  iotdb:
    image: apache/iotdb:1.2.0
    ports:
      - "6667:6667"
    volumes:
      - iotdb-data:/iotdb/data
  
  postgres:
    image: postgres:15
    ports:
      - "5432:5432"
    environment:
      POSTGRES_DB: ics_security
      POSTGRES_USER: ics_user
      POSTGRES_PASSWORD: secure_password
    volumes:
      - postgres-data:/var/lib/postgresql/data
  
  mongodb:
    image: mongo:7.0
    ports:
      - "27017:27017"
    volumes:
      - mongo-data:/data/db
  
  neo4j:
    image: neo4j:5.13
    ports:
      - "7474:7474"
      - "7687:7687"
    environment:
      NEO4J_AUTH: neo4j/password
    volumes:
      - neo4j-data:/data
  
  # Application Services
  data-ingestion:
    build: ./services/data-ingestion
    depends_on:
      - kafka
    environment:
      KAFKA_BOOTSTRAP_SERVERS: kafka:9092
  
  detection-service:
    build: ./services/detection
    depends_on:
      - kafka
      - iotdb
      - postgres
    environment:
      KAFKA_BOOTSTRAP_SERVERS: kafka:9092
      IOTDB_HOST: iotdb
      POSTGRES_HOST: postgres
  
  fl-server:
    build: ./services/fl-server
    ports:
      - "8080:8080"
    depends_on:
      - postgres
  
  fl-client-a:
    build: ./services/fl-client
    environment:
      FL_SERVER_ADDRESS: fl-server:8080
      FACILITY_ID: facility_a
  
  fl-client-b:
    build: ./services/fl-client
    environment:
      FL_SERVER_ADDRESS: fl-server:8080
      FACILITY_ID: facility_b
  
  fl-client-c:
    build: ./services/fl-client
    environment:
      FL_SERVER_ADDRESS: fl-server:8080
      FACILITY_ID: facility_c
  
  api-gateway:
    build: ./services/api
    ports:
      - "8000:8000"
    depends_on:
      - postgres
      - mongodb
      - neo4j
  
  dashboard:
    build: ./frontend
    ports:
      - "3000:3000"
    depends_on:
      - api-gateway

volumes:
  iotdb-data:
  postgres-data:
  mongo-data:
  neo4j-data:
```

### Resource Requirements

**Minimum System:**
- CPU: 4 cores
- RAM: 16 GB
- Disk: 50 GB SSD
- OS: Ubuntu 22.04 / macOS 13+ / Windows 11 with WSL2

**Recommended System:**
- CPU: 8 cores
- RAM: 32 GB
- Disk: 100 GB SSD
- GPU: Optional (speeds up ML training)

### Deployment Steps

```bash
# 1. Clone repository
git clone https://github.com/your-org/federated-ics-engine.git
cd federated-ics-engine

# 2. Build containers
docker-compose build

# 3. Start services
docker-compose up -d

# 4. Initialize databases
./scripts/init-databases.sh

# 5. Load MITRE ATT&CK data
./scripts/load-mitre-attack.sh

# 6. Verify deployment
./scripts/health-check.sh

# 7. Access dashboard
open http://localhost:3000
```

---

## Development Milestones (6 Weeks)

### Week 1 (Oct 13-19): Foundation Setup
**Goal:** Infrastructure and data pipeline

**Deliverables:**
- ✅ Docker Compose environment
- ✅ Kafka cluster (3 brokers)
- ✅ All databases (IoTDB, PostgreSQL, MongoDB, Neo4j)
- ✅ Modbus parser service
- ✅ Sensor data simulator
- ✅ Basic data ingestion pipeline

**Success Criteria:**
- All containers running
- Data flows from simulator → Kafka → databases
- Can query stored data

**Effort:** 40 hours

---

### Week 2 (Oct 20-26): Core Detection
**Goal:** Implement detection models

**Deliverables:**
- ✅ LSTM Autoencoder implementation
- ✅ Isolation Forest implementation
- ✅ Physics Rules Engine
- ✅ Correlation Engine (basic)
- ✅ Alert generation and storage

**Success Criteria:**
- Models detect anomalies in simulated data
- Alerts stored in PostgreSQL
- Correlation combines multiple signals

**Effort:** 40 hours

---

### Week 3 (Oct 27 - Nov 2): Federated Learning Foundation
**Goal:** FL infrastructure

**Deliverables:**
- ✅ FL Server (Flower)
- ✅ FL Client (3 instances)
- ✅ Differential Privacy integration (Opacus)
- ✅ Model serialization and distribution
- ✅ Basic aggregation (FedAvg)

**Success Criteria:**
- FL round completes successfully
- 3 clients train and upload weights
- Server aggregates and distributes model
- Privacy noise applied

**Effort:** 40 hours

---

### Week 4 (Nov 3-9): Attack Prediction & Graph
**Goal:** GNN and MITRE ATT&CK integration

**Deliverables:**
- ✅ Neo4j MITRE ATT&CK graph
- ✅ Graph Neural Network (GAT)
- ✅ Attack prediction service
- ✅ Technique mapping
- ✅ Prediction API endpoints

**Success Criteria:**
- MITRE graph loaded in Neo4j
- GNN predicts next techniques
- Predictions accessible via API

**Effort:** 40 hours

---

### Week 5 (Nov 10-16): API & Dashboard
**Goal:** User interface and API

**Deliverables:**
- ✅ REST API (FastAPI)
- ✅ WebSocket for real-time updates
- ✅ React dashboard (5 pages)
- ✅ Alert visualization
- ✅ Attack graph visualization
- ✅ FL round monitoring

**Success Criteria:**
- Dashboard displays real-time alerts
- Attack graph interactive
- FL status visible
- WebSocket updates work

**Effort:** 40 hours

---

### Week 6 (Nov 17-23): Integration & Demo Prep
**Goal:** End-to-end integration and demo scenarios

**Deliverables:**
- ✅ 3 demo scenarios implemented
- ✅ Performance optimization
- ✅ Documentation (README, API docs, deployment guide)
- ✅ Video walkthrough (5-10 minutes)
- ✅ Presentation slides
- ✅ Bug fixes and polish

**Success Criteria:**
- All demo scenarios work reliably
- Documentation complete
- Video recorded
- System stable

**Effort:** 40 hours

---

### Week 6.5 (Nov 24-30): Buffer & Final Polish
**Goal:** Buffer time for issues and final touches

**Deliverables:**
- ✅ Address any remaining bugs
- ✅ Performance tuning
- ✅ Final documentation review
- ✅ Presentation rehearsal
- ✅ Deployment verification

**Success Criteria:**
- System ready for demonstration
- All deliverables complete
- Confident in demo execution

**Effort:** 20 hours

---

## Total Effort Estimate

**Total Hours:** 260 hours over 6 weeks

**Team Composition Options:**

**Option 1: Solo Developer**
- 260 hours / 6 weeks = ~43 hours/week
- Challenging but feasible with focus

**Option 2: 2 Developers**
- 260 hours / 2 = 130 hours each
- ~22 hours/week per person
- More realistic

**Option 3: 3 Developers**
- 260 hours / 3 = ~87 hours each
- ~15 hours/week per person
- Comfortable pace with buffer

**Recommended:** 2-3 developers for realistic timeline

---

## Risk Mitigation

### Technical Risks

**Risk 1: FL Integration Complexity**
- **Mitigation:** Use Flower framework (proven), start simple
- **Fallback:** Demonstrate FL concept with manual model sharing

**Risk 2: Performance Issues**
- **Mitigation:** Early performance testing, optimize in Week 6
- **Fallback:** Reduce data volume for demo

**Risk 3: ML Model Training Time**
- **Mitigation:** Use pre-trained models, small datasets for demo
- **Fallback:** Use simpler models (reduce LSTM layers)

### Schedule Risks

**Risk 1: Scope Creep**
- **Mitigation:** Strict adherence to MVP scope, defer nice-to-haves
- **Fallback:** Week 6.5 buffer time

**Risk 2: Integration Issues**
- **Mitigation:** Weekly integration testing, modular architecture
- **Fallback:** Prioritize core features, cut secondary features

**Risk 3: Learning Curve**
- **Mitigation:** Use familiar technologies, leverage documentation
- **Fallback:** Simplify complex components

---

## Success Metrics

### Technical Metrics
- ✅ Detection latency: <30 seconds
- ✅ Detection accuracy: >90% on test scenarios
- ✅ False positive rate: <10%
- ✅ FL round duration: <5 minutes (3 clients)
- ✅ API response time: <100ms (p95)
- ✅ Dashboard update latency: <1 second

### Deliverable Metrics
- ✅ All 8 requirements met
- ✅ 3 demo scenarios working
- ✅ Documentation complete
- ✅ Video walkthrough recorded
- ✅ System deployable in <30 minutes

### Demo Quality Metrics
- ✅ Demo runs without errors
- ✅ Visualizations clear and compelling
- ✅ FL collaboration visible
- ✅ Attack prediction demonstrated
- ✅ Real-time detection shown

---

## Next Steps

After design approval, proceed to create the implementation task list with specific coding tasks for each week.

