# Differential Privacy (ε, δ)

## What is Differential Privacy?

Differential privacy is a mathematical framework that provides provable privacy guarantees when sharing statistical information about a dataset. It ensures that the presence or absence of any single individual's data doesn't significantly affect the output.

## Core Concept

**Goal:** Share useful aggregate information while protecting individual privacy.

**Method:** Add carefully calibrated random noise to the data or results.

## The Parameters

### ε (Epsilon) - Privacy Budget

- **Controls the privacy-utility tradeoff**
- **Lower ε = More private** (more noise added)
- **Higher ε = Less private** (less noise, more accurate)

**Typical Values:**
- ε < 1.0: Strong privacy (academic standard)
- ε = 1.0 - 3.0: Moderate privacy (practical applications)
- ε > 5.0: Weak privacy (minimal protection)

**Project Value: ε = 2.0** (moderate, balanced)

### δ (Delta) - Failure Probability

- **Probability that the privacy guarantee fails**
- **Lower δ = Stronger guarantee**
- Typically very small: 10⁻⁵ to 10⁻⁷

**Project Value: δ = 10⁻⁵ = 0.00001 = 0.001%**

## How It Works

### Basic Mechanism

```
Original value: 42
Noise (from ε): ±3
Published value: 45

An adversary cannot determine if the true value was 42, 40, or 45
```

### In Federated Learning

```
1. Facility trains model locally
2. Computes weight updates
3. Adds noise calibrated to ε before sending
4. Server receives noisy weights
5. Aggregates across facilities (noise averages out)
```

## Mathematical Guarantee

**(ε, δ)-Differential Privacy:**

For any two datasets D₁ and D₂ that differ by one record:

```
P[M(D₁) ∈ S] ≤ e^ε × P[M(D₂) ∈ S] + δ
```

Where:
- M = Mechanism (algorithm)
- S = Any possible output
- e^ε ≈ 7.4 for ε=2.0

**Interpretation:** The output distribution changes by at most a factor of e^ε when one record changes.

## Why It Matters for ICS

### 1. Regulatory Compliance
- NERC CIP, IEC 62443, GDPR requirements
- Provable privacy guarantees
- Auditable protection

### 2. Competitive Protection
- Operational data stays private
- Production rates not revealed
- Equipment configurations protected

### 3. Trust Building
- Mathematical proof, not just promises
- Facilities more willing to participate
- Enables broader collaboration

## Practical Example

### Without Differential Privacy

```
Facility A: Attack detected at 08:23, PLC-REACTOR-01
Sends: weights = [0.543, -0.234, 0.891, ...]

Risk: Weights might reveal:
- Specific equipment configuration
- Attack timing patterns
- Operational parameters
```

### With Differential Privacy (ε=2.0)

```
Facility A: Attack detected at 08:23, PLC-REACTOR-01
Adds noise: Laplace(0, Δf/ε)
Sends: weights = [0.556, -0.221, 0.903, ...]

Protection: Cannot reverse-engineer specific details
Utility: Pattern still useful for learning
```

## Noise Mechanisms

### Laplace Mechanism
- Most common for numerical data
- Noise ~ Laplace(0, Δf/ε)
- Δf = sensitivity (max change from one record)

### Gaussian Mechanism
- Used for (ε, δ)-DP
- Noise ~ Gaussian(0, σ²)
- σ calibrated to ε and δ

## Privacy Budget Management

### Composition
Multiple queries consume privacy budget:
```
Query 1: ε₁ = 1.0
Query 2: ε₂ = 1.0
Total: ε_total = 2.0
```

### In Federated Learning
Each training round consumes budget:
```
Round 1: ε = 0.5
Round 2: ε = 0.5
Round 3: ε = 0.5
Round 4: ε = 0.5
Total per day: ε = 2.0
```

## Trade-offs

### More Privacy (Lower ε)
✅ Stronger protection
✅ Better compliance
❌ Less accurate models
❌ Slower learning

### Less Privacy (Higher ε)
✅ More accurate models
✅ Faster learning
❌ Weaker protection
❌ Potential data leakage

### Project Choice (ε=2.0)
⚖️ Balanced approach
- Sufficient privacy for critical infrastructure
- Good model accuracy
- Industry-standard value

## Implementation Tools

### Python Libraries

**Opacus (PyTorch):**
```python
from opacus import PrivacyEngine

privacy_engine = PrivacyEngine()
model, optimizer, data_loader = privacy_engine.make_private(
    module=model,
    optimizer=optimizer,
    data_loader=data_loader,
    noise_multiplier=1.1,
    max_grad_norm=1.0,
)
```

**TensorFlow Privacy:**
```python
from tensorflow_privacy import DPGradientDescentOptimizer

optimizer = DPGradientDescentOptimizer(
    l2_norm_clip=1.0,
    noise_multiplier=1.1,
    num_microbatches=1,
    learning_rate=0.01
)
```

## Key Takeaways

1. **ε controls privacy-utility tradeoff** - Lower is more private
2. **δ is failure probability** - Should be very small
3. **Noise is calibrated mathematically** - Not random guessing
4. **Provides provable guarantees** - Can prove privacy level
5. **Essential for federated learning** - Enables safe collaboration
6. **Project uses (ε=2.0, δ=10⁻⁵)** - Moderate, practical privacy

## Further Reading

- Original paper: Dwork (2006) "Differential Privacy"
- Book: "The Algorithmic Foundations of Differential Privacy"
- Tutorial: https://programming-dp.com/
- Opacus docs: https://opacus.ai/
