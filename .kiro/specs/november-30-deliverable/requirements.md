# Requirements Document

## Introduction

This document outlines the requirements for delivering a demonstrable prototype of the Federated ICS Threat Correlation Engine by November 30, 2025. The deliverable must include technical documentation, system design artifacts, and a working prototype that showcases the core capabilities of the system.

**Timeline:** October 13 - November 30, 2025 (~6 weeks)

**Target Audience:** Technical evaluators, stakeholders, competition judges

**Scope:** MVP prototype with core features demonstrable through live demo or video walkthrough

---

## Requirements

### Requirement 1: Technical Concept Documentation

**User Story:** As a technical evaluator, I want comprehensive documentation of the tools, technologies, and development approach, so that I can assess the technical feasibility and architecture decisions.

#### Acceptance Criteria

1. WHEN the technical concept document is reviewed THEN it SHALL include a complete list of all technologies with version numbers and justification for selection
2. WHEN the technical concept document is reviewed THEN it SHALL include a development roadmap with weekly milestones from October 13 to November 30
3. WHEN the technical concept document is reviewed THEN it SHALL include technology stack diagrams showing how components interact
4. IF a technology choice has alternatives THEN the document SHALL explain why the selected technology was chosen
5. WHEN the technical concept is reviewed THEN it SHALL include deployment architecture (Docker Compose setup)
6. WHEN the technical concept is reviewed THEN it SHALL include data flow diagrams showing how data moves through the system
7. WHEN the technical concept is reviewed THEN it SHALL include API specifications for key interfaces

---

### Requirement 2: System Design and Mockups

**User Story:** As a stakeholder, I want visual representations of the system architecture and user interfaces, so that I can understand how the system works and what users will experience.

#### Acceptance Criteria

1. WHEN the system design is reviewed THEN it SHALL include a high-level architecture diagram with all major components
2. WHEN the system design is reviewed THEN it SHALL include detailed component diagrams for each subsystem (data ingestion, detection, federated learning, storage, API)
3. WHEN the system design is reviewed THEN it SHALL include database schemas for all storage systems (IoTDB, PostgreSQL, MongoDB, Neo4j)
4. WHEN the system design is reviewed THEN it SHALL include UI mockups for the dashboard showing alerts, system status, and attack visualization
5. WHEN the system design is reviewed THEN it SHALL include sequence diagrams for critical workflows (alert generation, FL round, attack prediction)
6. WHEN the system design is reviewed THEN it SHALL include network topology diagrams showing deployment architecture
7. WHEN mockups are reviewed THEN they SHALL be interactive or high-fidelity (Figma, Adobe XD, or similar)
8. WHEN the federated learning workflow is reviewed THEN it SHALL include visual diagrams showing client-server interaction

---

### Requirement 3: Working Prototype - Core Detection

**User Story:** As a demo viewer, I want to see the system detect threats in real-time from industrial protocols, so that I can verify the core detection capabilities work.

#### Acceptance Criteria

1. WHEN industrial protocol data (Modbus) is ingested THEN the system SHALL parse and store the data in appropriate databases
2. WHEN sensor data exhibits anomalous behavior THEN the LSTM Autoencoder SHALL detect the anomaly within 30 seconds
3. WHEN an outlier occurs in the data THEN the Isolation Forest SHALL flag it as suspicious
4. WHEN process physics constraints are violated THEN the Physics Rules Engine SHALL generate an alert
5. WHEN multiple detection sources flag the same asset THEN the Correlation Engine SHALL create a unified high-severity alert
6. WHEN alerts are generated THEN they SHALL be visible in the web dashboard in real-time
7. WHEN the system is deployed THEN all components SHALL run via Docker Compose with a single command

---

### Requirement 4: Working Prototype - Federated Learning

**User Story:** As a technical evaluator, I want to see federated learning in action across simulated facilities, so that I can verify the privacy-preserving collaborative defense capability.

#### Acceptance Criteria

1. WHEN the FL server initiates a round THEN it SHALL distribute the global model to all connected FL clients
2. WHEN an FL client receives a model THEN it SHALL train on local data for 5 epochs
3. WHEN local training completes THEN the FL client SHALL apply differential privacy noise (ε=2.0, δ=10⁻⁵) to weight updates
4. WHEN the FL server receives weight updates from clients THEN it SHALL aggregate them using coordinate-wise median
5. WHEN aggregation completes THEN the FL server SHALL distribute the improved global model to all clients
6. WHEN a simulated attack occurs at one facility THEN other facilities SHALL show improved detection after the next FL round
7. WHEN the FL process is demonstrated THEN it SHALL show at least 3 simulated facilities participating
8. WHEN privacy is evaluated THEN the system SHALL demonstrate that raw data never leaves each facility

---

### Requirement 5: Working Prototype - Attack Prediction

**User Story:** As a security analyst, I want the system to predict the attacker's next move based on current activity, so that I can proactively defend critical assets.

#### Acceptance Criteria

1. WHEN an attack technique is detected THEN the Graph Neural Network SHALL predict the next likely technique with probability scores
2. WHEN predictions are generated THEN they SHALL reference MITRE ATT&CK for ICS technique IDs
3. WHEN predictions are displayed THEN they SHALL show the top 3 most likely next steps with probabilities
4. WHEN predictions are generated THEN they SHALL identify target assets and estimated timeframe
5. WHEN the attack graph is queried THEN it SHALL be stored in Neo4j with MITRE ATT&CK relationships
6. WHEN predictions are shown in the dashboard THEN they SHALL be visualized as a graph showing attack progression

---

### Requirement 6: Demonstration Capability

**User Story:** As a presenter, I want multiple ways to demonstrate the system (live demo, deployed version, video walkthrough), so that I can effectively showcase capabilities regardless of technical constraints.

#### Acceptance Criteria

1. WHEN the system is deployed THEN it SHALL be accessible via a web URL for live demonstration
2. WHEN the live demo is performed THEN it SHALL include pre-loaded scenarios that can be triggered on demand
3. WHEN a video walkthrough is created THEN it SHALL be 5-10 minutes long covering all core features
4. WHEN the video is reviewed THEN it SHALL show: data ingestion, threat detection, attack prediction, federated learning round, and dashboard visualization
5. WHEN deployment instructions are provided THEN a new user SHALL be able to deploy the system in under 30 minutes
6. WHEN demo scenarios are executed THEN they SHALL complete within 2-3 minutes each
7. WHEN the system is demonstrated THEN it SHALL include at least 3 attack scenarios: Modbus manipulation, unauthorized access, and coordinated multi-stage attack

---

### Requirement 7: Documentation and Presentation Materials

**User Story:** As an evaluator, I want clear documentation and presentation materials, so that I can understand the system without requiring live support.

#### Acceptance Criteria

1. WHEN the README is reviewed THEN it SHALL include quick start instructions, architecture overview, and demo scenarios
2. WHEN the technical documentation is reviewed THEN it SHALL include API documentation, configuration guide, and troubleshooting section
3. WHEN presentation slides are reviewed THEN they SHALL cover problem statement, solution overview, technical architecture, demo walkthrough, and results
4. WHEN the deployment guide is followed THEN it SHALL enable deployment on a fresh Ubuntu/macOS system
5. WHEN the architecture documentation is reviewed THEN it SHALL match the implemented system
6. WHEN demo scripts are provided THEN they SHALL include step-by-step instructions for each demonstration scenario

---

### Requirement 8: Performance and Quality Metrics

**User Story:** As a technical evaluator, I want to see measurable performance metrics, so that I can assess whether the system meets its stated goals.

#### Acceptance Criteria

1. WHEN detection latency is measured THEN it SHALL be less than 30 seconds from event to alert
2. WHEN detection accuracy is measured THEN it SHALL achieve at least 90% accuracy on test scenarios
3. WHEN false positive rate is measured THEN it SHALL be less than 10% on normal operational data
4. WHEN FL round duration is measured THEN it SHALL complete within 5 minutes for 3 simulated facilities
5. WHEN system resource usage is measured THEN it SHALL run on a machine with 16GB RAM and 4 CPU cores
6. WHEN the dashboard is tested THEN it SHALL update in real-time with less than 1 second latency
7. WHEN the system is load tested THEN it SHALL handle at least 1000 messages per second through Kafka

---

## Success Criteria Summary

The deliverable will be considered successful when:

✅ **Technical Concept** - Complete documentation of tools, technologies, and milestones
✅ **System Design** - Architecture diagrams, mockups, and technical structure documented
✅ **Prototype** - Working system demonstrating core detection, federated learning, and attack prediction
✅ **Demo Ready** - Live deployment, video walkthrough, and presentation materials prepared
✅ **Performance** - Meets stated metrics for latency, accuracy, and resource usage
✅ **Documentation** - Clear, comprehensive, and enables independent evaluation

---

## Out of Scope (Deferred to Future Phases)

- Production-grade security hardening
- Kubernetes deployment
- Support for all 4 industrial protocols (focus on Modbus + 1 other)
- Advanced forensic capabilities
- Red team simulator with 15+ scenarios (include 3-5 basic scenarios)
- Multi-tenant support
- Real facility deployment
- Advanced Byzantine-robust aggregation (use basic outlier detection)
- Secure aggregation with cryptographic masking (use differential privacy only)

---

## Assumptions

1. Development will use simulated ICS environments (OpenPLC, Modbus simulators)
2. Federated learning will be demonstrated with 3 simulated facilities on the same machine
3. Attack scenarios will be pre-scripted and triggered programmatically
4. Dashboard will be functional but may not be fully polished
5. Focus is on demonstrating technical feasibility, not production readiness
6. Team has access to development machines with adequate resources (16GB+ RAM)
7. Open-source tools and frameworks will be used throughout
