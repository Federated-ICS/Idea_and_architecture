# Implementation Plan - November 30 Demo

## Overview

This implementation plan breaks down the November 30 deliverable into discrete, actionable coding tasks. Each task builds incrementally on previous work and references specific requirements from the requirements document.

**Timeline:** 6 weeks (October 14 - November 30, 2025)
**Focus:** 5 MUST HAVE features - FL, Detection, Dashboard, Privacy, Attack Prediction

---

## Phase 1: Foundation & Infrastructure (Week 1-2)

### 1. Project Setup and Infrastructure

- [ ] 1.1 Initialize project structure and repository
  - Create root directory structure: backend/, frontend/, docker/, docs/, scripts/
  - Initialize Git repository with .gitignore for Python, Node, and Docker
  - Create requirements.txt and package.json files
  - _Requirements: 1.5, 7.1_

- [ ] 1.2 Create Docker Compose configuration
  - Define services: Kafka, Zookeeper, PostgreSQL, IoTDB, Neo4j
  - Configure network and volume mappings
  - Set environment variables for all services
  - Add health checks for service readiness
  - _Requirements: 1.5, 3.7, 7.4_

- [ ] 1.3 Implement database initialization scripts
  - Create PostgreSQL schema (alerts, fl_rounds, predictions tables)
  - Create IoTDB initialization script (3 facility databases with timeseries)
  - Create Neo4j initialization script for MITRE ATT&CK graph structure
  - Write database migration scripts
  - _Requirements: 1.3, 2.3, 7.4_

- [ ] 1.4 Set up Kafka topics and configuration
  - Create topics: sensor_data, alerts, fl_events, predictions
  - Configure retention policies and partitioning
  - Write Kafka producer/consumer utility classes
  - _Requirements: 1.6, 3.1_

- [ ] 1.5 Create backend API project structure
  - Initialize FastAPI application with project layout
  - Set up configuration management (environment variables)
  - Implement database connection utilities (PostgreSQL, IoTDB, Neo4j)
  - Create logging configuration
  - _Requirements: 1.7, 7.2_


### 2. Data Simulator Implementation

- [ ] 2.1 Implement Modbus simulator core
  - Create Modbus TCP server simulation for 3 facilities
  - Implement sensor data generation (temperature, pressure, flow_rate, valve_position)
  - Generate normal operation patterns (sine waves, steady states with noise)
  - Write dual-write logic to Kafka and IoTDB simultaneously
  - _Requirements: 3.1, 3.7_

- [ ] 2.2 Implement attack scenario injection system
  - Create attack scenario manager with trigger API
  - Implement "Sudden Spike" attack (immediate value jump for Isolation Forest)
  - Implement "Gradual Drift" attack (slow increase over 60s for LSTM)
  - Implement "Multi-Stage" attack (discovery → lateral movement → program download)
  - Add MITRE ATT&CK technique tagging to attack events
  - _Requirements: 3.1, 3.2, 3.3, 3.4, 5.2_

- [ ] 2.3 Create simulator control API
  - Implement REST endpoints to start/stop simulation
  - Add endpoints to trigger specific attack scenarios
  - Create endpoint to adjust simulation parameters (frequency, noise level)
  - Write unit tests for simulator logic
  - _Requirements: 6.2, 6.6_

---

## Phase 2: Detection Services (Week 2-3)

### 3. LSTM Autoencoder Implementation

- [ ] 3.1 Implement LSTM Autoencoder model architecture
  - Create PyTorch model: Encoder (Input(60,10) → LSTM(128) → LSTM(64))
  - Create PyTorch model: Decoder (LSTM(64) → LSTM(128) → Output(60,10))
  - Implement training loop with MSE loss
  - Add model save/load functionality
  - _Requirements: 3.2, 8.1_

- [ ] 3.2 Create LSTM training pipeline
  - Implement data loader for IoTDB time-series data (60-second windows)
  - Create training script with normal operation data
  - Calculate and store 95th percentile threshold from training errors
  - Write model evaluation and validation logic
  - _Requirements: 3.2, 8.2_

- [ ] 3.3 Implement LSTM detection service
  - Create detection service that queries IoTDB for 60-second windows
  - Implement real-time inference with reconstruction error calculation
  - Compare reconstruction error against threshold
  - Generate alerts when threshold exceeded
  - Publish alerts to Kafka and store in PostgreSQL
  - _Requirements: 3.2, 3.6, 8.1_

- [ ] 3.4 Write unit tests for LSTM detection
  - Test model inference with known anomalies
  - Test threshold detection logic
  - Test alert generation and storage
  - Verify latency < 30 seconds
  - _Requirements: 8.1, 8.2_


### 4. Isolation Forest Implementation

- [ ] 4.1 Implement Isolation Forest detector
  - Create sklearn IsolationForest model (100 estimators, contamination=0.01)
  - Implement training on normal operation data from IoTDB
  - Add model persistence (save/load)
  - _Requirements: 3.3, 8.2_

- [ ] 4.2 Create Isolation Forest detection service
  - Implement service that queries IoTDB for latest sensor values
  - Run inference and calculate anomaly scores
  - Generate alerts when score > 0.6 threshold
  - Publish alerts to Kafka and store in PostgreSQL
  - _Requirements: 3.3, 3.6, 8.1_

- [ ] 4.3 Write unit tests for Isolation Forest
  - Test detection with sudden spike scenarios
  - Verify latency < 5 seconds
  - Test alert generation
  - Validate false positive rate < 10%
  - _Requirements: 8.1, 8.3_

### 5. Physics Rules Engine Implementation

- [ ] 5.1 Implement physics rules engine core
  - Create rule definition system (range checks, rate-of-change, dependencies)
  - Implement rule evaluation engine
  - Add configurable rules per facility (temperature, pressure limits)
  - Create severity classification (CRITICAL, WARNING, INFO)
  - _Requirements: 3.4, 8.1_

- [ ] 5.2 Create physics rules detection service
  - Implement service that subscribes to Kafka sensor data stream
  - Evaluate rules in real-time on incoming data
  - Generate alerts for rule violations
  - Publish alerts to Kafka and store in PostgreSQL
  - _Requirements: 3.4, 3.6, 8.1_

- [ ] 5.3 Write unit tests for physics rules
  - Test range violation detection
  - Test rate-of-change detection
  - Verify latency < 1 second
  - Test configurable rule loading
  - _Requirements: 8.1_

### 6. Correlation Engine Implementation

- [ ] 6.1 Implement correlation engine logic
  - Create alert aggregation by asset and time window (5 minutes)
  - Implement confidence calculation based on source agreement
  - Implement severity escalation (3 sources=CRITICAL, 2=HIGH, 1=original)
  - Add deduplication logic for similar alerts
  - _Requirements: 3.5, 8.2_

- [ ] 6.2 Create correlation service
  - Implement service that subscribes to all detection alerts from Kafka
  - Apply correlation logic to group and enhance alerts
  - Store correlated alerts in PostgreSQL with source tracking
  - Publish correlated alerts to WebSocket for dashboard
  - _Requirements: 3.5, 3.6_

- [ ] 6.3 Write unit tests for correlation engine
  - Test multi-source alert correlation
  - Test confidence calculation accuracy
  - Test severity escalation logic
  - Verify latency < 5 seconds
  - _Requirements: 8.1_


---

## Phase 3: Federated Learning System (Week 3)

### 7. FL Server Implementation

- [ ] 7.1 Implement Flower FL server core
  - Create Flower server with FedMedian aggregation strategy
  - Implement model versioning and tracking
  - Add minimum 3 clients requirement validation
  - Configure server to run on port 8080
  - _Requirements: 4.1, 4.5_

- [ ] 7.2 Create FL round management system
  - Implement FL round lifecycle (initiate, distribute, collect, aggregate)
  - Create round tracking in PostgreSQL (round_id, status, metrics)
  - Add manual trigger API endpoint for demo
  - Implement round history and metrics storage
  - _Requirements: 4.1, 4.5, 4.6_

- [ ] 7.3 Implement model distribution and aggregation
  - Create model serialization and distribution logic
  - Implement FedMedian aggregation (coordinate-wise median)
  - Add Byzantine-robust outlier detection
  - Store aggregated model and distribute to clients
  - _Requirements: 4.4, 4.5_

- [ ] 7.4 Write unit tests for FL server
  - Test round initiation and completion
  - Test aggregation with 3 clients
  - Test model versioning
  - Verify round duration < 6 minutes
  - _Requirements: 4.5, 8.4_

### 8. FL Client Implementation

- [ ] 8.1 Implement Flower FL client core
  - Create Flower client that connects to FL server
  - Implement model receive and load functionality
  - Add local training loop (5 epochs on IoTDB data)
  - Configure client for each facility (facility_a, facility_b, facility_c)
  - _Requirements: 4.2, 4.6_

- [ ] 8.2 Integrate Opacus for differential privacy
  - Add Opacus PrivacyEngine to training loop
  - Configure noise multiplier (1.1 for ε=2.0)
  - Implement gradient clipping (L2 norm, max=1.0)
  - Add privacy accounting (ε=2.0, δ=10⁻⁵)
  - _Requirements: 4.3, 4.8_

- [ ] 8.3 Implement weight update transmission
  - Create weight extraction and serialization
  - Apply differential privacy noise to weight updates
  - Send weight updates to FL server
  - Track privacy budget consumption
  - _Requirements: 4.3, 4.4_

- [ ] 8.4 Write unit tests for FL client
  - Test local training on facility data
  - Test differential privacy application
  - Test weight update transmission
  - Verify privacy guarantees (ε=2.0, δ=10⁻⁵)
  - _Requirements: 4.3, 4.8, 8.4_

### 9. FL Integration and Testing

- [ ] 9.1 Implement end-to-end FL round
  - Connect 3 FL clients to FL server
  - Test complete FL round (distribute → train → aggregate → distribute)
  - Verify all clients receive updated model
  - Measure and optimize round duration
  - _Requirements: 4.5, 4.6, 4.7_

- [ ] 9.2 Create FL monitoring and metrics
  - Implement real-time FL status tracking
  - Add client connection status monitoring
  - Track training metrics (loss, accuracy per round)
  - Store privacy metrics for dashboard display
  - _Requirements: 4.6, 4.8_

- [ ] 9.3 Write integration tests for FL system
  - Test FL round with simulated attack scenario
  - Verify improved detection after FL round
  - Test privacy preservation (data stays local)
  - Validate round completion within 5-6 minutes
  - _Requirements: 4.6, 4.7, 8.4_


---

## Phase 4: Attack Prediction System (Week 4)

### 10. MITRE ATT&CK Graph Setup

- [ ] 10.1 Load MITRE ATT&CK for ICS into Neo4j
  - Download MITRE ATT&CK for ICS dataset
  - Create Neo4j nodes for techniques (T0846, T0800, T0843, T0858, etc.)
  - Create LEADS_TO relationships with probability weights
  - Add technique metadata (name, description, tactics)
  - _Requirements: 5.2, 5.5_

- [ ] 10.2 Create attack sequence training data
  - Generate synthetic attack sequences from MITRE relationships
  - Create training dataset with technique progressions
  - Add temporal information and target asset data
  - Store training sequences in PostgreSQL
  - _Requirements: 5.1, 5.2_

- [ ] 10.3 Implement Neo4j query utilities
  - Create utility functions for technique lookups
  - Implement graph traversal queries for attack paths
  - Add relationship probability queries
  - Write unit tests for graph queries
  - _Requirements: 5.5_

### 11. Graph Neural Network Implementation

- [ ] 11.1 Implement GAT model architecture
  - Create PyTorch Geometric GAT model (Layer 1: GATConv(64, heads=4))
  - Add dropout layer (0.6) and Layer 2: GATConv(32, heads=4)
  - Implement classifier layer (Linear → Softmax)
  - Add model save/load functionality
  - _Requirements: 5.1, 5.3_

- [ ] 11.2 Create GNN training pipeline
  - Implement data loader for attack sequences and graph structure
  - Create training loop with cross-entropy loss
  - Train on attack sequence dataset
  - Implement model evaluation and validation
  - _Requirements: 5.1, 5.3_

- [ ] 11.3 Implement GNN prediction service
  - Create service that receives current technique from alerts
  - Query Neo4j for graph structure around current technique
  - Run GNN inference to predict next techniques
  - Generate top-3 predictions with probabilities
  - _Requirements: 5.1, 5.3_

- [ ] 11.4 Write unit tests for GNN
  - Test prediction accuracy on known attack sequences
  - Verify top-3 predictions include correct next step
  - Test inference latency
  - Validate probability distributions sum to 1.0
  - _Requirements: 5.1, 5.3_

### 12. Prediction Service Integration

- [ ] 12.1 Implement prediction service
  - Create service that subscribes to alerts from Kafka
  - Extract MITRE technique IDs from alerts
  - Trigger GNN prediction for detected techniques
  - Store predictions in PostgreSQL with confidence scores
  - _Requirements: 5.1, 5.4_

- [ ] 12.2 Add prediction enrichment
  - Identify target assets based on current attack context
  - Estimate timeframe for next attack step (15-60 minutes)
  - Add prediction validation tracking (mark when prediction confirmed)
  - _Requirements: 5.4_

- [ ] 12.3 Create prediction API endpoints
  - Implement GET /api/predictions - list predictions
  - Implement GET /api/predictions/{id} - prediction details
  - Implement POST /api/predictions/predict - manual trigger
  - Add WebSocket channel for real-time prediction updates
  - _Requirements: 5.3, 5.6_

- [ ] 12.4 Write integration tests for prediction system
  - Test end-to-end: alert → prediction → storage
  - Verify prediction appears on dashboard
  - Test multi-stage attack scenario with prediction validation
  - _Requirements: 5.1, 5.6, 5.7_


---

## Phase 5: API Gateway and Backend Services (Week 4-5)

### 13. API Gateway Implementation

- [ ] 13.1 Implement core API endpoints
  - Create GET /api/alerts with filtering (severity, facility, status)
  - Create GET /api/alerts/{id} for alert details
  - Create PUT /api/alerts/{id}/status for status updates
  - Add request validation and error handling
  - _Requirements: 3.6, 7.2_

- [ ] 13.2 Implement FL API endpoints
  - Create POST /api/fl/rounds/trigger to start FL round
  - Create GET /api/fl/rounds to list rounds with history
  - Create GET /api/fl/rounds/{id} for round details
  - Create GET /api/fl/clients for connected client status
  - _Requirements: 4.6, 7.2_

- [ ] 13.3 Implement demo control endpoints
  - Create POST /api/demo/scenarios/attack to trigger attack scenarios
  - Create POST /api/demo/scenarios/fl to trigger FL demo
  - Create GET /api/demo/status for demo state
  - Add scenario parameter validation
  - _Requirements: 6.2, 6.6_

- [ ] 13.4 Implement system endpoints
  - Create GET /api/system/health for health checks
  - Create GET /api/system/metrics for performance metrics
  - Add service status monitoring
  - _Requirements: 7.2, 8.5_

- [ ] 13.5 Write API documentation and tests
  - Generate OpenAPI/Swagger documentation from FastAPI
  - Write unit tests for all endpoints
  - Test error handling and validation
  - Verify response times < 100ms (p95)
  - _Requirements: 7.2, 8.5_

### 14. WebSocket Implementation

- [ ] 14.1 Implement WebSocket server
  - Create WebSocket endpoint at /ws
  - Implement connection management and authentication
  - Add channel subscription system (alerts, fl_status, predictions, system)
  - Handle client disconnections gracefully
  - _Requirements: 3.6, 8.6_

- [ ] 14.2 Implement real-time event broadcasting
  - Subscribe to Kafka topics for alerts, FL events, predictions
  - Broadcast events to connected WebSocket clients
  - Add message filtering by subscribed channels
  - Implement rate limiting to prevent flooding
  - _Requirements: 3.6, 8.6_

- [ ] 14.3 Write WebSocket tests
  - Test connection establishment and channel subscription
  - Test real-time alert broadcasting
  - Test FL status updates during rounds
  - Verify latency < 1 second
  - _Requirements: 8.6_


---

## Phase 6: Dashboard Frontend (Week 5)

### 15. Frontend Project Setup

- [ ] 15.1 Initialize React project with TypeScript
  - Create React app with TypeScript template
  - Install dependencies (MUI, Recharts, D3.js, axios, socket.io-client)
  - Configure TypeScript and ESLint
  - Set up project structure (components/, pages/, services/, types/)
  - _Requirements: 2.4, 7.1_

- [ ] 15.2 Create API service layer
  - Implement axios client with base configuration
  - Create API functions for all backend endpoints
  - Add TypeScript interfaces for API responses
  - Implement error handling and retry logic
  - _Requirements: 2.4, 7.2_

- [ ] 15.3 Implement WebSocket service
  - Create WebSocket client connection manager
  - Implement channel subscription system
  - Add reconnection logic with exponential backoff
  - Create React hooks for WebSocket data (useAlerts, useFLStatus, usePredictions)
  - _Requirements: 2.4, 8.6_

### 16. Alerts Dashboard Page

- [ ] 16.1 Create alert feed component
  - Implement real-time alert list with auto-scroll
  - Add alert card component with severity color coding
  - Display alert details (timestamp, facility, sources, confidence)
  - Implement filtering by severity, facility, and status
  - _Requirements: 2.4, 3.6_

- [ ] 16.2 Create facility status cards
  - Implement 3 facility status cards (facility_a, facility_b, facility_c)
  - Add color-coded status indicators (green/yellow/red)
  - Display current alert count per facility
  - Show detection method status (LSTM, IF, Physics)
  - _Requirements: 2.4, 3.6_

- [ ] 16.3 Implement alert details modal
  - Create modal component for full alert context
  - Display all alert metadata and correlation info
  - Show detection sources and confidence breakdown
  - Add acknowledge and resolve action buttons
  - _Requirements: 2.4, 3.6_

- [ ] 16.4 Add real-time updates
  - Connect alert feed to WebSocket alerts channel
  - Implement smooth animations for new alerts
  - Update facility status cards in real-time
  - Add notification sound for critical alerts
  - _Requirements: 2.4, 3.6, 8.6_

### 17. FL Status Page

- [ ] 17.1 Create FL round progress component
  - Implement progress bar showing round completion percentage
  - Display current round phase (distributing, training, aggregating)
  - Show estimated time remaining
  - Add round history timeline
  - _Requirements: 2.4, 4.6_

- [ ] 17.2 Create client status indicators
  - Implement 3 client status cards (one per facility)
  - Show training progress per client
  - Display connection status (connected/disconnected)
  - Show local training metrics (loss, accuracy)
  - _Requirements: 2.4, 4.6_

- [ ] 17.3 Implement privacy metrics display
  - Create privacy metrics panel showing ε, δ values
  - Display data transmitted (weights only, ~10 MB)
  - Show "Raw data stays local" indicator
  - Add privacy guarantee explanation tooltip
  - _Requirements: 2.4, 4.8_

- [ ] 17.4 Add FL control and history
  - Implement "Trigger FL Round" button
  - Create round history table with metrics
  - Display model version tracking
  - Show performance improvement over rounds
  - _Requirements: 2.4, 4.6_

- [ ] 17.5 Connect to WebSocket FL updates
  - Subscribe to fl_status channel
  - Update progress bar in real-time
  - Update client status during training
  - Show completion notification
  - _Requirements: 2.4, 4.6, 8.6_


### 18. Attack Graph Page

- [ ] 18.1 Implement D3.js force-directed graph
  - Create D3.js graph visualization component
  - Implement force simulation for node positioning
  - Add zoom and pan controls
  - Implement node and edge rendering
  - _Requirements: 2.4, 5.6_

- [ ] 18.2 Add attack technique visualization
  - Highlight current technique node (red)
  - Highlight predicted techniques (orange)
  - Display probability labels on edges
  - Show technique IDs and names
  - _Requirements: 2.4, 5.3, 5.6_

- [ ] 18.3 Create technique details panel
  - Implement side panel for technique information
  - Display MITRE ATT&CK technique details on node click
  - Show technique description, tactics, and mitigations
  - Display prediction confidence and timeframe
  - _Requirements: 2.4, 5.3, 5.4_

- [ ] 18.4 Add attack path history
  - Implement attack path visualization showing progression
  - Highlight completed attack steps
  - Show predicted vs actual attack paths
  - Add timeline of attack progression
  - _Requirements: 2.4, 5.6_

- [ ] 18.5 Connect to WebSocket predictions
  - Subscribe to predictions channel
  - Update graph when new predictions arrive
  - Animate node highlighting for new predictions
  - Update technique details panel automatically
  - _Requirements: 2.4, 5.6, 8.6_

### 19. Dashboard Integration and Polish

- [ ] 19.1 Create main navigation and layout
  - Implement navigation bar with page links
  - Create responsive layout with sidebar
  - Add system status indicator in header
  - Implement dark/light theme toggle
  - _Requirements: 2.4, 7.1_

- [ ] 19.2 Add system overview dashboard
  - Create overview page with key metrics
  - Display total alerts, active FL rounds, recent predictions
  - Show system health status
  - Add quick links to detailed pages
  - _Requirements: 2.4, 2.6_

- [ ] 19.3 Implement error handling and loading states
  - Add loading spinners for async operations
  - Implement error boundaries for component errors
  - Show user-friendly error messages
  - Add retry mechanisms for failed requests
  - _Requirements: 2.4, 7.2_

- [ ] 19.4 Write frontend tests
  - Write unit tests for components using React Testing Library
  - Test WebSocket connection and updates
  - Test user interactions (filtering, modal opening, button clicks)
  - Verify page load time < 2 seconds
  - _Requirements: 2.4, 8.6_


---

## Phase 7: Integration and Demo Scenarios (Week 6)

### 20. End-to-End Integration

- [ ] 20.1 Integrate all backend services
  - Connect simulator → Kafka → IoTDB pipeline
  - Connect detection services → correlation → alerts
  - Connect alerts → FL trigger → FL round
  - Connect alerts → prediction service → Neo4j
  - _Requirements: 3.1, 3.5, 4.5, 5.1_

- [ ] 20.2 Integrate backend with frontend
  - Connect dashboard to API Gateway
  - Verify WebSocket real-time updates work end-to-end
  - Test all API endpoints from frontend
  - Verify data flow from simulator to dashboard
  - _Requirements: 2.4, 3.6, 8.6_

- [ ] 20.3 Create Docker Compose orchestration
  - Add all application services to docker-compose.yml
  - Configure service dependencies and startup order
  - Add health checks and restart policies
  - Create single-command startup script
  - _Requirements: 1.5, 3.7, 7.4_

- [ ] 20.4 Write integration tests
  - Test complete detection flow (simulator → alert → dashboard)
  - Test complete FL flow (alert → FL round → improved detection)
  - Test complete prediction flow (alert → prediction → dashboard)
  - Verify all services communicate correctly
  - _Requirements: 8.1, 8.4_

### 21. Demo Scenario 1: Multi-Layered Detection + FL

- [ ] 21.1 Implement Scenario 1 orchestration
  - Create demo script for "Multi-Layered Detection + FL" scenario
  - Implement automated scenario execution via API
  - Add timing controls for demo pacing
  - Create scenario reset functionality
  - _Requirements: 6.2, 6.6, 6.7_

- [ ] 21.2 Test Scenario 1 reliability
  - Run scenario 10 times and verify success rate > 95%
  - Measure timing consistency (should complete in 6-7 minutes)
  - Verify all 3 detection methods trigger
  - Verify FL round completes successfully
  - Verify improved detection after FL round
  - _Requirements: 6.2, 6.6, 6.7, 8.1, 8.4_

- [ ] 21.3 Create Scenario 1 documentation
  - Write step-by-step demo guide for Scenario 1
  - Document expected timeline and outcomes
  - Add troubleshooting section for common issues
  - Create visual aids (screenshots, diagrams)
  - _Requirements: 6.6, 7.6_

### 22. Demo Scenario 2: Attack Prediction

- [ ] 22.1 Implement Scenario 2 orchestration
  - Create demo script for "Attack Prediction" scenario
  - Implement multi-stage attack injection (T0846 → T0843)
  - Add timing controls for prediction demonstration
  - Create scenario reset functionality
  - _Requirements: 6.2, 6.6, 6.7_

- [ ] 22.2 Test Scenario 2 reliability
  - Run scenario 10 times and verify success rate > 95%
  - Measure timing consistency (should complete in 2-3 minutes)
  - Verify GNN prediction accuracy
  - Verify prediction validation when attacker proceeds
  - _Requirements: 5.1, 5.7, 6.2, 6.6_

- [ ] 22.3 Create Scenario 2 documentation
  - Write step-by-step demo guide for Scenario 2
  - Document expected predictions and probabilities
  - Add explanation of MITRE ATT&CK techniques used
  - Create visual aids for attack graph
  - _Requirements: 6.6, 7.6_

### 23. Demo Scenario 3: Privacy Demonstration

- [ ] 23.1 Implement privacy metrics tracking
  - Add logging for data transmission during FL rounds
  - Track and display raw data size vs weight size
  - Show privacy budget consumption (ε, δ)
  - Create visual comparison of data sharing approaches
  - _Requirements: 4.8, 6.2_

- [ ] 23.2 Create privacy demonstration script
  - Create demo script emphasizing privacy preservation
  - Show that raw data never leaves facility
  - Demonstrate differential privacy in action
  - Compare with traditional data sharing
  - _Requirements: 4.8, 6.2_

- [ ] 23.3 Test privacy demonstration
  - Verify privacy metrics display correctly
  - Verify ε=2.0, δ=10⁻⁵ guarantees maintained
  - Test demonstration script for clarity
  - _Requirements: 4.8, 6.2_


### 24. Performance Optimization and Testing

- [ ] 24.1 Optimize detection latency
  - Profile LSTM inference and optimize bottlenecks
  - Optimize IoTDB queries for time-series data
  - Add caching where appropriate
  - Verify total detection latency < 30 seconds
  - _Requirements: 8.1_

- [ ] 24.2 Optimize FL round duration
  - Profile FL training and identify bottlenecks
  - Optimize data loading from IoTDB
  - Tune training parameters for speed
  - Verify FL round completes in 5-6 minutes
  - _Requirements: 8.4_

- [ ] 24.3 Optimize dashboard performance
  - Implement virtual scrolling for alert lists
  - Optimize D3.js graph rendering
  - Add debouncing for real-time updates
  - Verify dashboard updates < 1 second latency
  - _Requirements: 8.6_

- [ ] 24.4 Run performance tests
  - Use locust to test API throughput
  - Test Kafka throughput (target: 1000 msg/sec)
  - Test database write performance (target: 500 inserts/sec)
  - Verify system runs on 16GB RAM, 4 CPU cores
  - _Requirements: 8.5, 8.7_

- [ ] 24.5 Measure and document metrics
  - Measure detection accuracy on test scenarios (target: > 90%)
  - Measure false positive rate (target: < 10%)
  - Document all performance metrics
  - Create performance dashboard or report
  - _Requirements: 8.1, 8.2, 8.3_

---

## Phase 8: Documentation and Presentation (Week 6)

### 25. Technical Documentation

- [ ] 25.1 Create README.md
  - Write project overview and value proposition
  - Add quick start guide (< 30 minutes to deploy)
  - Document system requirements
  - Add architecture overview diagram
  - Include demo scenario descriptions
  - _Requirements: 7.1, 7.4_

- [ ] 25.2 Create DEPLOYMENT.md
  - Write detailed deployment instructions
  - Document Docker Compose setup
  - Add database initialization steps
  - Include troubleshooting section
  - Add verification steps
  - _Requirements: 7.4_

- [ ] 25.3 Create DEMO-GUIDE.md
  - Write step-by-step instructions for each demo scenario
  - Include expected outcomes and timings
  - Add screenshots and visual aids
  - Document demo control API endpoints
  - _Requirements: 7.6_

- [ ] 25.4 Generate API documentation
  - Generate OpenAPI/Swagger docs from FastAPI
  - Add endpoint descriptions and examples
  - Document request/response schemas
  - Include authentication and error handling
  - _Requirements: 1.7, 7.2_

- [ ] 25.5 Create architecture documentation
  - Document system architecture with diagrams
  - Explain component interactions and data flows
  - Document technology choices and justifications
  - Add database schemas and API specifications
  - _Requirements: 1.1, 1.3, 1.6, 2.1, 2.2, 2.3_


### 26. Presentation Materials

- [ ] 26.1 Create presentation slides
  - Design slide deck covering problem statement
  - Add solution overview and architecture slides
  - Include demo walkthrough slides
  - Add results and metrics slides
  - Create conclusion and future work slides
  - _Requirements: 7.3_

- [ ] 26.2 Record demo video
  - Record 5-10 minute video walkthrough
  - Show data ingestion and normal operation
  - Demonstrate threat detection (all 3 methods)
  - Show FL round in action
  - Demonstrate attack prediction
  - Show dashboard visualization
  - _Requirements: 6.3, 6.4_

- [ ] 26.3 Edit and polish video
  - Add narration explaining each step
  - Add text overlays for key points
  - Include timing indicators
  - Add intro and outro slides
  - Export in high quality format
  - _Requirements: 6.4_

- [ ] 26.4 Prepare live demo backup
  - Create pre-recorded video as backup
  - Prepare demo script with timing
  - Test demo on fresh system
  - Create contingency plan for technical issues
  - _Requirements: 6.1, 6.2, 6.5_

### 27. Final Testing and Polish

- [ ] 27.1 Conduct end-to-end system test
  - Deploy system on fresh machine
  - Run all demo scenarios
  - Verify all features work as expected
  - Test with different attack variations
  - _Requirements: 6.5, 8.1_

- [ ] 27.2 Fix critical bugs
  - Address any bugs found during testing
  - Fix issues affecting demo reliability
  - Improve error handling and recovery
  - _Requirements: 8.1_

- [ ] 27.3 Polish user experience
  - Improve dashboard aesthetics
  - Add helpful tooltips and explanations
  - Improve error messages
  - Add loading indicators where needed
  - _Requirements: 2.4, 2.7_

- [ ] 27.4 Verify all requirements met
  - Review requirements document and check off all items
  - Verify all acceptance criteria satisfied
  - Document any deferred features
  - Create final checklist
  - _Requirements: All_

- [ ] 27.5 Prepare for demonstration
  - Practice demo presentation (< 10 minutes)
  - Prepare answers to anticipated questions
  - Test demo on presentation hardware
  - Create demo day checklist
  - _Requirements: 6.1, 6.2, 6.3_

---

## Success Criteria

### Minimum Success (Must Achieve)
- ✅ FL round completes with 3 clients
- ✅ At least 2 detection methods working
- ✅ Dashboard shows alerts and FL status
- ✅ Can demonstrate collaborative defense

### Target Success (Goal)
- ✅ All 3 detection methods working (LSTM, IF, Physics)
- ✅ Correlation engine combining signals
- ✅ Attack prediction (GNN) working
- ✅ Polished dashboard with real-time updates
- ✅ Privacy demonstration clear

### Stretch Success (Bonus)
- ✅ Multiple attack scenarios working reliably
- ✅ Performance metrics dashboard
- ✅ Video walkthrough polished and professional
- ✅ System deployable in < 30 minutes

---

## Notes

- Each task should be completed and tested before moving to the next
- Integration testing should happen continuously, not just at the end
- Demo scenarios should be tested frequently to ensure reliability
- Documentation should be updated as features are implemented
- Focus on the 5 MUST HAVE features: FL, Detection, Dashboard, Privacy, Attack Prediction
- Keep implementation minimal and demo-focused

---

**Total Tasks:** 27 major tasks with 100+ sub-tasks
**Timeline:** 6 weeks (October 14 - November 30, 2025)
**Estimated Effort:** 240-300 hours
**Team Size:** 1-2 developers

---

**Document Version:** 1.0  
**Last Updated:** October 14, 2025  
**Status:** Ready for Implementation

