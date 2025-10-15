# What MUST Be in the Demo

## Overview

This document defines the absolute essential features that must be demonstrated by November 30, 2025 to prove the unique value proposition of the Federated ICS Threat Correlation Engine.

---

## MUST HAVE - Core Demo Requirements

### 1. Federated Learning (THE KILLER FEATURE) 🏆

**Why it's critical:** This is your ONLY unique differentiator vs competitors

**Must demonstrate:**
- ✅ 3 simulated facilities (Facility A, B, C)
- ✅ FL Server coordinating rounds
- ✅ Visual proof that:
  - Facility A gets attacked
  - FL round triggered
  - Model updated and distributed
  - Facilities B & C now detect the same attack
- ✅ Show timing: "6-24 hours" vs "weeks/months" for traditional sharing

**Demo scenario:**
```
1. Show Facility A detecting novel attack
2. Trigger FL round (show it happening)
3. Show Facilities B & C receiving updated model
4. Replay same attack on Facility B → Now detected!
5. Show dashboard: "All facilities protected in 6 hours"
```

**Why essential:** Without this, you're just another ICS security tool.

---

### 2. Real-Time Detection (PROOF IT WORKS) ✅

**Why it's critical:** Need to show the system actually detects threats with multiple methods

**Must demonstrate:**
- ✅ THREE working detection methods running in parallel:
  - **LSTM Autoencoder** - Behavioral anomalies (temporal patterns)
  - **Isolation Forest** - Point anomalies (sudden outliers)
  - **Physics Rules Engine** - Process violations (safety constraints)
- ✅ Real-time alerts appearing on dashboard (<30 seconds)
- ✅ Clear visualization of anomaly detection
- ✅ Correlation showing multiple sources agree

**Why all three methods:**
- **LSTM:** Detects subtle behavioral changes over time
- **Isolation Forest:** Catches sudden spikes and outliers LSTM might miss
- **Physics Rules:** Validates against known safety constraints
- **Together:** Reduces false positives, increases confidence

**Demo scenario:**
```
1. Show normal operation (green, all good)
2. Inject Modbus attack with sudden value spike
3. Isolation Forest detects outlier immediately (<5 seconds)
4. LSTM detects behavioral anomaly (within 30 seconds)
5. Physics rules violated (temperature out of range)
6. Correlation engine: "Detected by all 3 sources - CRITICAL"
7. Alert appears on dashboard with high confidence (0.95)
```

**Why essential:** 
- Proves robust multi-layered detection
- Shows defense-in-depth approach
- Demonstrates low false positive rate (all 3 agree = real threat)

---

### 3. Dashboard Visualization (SHOW, DON'T TELL) 📊

**Why it's critical:** Visual proof is more convincing than logs

**Must have pages:**
- ✅ **Alerts Dashboard** - Real-time alert feed
- ✅ **FL Status Page** - Show rounds, clients, model updates
- ✅ **System Overview** - 3 facilities, their status

**Minimum viable:**
- Simple, clean interface (doesn't need to be fancy)
- Real-time updates (WebSocket)
- Clear status indicators (green/red)

**Demo scenario:**
```
1. Show dashboard with 3 facilities
2. Facility A: Alert appears (red)
3. FL Status: Round in progress
4. Facilities B & C: Models updating
5. All facilities: Now protected (green)
```

**Why essential:** Makes the abstract concept concrete and visible.

---

### 4. Privacy Preservation (TRUST FACTOR) 🔒

**Why it's critical:** Addresses the "why not just share data?" question

**Must demonstrate:**
- ✅ Show that raw data stays local
- ✅ Only model weights transmitted (~10 MB)
- ✅ Differential privacy applied (ε=2.0, δ=10⁻⁵)

**Demo scenario:**
```
1. Show Facility A's sensitive data (temperature, pressure)
2. Show FL round: "Transmitting weights only (10 MB)"
3. Show privacy metrics: "ε=2.0 guarantee"
4. Emphasize: "Raw data never left Facility A"
```

**Why essential:** This is what makes FL acceptable for critical infrastructure.

---

### 5. Attack Prediction (WOW FACTOR) 🔮

**Why it's critical:** Proactive defense - predicts attacker's next move

**Must demonstrate:**
- ✅ Graph Neural Network (GAT) predicting next attack steps
- ✅ MITRE ATT&CK for ICS technique mapping
- ✅ Probability scores for next likely techniques
- ✅ Visual graph showing attack progression

**Demo scenario:**
```
1. Show current attack: T0846 (Remote System Discovery)
2. GNN predicts next steps:
   - T0843 (Program Download) - 67% probability
   - T0800 (Lateral Movement) - 23% probability
   - T0858 (Change Operating Mode) - 10% probability
3. Show target asset: PLC-REACTOR-01
4. Show timeframe: "15-60 minutes"
5. Attacker proceeds with T0843 → System predicted it!
6. Show proactive defense: "Blocked based on prediction"
```

**Why essential:** Shifts from reactive to proactive defense - unique competitive advantage.

**Federated Learning Enhancement:**
- GNN learns attack patterns from all facilities
- When Facility A sees new attack chain, all facilities learn it
- Improves prediction accuracy across the network

---

## NICE TO HAVE - Enhances Demo

---

### 6. Protocol Anomaly Detection

**Why it's nice:** Additional layer but not essential with 3 methods already

**If you have time:**
- Deep packet inspection for protocol violations
- Malformed message detection
- Unusual command sequences

**Skip for MVP:** Focus on LSTM, Isolation Forest, and Physics Rules first

---

### 7. Multiple Protocols

**Why it's nice:** Shows versatility but not essential

**Minimum:** Modbus only

**Nice to have:** + DNP3

**Skip for MVP:** OPC-UA, S7

---

## CAN SKIP - Not Essential for Demo

### 8. Advanced Forensics ❌
- Time-travel investigation nice but not essential
- Can be basic query interface for MVP

### 9. Red Team Simulator ❌
- Automated testing not essential
- Manual attack injection sufficient

### 10. MongoDB ❌
- Can use PostgreSQL for logs
- Reduces complexity (but keep if time allows)

### 11. Multiple Protocols Beyond Modbus ❌
- DNP3, OPC-UA, S7 can wait
- Modbus sufficient for demo

---

## Absolute Minimum Demo (If Time is Tight)

### The 3-Minute Demo

**Script:**

**1. SHOW: 3 facilities running, all green**
- "Three power plants, each with their own ICS"

**2. ATTACK: Facility A gets hit**
- "Novel Modbus attack on Facility A"
- → Alert appears, LSTM detects anomaly

**3. FEDERATED LEARNING: Round triggered**
- "FL round initiated - facilities learning together"
- → Show FL status: training, aggregating, distributing

**4. PROTECTION: Facilities B & C now protected**
- "6 hours later, all facilities protected"
- → Replay attack on Facility B → Immediately detected!

**5. PRIVACY: Emphasize data sovereignty**
- "Facility A's data never left their premises"
- → Show: Only 10 MB weights transmitted

**6. COMPARISON: Traditional vs Federated**
- "Traditional: Weeks to share threat intelligence"
- "Federated: 6 hours, automatic, privacy-preserving"

**This proves:**
- ✅ Detection works
- ✅ FL works
- ✅ Privacy preserved
- ✅ Faster than competitors
- ✅ Collaborative defense

---

## Implementation Priority Order

### Week 1-2: Foundation + Detection
1. Docker Compose setup
2. Kafka + PostgreSQL + IoTDB
3. Modbus simulator
4. LSTM Autoencoder working
5. Isolation Forest working
6. Physics Rules working
7. Correlation Engine
8. Basic alerts

### Week 3: Federated Learning (CRITICAL)
7. FL Server (Flower)
8. 3 FL Clients
9. Differential Privacy (Opacus)
10. Model distribution working
11. FL round completes successfully

### Week 4: Attack Prediction & Graph
12. Neo4j MITRE ATT&CK graph
13. Graph Neural Network (GAT) implementation
14. Attack prediction service
15. Technique mapping and API endpoints

### Week 5: Dashboard
16. Alerts page
17. FL status page
18. Attack graph visualization (D3.js)
19. System overview
20. Real-time updates

### Week 6: Integration & Demo
21. Connect all components end-to-end
22. 3 demo scenarios (detection, FL, prediction)
23. Video walkthrough
24. Documentation
25. Polish and practice

---

## Success Criteria for Demo

### Minimum Success
- ✅ FL round completes with 3 facilities
- ✅ Detection works (at least 2 of 3 methods)
- ✅ Dashboard shows it visually
- ✅ Can demonstrate collaborative defense

### Good Success
- ✅ Above + all 3 detection methods working (LSTM, IF, Physics)
- ✅ Above + correlation engine combining signals
- ✅ Above + attack prediction (GNN)
- ✅ Above + polished dashboard
- ✅ Above + clear privacy demonstration

### Excellent Success
- ✅ Above + Isolation Forest
- ✅ Above + multiple protocols
- ✅ Above + forensics
- ✅ Above + advanced correlation

---

## The One Thing That MUST Work

**If you can only get ONE thing working perfectly:**

### Federated Learning demonstration showing:
1. Facility A detects attack
2. FL round triggered
3. All facilities learn
4. Facility B now protected
5. Data stayed local

**This alone proves your unique value proposition.**

---

## Feature Comparison Matrix

| Feature | Priority | Complexity | Impact | Include? |
|---------|----------|------------|--------|----------|
| Federated Learning | CRITICAL | High | Very High | ✅ MUST |
| LSTM Detection | CRITICAL | Medium | High | ✅ MUST |
| Isolation Forest | CRITICAL | Low | High | ✅ MUST |
| Physics Rules | CRITICAL | Low | High | ✅ MUST |
| Correlation Engine | CRITICAL | Low | Very High | ✅ MUST |
| Dashboard (Basic) | HIGH | Medium | High | ✅ MUST |
| Privacy Demo | HIGH | Low | High | ✅ MUST |
| Attack Prediction (GNN) | CRITICAL | High | Very High | ✅ MUST |
| Neo4j | HIGH | Medium | High | ✅ MUST |
| MongoDB | LOW | Low | Low | ❌ SKIP |
| Multiple Protocols | LOW | Medium | Low | ❌ SKIP |
| Forensics | LOW | Medium | Low | ❌ SKIP |

---

## Risk Mitigation

### If Federated Learning is Delayed
**Fallback:** Show simulated FL with manual model updates
**Impact:** Still demonstrates concept, less impressive

### If One Detection Method Fails
**Fallback:** Demonstrate with 2 out of 3 methods (any combination works)
**Impact:** Still shows multi-layered approach, slightly less impressive
**Note:** Having all 3 working is ideal but not critical - 2 methods sufficient

### If Dashboard is Behind
**Fallback:** Use command-line demo with logs
**Impact:** Less visual but proves functionality

### If Integration Fails
**Fallback:** Demonstrate components separately
**Impact:** Less impressive but shows individual capabilities

---

## Demo Scenarios

### Scenario 1: Multi-Layered Detection + FL

**Setup:**
- 3 facilities running normally
- All showing green status
- All 3 detection methods active

**Execution:**
1. **Inject Modbus write attack on Facility A** (sudden value spike)
2. **Isolation Forest detects outlier** (< 5 seconds)
   - "Anomaly score: 0.85 - Sudden spike detected"
3. **LSTM detects behavioral anomaly** (< 30 seconds)
   - "Reconstruction error: 0.92 - Pattern deviation detected"
4. **Physics rules violated** (temperature out of safe range)
   - "CRITICAL: Temperature 380°C exceeds max 350°C"
5. **Correlation Engine combines all three**
   - "HIGH CONFIDENCE ALERT: Detected by 3/3 sources"
   - Confidence: 0.95, Severity: CRITICAL
6. **Alert displayed on dashboard** with all source details
7. **FL round triggered automatically**
8. Show FL progress: training, aggregating, distributing
9. **Replay same attack on Facility B**
10. All 3 detection methods now catch it immediately
11. Show privacy metrics: only weights transmitted

**Duration:** 4-6 minutes

**Key Message:** 
- "Defense in depth: Multiple detection layers working together"
- "One facility's experience protects all others within hours"
- "Low false positives: All 3 methods agree = real threat"

---

### Scenario 2: Attack Prediction in Action

**Setup:**
- System running normally
- MITRE ATT&CK graph loaded in Neo4j
- GNN model trained on attack sequences

**Execution:**
1. Attacker performs reconnaissance (T0846: Remote System Discovery)
2. System detects and alerts
3. GNN analyzes current state and predicts next steps:
   - T0843 (Program Download) - 67% probability
   - T0800 (Lateral Movement) - 23% probability
   - T0858 (Change Operating Mode) - 10% probability
4. Show attack graph visualization with predicted path highlighted
5. Show target asset: PLC-REACTOR-01
6. Show timeframe: "Next step expected in 15-60 minutes"
7. Proactive defense: Increase monitoring on PLC-REACTOR-01
8. Attacker attempts T0843 (Program Download)
9. System: "Predicted attack confirmed! Blocking..."
10. Show how prediction enabled proactive defense

**Duration:** 3-5 minutes

**Key Message:** "Don't just react to attacks - predict and prevent them"

**Federated Learning Enhancement:**
- Show that GNN learned this attack pattern from Facility A
- Now all facilities can predict this attack chain
- Demonstrate collaborative intelligence improving predictions

---

### Scenario 3: Multi-Facility Collaborative Defense

**Setup:**
- 3 facilities with different baseline patterns
- Show that each has unique operational characteristics

**Execution:**
1. Show Facility A's normal operation
2. Introduce subtle anomaly (slow drift)
3. LSTM trained only on Facility A data misses it
4. Trigger FL round with data from all 3 facilities
5. Updated model now detects the subtle anomaly
6. Demonstrate improved accuracy from collaborative learning

**Duration:** 3-5 minutes

**Key Message:** "Collective intelligence improves detection for everyone"

---

## Updated 3-Minute Demo (With Attack Prediction)

### Enhanced Demo Script

**1. SHOW: 3 facilities running, all green** (20 seconds)
- "Three power plants, each with their own ICS"

**2. ATTACK: Facility A gets hit** (30 seconds)
- "Novel multi-stage attack on Facility A"
- → Alert appears, LSTM detects anomaly
- → GNN predicts: "Next step: Program Download (67% probability)"

**3. PREDICTION VALIDATED** (20 seconds)
- Attacker proceeds with predicted technique
- "System predicted this! Proactive defense activated"

**4. FEDERATED LEARNING: Round triggered** (40 seconds)
- "FL round initiated - all facilities learning attack pattern"
- → Show FL status: training, aggregating, distributing
- → GNN model updated with new attack chain

**5. PROTECTION: Facilities B & C now protected** (40 seconds)
- "6 hours later, all facilities protected AND can predict"
- → Replay attack on Facility B
- → Immediately detected AND next step predicted!

**6. PRIVACY & COMPARISON** (30 seconds)
- "Raw data never left premises, only model weights"
- "Traditional: Weeks to share + reactive only"
- "Federated: 6 hours + proactive prediction"

**This proves:**
- ✅ Detection works
- ✅ Prediction works (proactive defense)
- ✅ FL works
- ✅ Privacy preserved
- ✅ Faster than competitors
- ✅ Collaborative defense with intelligence

---

## Documentation Requirements

### Must Have Documentation
1. **README.md** - Quick start guide
2. **DEPLOYMENT.md** - Step-by-step deployment
3. **DEMO-GUIDE.md** - How to run demo scenarios
4. **ARCHITECTURE.md** - System overview
5. **API-DOCS** - Auto-generated from FastAPI

### Nice to Have Documentation
6. User manual
7. Troubleshooting guide
8. Development guide
9. Testing guide

---

## Video Walkthrough Requirements

### Must Include (5-10 minutes)
1. **Introduction** (30 seconds)
   - Problem statement
   - Solution overview

2. **Architecture** (1 minute)
   - Show 6-layer diagram
   - Explain FL concept

3. **Live Demo** (3-5 minutes)
   - Run Scenario 1
   - Show all key features

4. **Privacy Explanation** (1 minute)
   - Differential privacy
   - Data sovereignty

5. **Results & Impact** (1 minute)
   - Performance metrics
   - Comparison with competitors

6. **Conclusion** (30 seconds)
   - Key takeaways
   - Future work

---

## Presentation Slides Requirements

### Must Have Slides
1. Title slide
2. Problem statement
3. Solution overview
4. Architecture diagram
5. Federated learning explanation
6. Live demo (or video)
7. Privacy guarantees
8. Results and metrics
9. Competitive advantages
10. Conclusion

### Nice to Have Slides
11. Technology stack
12. Implementation details
13. Team and timeline
14. Future roadmap
15. Q&A

---

## Final Checklist

### Before Demo Day

**Technical:**
- [ ] All 3 FL clients connect to server
- [ ] FL round completes successfully
- [ ] LSTM detects anomalies
- [ ] Physics rules trigger alerts
- [ ] Dashboard updates in real-time
- [ ] Demo scenarios run reliably (test 10x)
- [ ] System starts with one command
- [ ] Backup plan if live demo fails

**Documentation:**
- [ ] README complete
- [ ] Deployment guide tested
- [ ] Demo guide written
- [ ] Video recorded and edited
- [ ] Presentation slides finalized

**Demo Preparation:**
- [ ] Demo script written
- [ ] Timing practiced (< 10 minutes)
- [ ] Backup video ready
- [ ] Questions anticipated
- [ ] Team roles assigned

---

## Key Messages to Emphasize

1. **"First federated learning for ICS"** - Novel approach
2. **"6-24 hours vs weeks/months"** - Speed advantage
3. **"Privacy-preserving by design"** - Mathematical guarantees
4. **"Collaborative defense"** - Industry-wide protection
5. **"Data sovereignty maintained"** - Regulatory compliance

---

**Document Version:** 1.0  
**Last Updated:** October 14, 2025  
**Status:** Approved for Implementation
