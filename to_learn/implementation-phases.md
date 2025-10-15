# Implementation Phases

## Overview

The Federated ICS Threat Correlation Engine is built in 4 phases over 3 months (October - December), progressing from foundation to production-ready system.

## Phase 1: Foundation & Core Detection (October)

### Goal
Build the foundational infrastructure and basic detection capabilities.

### Deliverables

**1. Docker Environment**
```
- Docker Compose setup
- Container orchestration
- Development environment
- Easy deployment for demo
```

**2. Protocol Parsers**
```
Industrial protocols:
- Modbus (TCP/RTU)
- DNP3
- OPC-UA
- S7 (Siemens)

Libraries:
- pyModbus
- dnp3
- opcua
- python-snap7
```

**3. Database Stack**
```
Apache IoTDB:
- Time-series sensor data
- High-frequency measurements

PostgreSQL:
- Alerts and incidents
- Metadata and configurations
- User management

MongoDB:
- Logs and events
- Protocol messages
- Semi-structured data

Neo4j:
- Attack graphs (prepared for Phase 2)
- MITRE ATT&CK relationships
```

**4. Data Ingestion with Kafka**
```
Apache Kafka setup:
- Message broker
- Topics for each data source
- Producer/consumer architecture
- Real-time data streaming
```

**5. LSTM Autoencoder**
```
Behavioral anomaly detection:
- Architecture: Encoder-Decoder
- Input: Time-series sequences
- Output: Reconstruction error
- Threshold-based detection
```

**6. Isolation Forest**
```
Point anomaly detection:
- Unsupervised learning
- Fast outlier detection
- Complements LSTM
```

**7. Physics Rules Engine**
```
Process-aware detection:
- Configurable rules
- Facility-specific constraints
- Safety thresholds
- Process validation
```

**8. Basic Dashboard**
```
Web interface:
- Real-time alerts
- System status
- Basic visualizations
- Alert management
```

### Success Criteria
- ✓ All components running in Docker
- ✓ Data flows from parsers → Kafka → databases
- ✓ LSTM and Isolation Forest detect anomalies
- ✓ Physics rules validate process constraints
- ✓ Dashboard displays alerts

### Timeline
4 weeks (October)

---

## Phase 2: Advanced Features & Federated Learning (November)

### Goal
Add advanced detection capabilities and implement federated learning infrastructure.

### Deliverables

**1. Protocol Anomaly Detection**
```
Deep packet inspection:
- Unusual function codes
- Invalid sequences
- Malformed messages
- Protocol violations
```

**2. Correlation Engine**
```
Multi-source correlation:
- Combine LSTM, Isolation Forest, Physics Rules
- Temporal correlation
- Spatial correlation
- Unified threat assessment
- Severity scoring
```

**3. Graph Neural Network for Prediction**
```
Attack prediction:
- GAT architecture (2 layers, 4 heads)
- MITRE ATT&CK graph
- Next-step prediction
- Probability distribution output
```

**4. Neo4j Attack Graph**
```
MITRE ATT&CK integration:
- Technique nodes
- Relationship edges
- Attack chain visualization
- Path analysis
```

**5. FL Client Module**
```
Facility-side federated learning:
- Local model training
- Weight computation
- Privacy protection (DP noise)
- Secure communication with server
```

**6. FL Aggregation Server**
```
Central coordination:
- Model distribution
- Weight aggregation
- Global model updates
- Round management
```

**7. Differential Privacy Engine**
```
Privacy protection:
- Noise generation (ε=2.0, δ=10⁻⁵)
- Calibrated to privacy budget
- Applied before transmission
- Opacus integration
```

### Success Criteria
- ✓ Correlation engine combines multiple signals
- ✓ GNN predicts next attack steps
- ✓ FL client trains models locally
- ✓ FL server aggregates updates
- ✓ Differential privacy applied to weights

### Timeline
4 weeks (November)

---

## Phase 3: Integration & Testing (December Weeks 1-2)

### Goal
Integrate all components, add security features, and validate system performance.

### Deliverables

**1. Forensic System**
```
Investigation capabilities:
- Always-on capture (protocol messages, metadata)
- Selective deep capture (full packets)
- Timeline reconstruction
- Evidence preservation
- Query interface
```

**2. Red Team Simulator**
```
Continuous validation:
- Mirror production traffic
- Inject attack scenarios (15+ types)
- Validate detection
- Measure coverage
- MITRE ATT&CK testing
```

**3. Secure Aggregation**
```
Enhanced privacy:
- Pairwise masking
- Server cannot see individual updates
- Cryptographic protection
- Key management
```

**4. Byzantine-Robust Aggregation**
```
Attack resistance:
- Outlier detection
- Coordinate-wise median
- Statistical filtering
- Malicious update exclusion
```

**5. End-to-End Integration**
```
Complete data flow:
- Protocols → Kafka → Detection → Correlation → Alerts
- Local training → FL client → FL server → Model update
- Attack → Detection → Prediction → Response
```

**6. Performance Optimization**
```
Tuning:
- Database query optimization
- Kafka throughput tuning
- Model inference speed
- Memory usage optimization
- Latency reduction (<30s target)
```

**7. Security Hardening**
```
Production security:
- TLS 1.3 encryption
- Mutual authentication
- API key rotation
- Rate limiting
- Network isolation
- Audit logging
```

**8. FL Validation**
```
Multi-facility testing:
- Simulated facilities (3-5)
- FL rounds execution
- Privacy verification
- Byzantine attack testing
- Performance measurement
```

### Success Criteria
- ✓ All components integrated
- ✓ Detection latency <30 seconds
- ✓ FL rounds complete successfully
- ✓ Red team validates 87% ATT&CK coverage
- ✓ Security measures in place
- ✓ System stable under load

### Timeline
2 weeks (December weeks 1-2)

---

## Phase 4: Demo & Deployment (December Weeks 3-4)

### Goal
Prepare for demonstration, create documentation, and enable pilot deployment.

### Deliverables

**1. API Gateway**
```
External interface:
- RESTful API
- GraphQL endpoint
- WebSocket for real-time
- Authentication/authorization
- Rate limiting
- API documentation
```

**2. Documentation**
```
Complete documentation:
- Architecture overview
- Deployment guide
- User manual
- API reference
- Configuration guide
- Troubleshooting
```

**3. Demonstration Scenarios**
```
Prepared demos:
- Normal operation
- Attack detection
- Attack prediction
- Federated learning round
- Multi-facility collaboration
- Forensic investigation
```

**4. Interactive Dashboards**
```
Enhanced UI:
- Real-time monitoring
- Attack visualization
- FL round status
- Performance metrics
- Attack graph explorer
- Forensic timeline
```

**5. Presentation Materials**
```
For stakeholders:
- Executive summary
- Technical deep-dive
- Live demo script
- Video demonstrations
- Performance results
- ROI analysis
```

**6. Pilot Deployment**
```
Production-ready:
- Deployment scripts
- Configuration templates
- Monitoring setup
- Backup/recovery
- Scaling guide
```

**7. FL Demonstration**
```
Multi-site showcase:
- 3-5 simulated facilities
- Real-time FL rounds
- Attack propagation demo
- Privacy preservation demo
- Performance metrics
```

### Success Criteria
- ✓ Complete documentation
- ✓ Polished demonstrations
- ✓ API fully functional
- ✓ Ready for pilot deployment
- ✓ Stakeholder presentations prepared

### Timeline
2 weeks (December weeks 3-4)

---

## Technology Stack by Phase

### Phase 1
```
Backend: Python 3.9+, FastAPI
Protocols: pyModbus, dnp3, opcua, python-snap7
Messaging: Apache Kafka
Storage: IoTDB, PostgreSQL, MongoDB, Neo4j
ML: TensorFlow/PyTorch, scikit-learn
Frontend: React 18+, TypeScript
Deployment: Docker, Docker Compose
```

### Phase 2
```
Add:
FL: Flower (flwr), TensorFlow Federated
Privacy: Opacus (differential privacy)
GNN: PyTorch Geometric
Visualization: D3.js for attack graphs
```

### Phase 3
```
Add:
Security: CrypTen (secure aggregation)
Testing: pytest, locust (load testing)
Monitoring: Prometheus, Grafana
```

### Phase 4
```
Add:
API: FastAPI, GraphQL (Strawberry)
Docs: Sphinx, MkDocs
Deployment: Kubernetes (optional)
```

## Risk Mitigation

### Technical Risks

**Risk: Integration complexity**
```
Mitigation:
- Modular architecture
- Well-defined interfaces
- Incremental integration
- Continuous testing
```

**Risk: Performance issues**
```
Mitigation:
- Early performance testing
- Optimization in Phase 3
- Scalable architecture
- Caching strategies
```

**Risk: FL implementation challenges**
```
Mitigation:
- Use proven frameworks (Flower)
- Start simple, add complexity
- Simulated testing before real deployment
- Expert consultation
```

### Schedule Risks

**Risk: Phase delays**
```
Mitigation:
- Buffer time in schedule
- Parallel workstreams
- MVP approach (core features first)
- Regular progress reviews
```

**Risk: Scope creep**
```
Mitigation:
- Clear phase deliverables
- Change control process
- Focus on core functionality
- Defer nice-to-haves
```

## Success Metrics

### Phase 1
- All components deployed and running
- Data ingestion working
- Basic detection functional

### Phase 2
- FL rounds completing successfully
- GNN making predictions
- Correlation engine operational

### Phase 3
- Detection latency <30 seconds
- 87% MITRE ATT&CK coverage
- FL privacy validated
- System stable

### Phase 4
- Documentation complete
- Demos polished
- Pilot-ready deployment

## Key Milestones

```
Week 4 (End of October):
✓ Phase 1 complete - Foundation ready

Week 8 (End of November):
✓ Phase 2 complete - FL operational

Week 10 (Mid December):
✓ Phase 3 complete - System integrated

Week 12 (End of December):
✓ Phase 4 complete - Demo ready
```

## Post-Implementation

### Phase 5 (Future): Production Deployment
```
- Kubernetes migration
- Multi-tenant support
- Cloud deployment
- Real facility pilots
- Industry consortium
```

### Phase 6 (Future): Enterprise Scale
```
- Global FL network
- 1000+ devices
- SaaS offering
- Advanced analytics
- Threat intelligence marketplace
```

## Key Takeaways

1. **4 phases over 3 months** - October to December
2. **Phase 1: Foundation** - Core detection
3. **Phase 2: FL implementation** - Collaborative learning
4. **Phase 3: Integration** - Security and testing
5. **Phase 4: Demo** - Production-ready
6. **Incremental approach** - Build, test, integrate
7. **Risk mitigation** - Modular, tested, proven tech
8. **Clear milestones** - Measurable progress

## Further Reading

- Project management: Agile methodology
- DevOps: CI/CD pipelines
- Testing: Test-driven development
- Documentation: Technical writing best practices
