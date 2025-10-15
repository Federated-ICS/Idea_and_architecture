# Graph Neural Networks (GNN) for Relationship Modeling

## What is a Graph Neural Network?

**GNN = Neural network that operates on graph-structured data**

Unlike traditional neural networks that work on grids (images) or sequences (text), GNNs work on graphs - data with nodes and edges representing relationships.

## Graph Basics

### What is a Graph?

```
Nodes (Vertices): Entities
Edges (Links): Relationships between entities

Example:
    A --- B
    |     |
    C --- D

Nodes: A, B, C, D
Edges: A-B, A-C, B-D, C-D
```

### Graph Representation

**Adjacency Matrix:**
```
    A  B  C  D
A [ 0  1  1  0 ]
B [ 1  0  0  1 ]
C [ 1  0  0  1 ]
D [ 0  1  1  0 ]

1 = connected, 0 = not connected
```

**Edge List:**
```
[(A, B), (A, C), (B, D), (C, D)]
```

## Why GNN for Attack Prediction?

### Attack Techniques Form a Graph

MITRE ATT&CK techniques have relationships:
```
T0846 (Discovery) ──→ T0843 (Program Download)
       ↓                      ↓
T0800 (Lateral Move) ──→ T0858 (Change Mode)
       ↓                      ↓
T0836 (Modify Param) ───→ T0831 (Impact)

Edges represent: "Technique A often leads to Technique B"
```

### Traditional ML Limitations

**Regular Neural Network:**
```
Input: [T0846 detected]
Output: [Probability of next technique]

Problem: Doesn't understand relationships
Treats all techniques as independent
```

**Graph Neural Network:**
```
Input: [T0846 detected] + [Graph structure]
Output: [Probability based on graph relationships]

Advantage: Knows T0846 → T0843 is common path
```

## How GNN Works

### Message Passing

Core idea: Nodes exchange information with neighbors

```
Step 1: Each node has features
Node A: [0.5, 0.3, 0.8]

Step 2: Collect messages from neighbors
Messages from B, C: [0.2, 0.4], [0.6, 0.1]

Step 3: Aggregate messages
Aggregated: sum or mean of messages

Step 4: Update node features
New A features: combine old features + aggregated messages

Step 5: Repeat for multiple layers
```

### Visual Example

```
Layer 0 (Initial):
A(0.5) --- B(0.3)
  |          |
C(0.7) --- D(0.2)

Layer 1 (After message passing):
A(0.4) --- B(0.45)  ← Updated based on neighbors
  |          |
C(0.35) -- D(0.5)

Layer 2:
A(0.42) -- B(0.38)  ← Further refined
  |          |
C(0.48) -- D(0.41)
```

## Graph Attention Network (GAT)

### What is Attention?

**Attention = Not all neighbors are equally important**

```
Node A connected to:
- Node B (very relevant)
- Node C (somewhat relevant)
- Node D (not very relevant)

Attention mechanism learns:
- Pay 70% attention to B
- Pay 25% attention to C
- Pay 5% attention to D
```

### How GAT Works

**Step 1: Compute Attention Scores**
```python
for each neighbor:
    score = attention_function(node_features, neighbor_features)
    
Example:
A → B: score = 0.8
A → C: score = 0.5
A → D: score = 0.1
```

**Step 2: Normalize Scores (Softmax)**
```python
attention_weights = softmax([0.8, 0.5, 0.1])
                  = [0.52, 0.32, 0.16]
```

**Step 3: Weighted Aggregation**
```python
message = 0.52 * B_features + 0.32 * C_features + 0.16 * D_features
```

**Step 4: Update Node**
```python
new_A_features = combine(A_features, message)
```

### Multi-Head Attention

**Project uses 4 attention heads**

```
Head 1: Focuses on one type of relationship
Head 2: Focuses on another type
Head 3: Focuses on yet another type
Head 4: Focuses on a fourth type

Final: Concatenate or average all heads
```

Why multiple heads?
- Different perspectives on the graph
- Captures different types of relationships
- More robust predictions

## Architecture for Attack Prediction

### Input Graph

```
Nodes: MITRE ATT&CK techniques
Edges: Historical attack progressions

Example:
T0846 ──0.67──> T0843
  |              |
 0.23          0.45
  |              |
  v              v
T0800 ──0.78──> T0858

Edge weights = probability of progression
```

### Node Features

```
For each technique:
- One-hot encoding: [0, 0, 1, 0, ...] (which technique)
- Detection status: [1] if detected, [0] if not
- Time since detection: [0.5] (normalized)
- Facility context: [0.2, 0.8, ...] (facility-specific features)
```

### Model Architecture

```
Input: Current attack state (which techniques detected)
       Shape: (num_techniques, feature_dim)

GAT Layer 1: 
- 4 attention heads
- Hidden dim: 64
- Activation: ELU

GAT Layer 2:
- 4 attention heads  
- Hidden dim: 32
- Activation: ELU

Output Layer:
- Dense layer
- Softmax activation
- Shape: (num_techniques,) - probability for each technique

Output: Probability distribution over next techniques
```

### Prediction Process

```
Step 1: Encode current state
detected_techniques = [T0846]
node_features = encode(detected_techniques)

Step 2: Forward pass through GAT
layer1_output = GAT_layer1(node_features, graph_structure)
layer2_output = GAT_layer2(layer1_output, graph_structure)

Step 3: Predict next techniques
probabilities = softmax(output_layer(layer2_output))

Step 4: Return top predictions
T0843: 67% probability
T0800: 23% probability
T0858: 10% probability
```

## Training the GNN

### Training Data

```
Historical attack sequences from multiple facilities:

Sequence 1: T0846 → T0843 → T0858 → T0831
Sequence 2: T0846 → T0800 → T0836 → T0831
Sequence 3: T0800 → T0843 → T0858
...

Convert to training examples:
Input: T0846 detected → Target: T0843
Input: T0843 detected → Target: T0858
Input: T0858 detected → Target: T0831
```

### Loss Function

```python
# Cross-entropy loss
loss = -sum(target * log(predicted))

Example:
Target: T0843 (one-hot: [0, 1, 0, 0, ...])
Predicted: [0.1, 0.67, 0.15, 0.08, ...]

Loss = -(0*log(0.1) + 1*log(0.67) + 0*log(0.15) + ...)
     = -log(0.67)
     = 0.4

Lower loss = better prediction
```

### Federated Learning

```
Each facility:
1. Has local attack history
2. Trains GNN on local data (5 epochs)
3. Computes weight updates
4. Sends updates to FL server

FL server:
1. Aggregates updates from all facilities
2. Creates improved global model
3. Distributes to all facilities

Result: Model learns attack patterns from entire industry
```

## Why GNN Works for ICS

### 1. Captures Attack Chains

```
Attackers follow patterns:
Reconnaissance → Access → Execution → Impact

GNN learns these chains from graph structure
```

### 2. Generalizes Across Facilities

```
Power plant attack: T0846 → T0843 → T0858
Water utility attack: T0846 → T0843 → T0836
Manufacturing attack: T0846 → T0800 → T0858

GNN learns: T0846 commonly leads to T0843 or T0800
Applies to all facility types
```

### 3. Handles Partial Information

```
Only detected: T0846
GNN considers:
- Direct neighbors (T0843, T0800)
- 2-hop neighbors (T0858, T0836)
- Graph structure

Predicts most likely paths
```

### 4. Adapts to New Patterns

```
New attack chain observed:
T0846 → T0890 (new technique)

GNN updates:
- Adds edge T0846 → T0890
- Adjusts attention weights
- Improves future predictions
```

## Practical Example

### Scenario

```
08:00 - Detected: T0846 (Remote System Discovery)
Question: What will attacker do next?
```

### GNN Process

```
Step 1: Locate T0846 in graph
Current node: T0846

Step 2: Check neighbors
Connected to: T0843, T0800, T0858

Step 3: Apply attention
T0846 → T0843: attention = 0.67 (strong)
T0846 → T0800: attention = 0.23 (moderate)
T0846 → T0858: attention = 0.10 (weak)

Step 4: Aggregate information
Weighted sum of neighbor features

Step 5: Predict
T0843 (Program Download): 67%
T0800 (Lateral Movement): 23%
T0858 (Change Mode): 10%

Step 6: Alert
"High probability of Program Download attack on PLC-REACTOR-01 within 15-60 minutes"
```

### Defensive Action

```
Based on prediction:
1. Monitor PLC-REACTOR-01 closely
2. Block unauthorized program downloads
3. Alert security team
4. Prepare incident response

Result: Proactive defense before attack happens
```

## Implementation

```python
import torch
import torch.nn as nn
from torch_geometric.nn import GATConv

class AttackPredictionGNN(nn.Module):
    def __init__(self, num_techniques, hidden_dim=64, num_heads=4):
        super().__init__()
        
        # GAT Layer 1
        self.gat1 = GATConv(
            in_channels=num_techniques,
            out_channels=hidden_dim,
            heads=num_heads,
            dropout=0.6
        )
        
        # GAT Layer 2
        self.gat2 = GATConv(
            in_channels=hidden_dim * num_heads,
            out_channels=hidden_dim // 2,
            heads=num_heads,
            dropout=0.6
        )
        
        # Output layer
        self.output = nn.Linear(
            hidden_dim // 2 * num_heads,
            num_techniques
        )
    
    def forward(self, x, edge_index):
        # Layer 1
        x = self.gat1(x, edge_index)
        x = torch.elu(x)
        
        # Layer 2
        x = self.gat2(x, edge_index)
        x = torch.elu(x)
        
        # Output
        x = self.output(x)
        return torch.softmax(x, dim=1)
```

## Advantages

✅ **Relationship-aware:** Understands technique connections
✅ **Generalizable:** Learns patterns across facilities
✅ **Interpretable:** Can explain predictions via graph paths
✅ **Adaptive:** Updates as new attacks observed
✅ **Federated:** Benefits from collaborative learning
✅ **Proactive:** Predicts before attack happens

## Limitations

❌ **Requires graph structure:** Need attack chain data
❌ **Cold start:** Limited predictions for new techniques
❌ **Computational cost:** More expensive than simple rules
❌ **Graph quality:** Predictions only as good as graph
❌ **Novel attacks:** May miss completely new attack patterns

## Key Takeaways

1. **GNN operates on graph-structured data**
2. **Perfect for attack prediction** - techniques form a graph
3. **Attention mechanism** - focuses on important relationships
4. **2 layers, 4 heads** - project architecture
5. **Learns from federated data** - industry-wide patterns
6. **Enables proactive defense** - predict next attack step
7. **67% accuracy** - project target for next-step prediction

## Further Reading

- Paper: "Graph Attention Networks" (Veličković et al., 2018)
- Paper: "Semi-Supervised Classification with Graph Convolutional Networks" (Kipf & Welling, 2017)
- Tutorial: PyTorch Geometric documentation
- Book: "Graph Representation Learning" (Hamilton, 2020)
