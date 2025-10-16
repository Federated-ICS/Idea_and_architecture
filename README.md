# Federated ICS Threat Correlation Engine

A privacy-preserving collaborative defense system for Industrial Control Systems (ICS) that enables multiple facilities to learn from each other's security threats without sharing sensitive operational data.

## Project Overview

This project implements a federated learning-based threat detection and correlation engine specifically designed for Industrial Control Systems. The system combines real-time anomaly detection, physics-aware validation, and AI-powered attack prediction to provide proactive security for critical infrastructure.

### Key Features

- **Real-Time Detection**: Multi-source threat detection with physics-aware anomaly detection using LSTM Autoencoders, Isolation Forest, and protocol-specific rules
- **AI-Powered Attack Prediction**: Graph Neural Networks (GNN) predict attacker's next move based on MITRE ATT&CK for ICS
- **Automated Forensic Playback**: Time-travel investigation with evidence reconstruction in under 30 seconds
- **Adversarial Resilience Testing**: Built-in red team mode with continuous validation
- **Federated Learning Network**: Privacy-preserving collaborative defense across facilities with mathematical privacy guarantees (ε=2.0, δ=10⁻⁵)

### Performance Metrics

- Detection Latency: <30 seconds
- Detection Accuracy: >95%
- False Positive Rate: <5%
- Attack Prediction Accuracy: 67%
- MITRE ATT&CK Coverage: 87%
- Threat Propagation Time: 6-24 hours (vs weeks with traditional methods)

---

## Directory Structure

### 📁 `architecture/`

Contains comprehensive system architecture documentation, design specifications, and technical considerations.

**Purpose**: High-level system design, architectural patterns, and technology stack decisions

**Key Files**:
- `5-core-features.md` - Detailed description of the 5 core features of the system
- `critical_architecture_considerations.md` - Industry 4.0 best practices and architectural recommendations
- `system-architecture-design.tex` - LaTeX source for system architecture diagrams
- `november-30-demo-architecture.tex` - Demo-specific architecture documentation
- `technology-comparison.tex` - Comparison of technology choices and alternatives
- `design-document.tex` - Complete design document in LaTeX format

**Use Cases**:
- Understanding the overall system design
- Making architectural decisions
- Reviewing Industry 4.0 compliance
- Planning deployment strategies

---

### 📁 `federated_learning/`

Core federated learning implementation details, design documents, and training strategies.

**Purpose**: Federated learning architecture, privacy mechanisms, and training datasets

**Key Files**:
- `minimal-design.md` - MVP design document focused on 5 MUST HAVE features for November 30 demo
- `system design.md` - Complete technical design document  
- `training-datasets.md` - Comprehensive guide to ICS datasets (SWaT, WADI, Morris & Gao, HAI)
- `ics-report-federated.tex` - LaTeX report on federated learning for ICS security

**Use Cases**:
- Implementing federated learning components
- Understanding privacy-preserving mechanisms
- Selecting and preparing training datasets
- Planning FL rounds and aggregation strategies
- Building MVP demo with essential features only

**Key Topics Covered**:
- Differential privacy (ε=2.0, δ=10⁻⁵)
- Secure aggregation protocols
- Byzantine-robust aggregation
- FL server and client architecture
- Model distribution and weight updates
- Demo-first simplified architecture with IoTDB for time-series data

---

### 📁 `to_learn/`

Educational resources and deep-dive explanations of key technical concepts used in the project.

**Purpose**: Learning materials for understanding the underlying technologies and algorithms

**Key Files**:
- `lstm-temporal-patterns.md` - How LSTM Autoencoders detect temporal anomalies
- `lstm-practical-guide.md` - Practical guide to LSTM for ICS anomaly detection with real-world examples
- `federated-learning-heterogeneous-data.md` - How federated learning works with different facilities and operating ranges
- `federated-learning-missing-sensors.md` - Handling facilities with different sensor configurations
- `isolation-forest.md` - Isolation Forest algorithm for outlier detection
- `gnn-relationship-modeling.md` - Graph Neural Networks for attack prediction
- `differential-privacy.md` - Differential privacy mechanisms and implementation
- `secure-aggregation.md` - Secure aggregation protocols for federated learning
- `byzantine-robust-aggregation.md` - Defending against malicious participants
- `gradient-clipping.md` - Gradient clipping for privacy and stability
- `practical-fl-parameters.md` - Practical federated learning parameter tuning
- `implementation-phases.md` - Step-by-step implementation guidance

**Use Cases**:
- Onboarding new team members
- Understanding ML/AI algorithms
- Learning federated learning concepts
- Understanding how facilities with different equipment can collaborate
- Troubleshooting implementation issues

**Key Topics Covered**:
- LSTM architecture and gates (forget, input, output)
- Temporal pattern detection in ICS sensor data
- Data normalization strategies for heterogeneous facilities
- Feature padding and masking for missing sensors
- Attack detection examples with real scenarios

---

### 📁 `search/`

Research findings, detailed explanations, and specific technical investigations.

**Purpose**: In-depth analysis of specific components and methodologies

**Key Files**:
- `ai-models-detailed.md` - Detailed explanation of AI models (LSTM, Isolation Forest, GNN)
- `detection-methods-explained.md` - Comprehensive guide to detection methodologies
- `federated-learning-normalization.md` - Data normalization strategies for federated learning
- `modbus-simulator-role.md` - Role and implementation of Modbus simulators

**Use Cases**:
- Deep technical understanding of specific components
- Research and development reference
- Troubleshooting detection algorithms
- Understanding protocol simulation

---

## Technology Stack

### Backend
- **Python 3.11** - Core backend language
- **FastAPI** - Modern async web framework
- **Apache Kafka** - Message broker for event streaming
- **Flower (flwr)** - Federated learning framework

### Machine Learning
- **PyTorch 2.1** - GNN implementation (PyTorch Geometric)
- **TensorFlow 2.14** - LSTM Autoencoder (Keras API)
- **Opacus** - Differential privacy for PyTorch
- **scikit-learn** - Isolation Forest implementation

### Databases
- **Apache IoTDB** - Time-series database for sensor data
- **PostgreSQL 15** - Relational database for alerts and metadata
- **MongoDB 7.0** - Document store for logs and protocol messages
- **Neo4j 5.x** - Graph database for MITRE ATT&CK relationships

### Frontend
- **React 18** - UI framework
- **TypeScript** - Type-safe JavaScript
- **Material-UI (MUI)** - Component library
- **Recharts + D3.js** - Data visualization

### Infrastructure
- **Docker** - Containerization
- **Docker Compose** - Multi-container orchestration

---

## Key Concepts

### Federated Learning
Enables multiple facilities to collaboratively train ML models without sharing raw data. Each facility trains locally and only shares model weight updates with privacy protection.

### Differential Privacy
Mathematical guarantee that individual data points cannot be identified from model outputs. Implemented with ε=2.0 and δ=10⁻⁵.

### MITRE ATT&CK for ICS
Framework of 81 techniques across 12 tactics specific to industrial control systems. Used for attack prediction and correlation.

### Physics-Aware Detection
Validates sensor readings against physical process constraints (temperature ranges, pressure limits, flow dependencies) to detect impossible or dangerous states.

---

## Project Timeline

**Target Demo Date**: November 30, 2025

**Development Phases**:
- Week 1: Foundation setup (infrastructure, data pipeline)
- Week 2: Core detection (LSTM, Isolation Forest, Physics Rules)
- Week 3: Federated learning foundation
- Week 4: Attack prediction and MITRE ATT&CK integration
- Week 5: API and dashboard development
- Week 6: Integration, demo scenarios, and polish

See `federated_learning/system design.md` for detailed weekly milestones and `federated_learning/minimal-design.md` for MVP-focused implementation.

---

## Contributing

This is a research and development project. For questions or contributions, please refer to the documentation in the respective folders.

---

## License

[Specify your license here]

---

## Contact

[Add contact information here]

---

## Additional Resources

### External Links
- MITRE ATT&CK for ICS: https://attack.mitre.org/matrices/ics/
- Flower Framework: https://flower.dev/
- Apache IoTDB: https://iotdb.apache.org/
- SWaT Dataset: https://itrust.sutd.edu.sg/

### Research Papers
- Refer to `federated_learning/training-datasets.md` for dataset papers
- Refer to `architecture/` folder for architectural research

---

**Last Updated**: October 2025  
**Project Status**: In Development  
**Version**: 1.0
