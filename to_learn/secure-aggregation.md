# Secure Aggregation for Privacy

## What is Secure Aggregation?

A cryptographic technique that allows a server to compute the sum/average of values from multiple parties without seeing individual contributions.

**Key Property:** Server sees the aggregate, but not individual values.

## The Problem

### Without Secure Aggregation

```
Facility A sends: weights_A = [0.5, -0.3, 0.8]
Facility B sends: weights_B = [0.6, -0.2, 0.7]
Facility C sends: weights_C = [0.4, -0.4, 0.9]

Server receives all three individually:
- Can see exactly what Facility A learned
- Can see exactly what Facility B learned
- Can see exactly what Facility C learned
- Could potentially reverse-engineer sensitive data
```

### Privacy Risks

Even though only weights are sent (not raw data):
```
Server could infer:
- "Facility A had unusual patterns" (maybe they were attacked)
- "Facility B's model changed drastically" (operational changes?)
- Relative differences between facilities
- Sensitive operational information
```

## The Solution

### With Secure Aggregation

```
Facility A: weights_A (encrypted/masked)
Facility B: weights_B (encrypted/masked)
Facility C: weights_C (encrypted/masked)

Server can ONLY compute:
Sum = weights_A + weights_B + weights_C

Server CANNOT see:
- Individual weights_A
- Individual weights_B
- Individual weights_C
```

## How It Works

### Method 1: Secret Sharing (Simplified)

**Step 1: Facilities Exchange Keys**
```
Facility A ↔ Facility B: Share secret key K_AB
Facility A ↔ Facility C: Share secret key K_AC
Facility B ↔ Facility C: Share secret key K_BC
```

**Step 2: Generate Masks**
```
Facility A generates:
- mask_AB from K_AB (shared with B)
- mask_AC from K_AC (shared with C)

Facility B generates:
- mask_BA from K_AB (same as mask_AB but opposite sign)
- mask_BC from K_BC (shared with C)

Facility C generates:
- mask_CA from K_AC (same as mask_AC but opposite sign)
- mask_CB from K_BC (same as mask_BC but opposite sign)
```

**Step 3: Mask and Send**
```
Facility A sends: weights_A + mask_AB + mask_AC
Facility B sends: weights_B + mask_BA + mask_BC
Facility C sends: weights_C + mask_CA + mask_CB
```

**Step 4: Server Aggregates**
```
Sum = (weights_A + mask_AB + mask_AC) +
      (weights_B + mask_BA + mask_BC) +
      (weights_C + mask_CA + mask_CB)

Masks cancel out:
mask_AB + mask_BA = 0
mask_AC + mask_CA = 0
mask_BC + mask_CB = 0

Result: weights_A + weights_B + weights_C

Server gets correct sum without seeing individual weights!
```

## Real-World Analogy

### Salary Averaging Without Revealing Salaries

Three employees want average salary without revealing individual salaries:

**Without Secure Aggregation:**
```
Alice: $50,000
Bob: $60,000
Carol: $70,000
Everyone sees everyone's salary 😞
```

**With Secure Aggregation:**
```
Alice and Bob agree on secret: +$10,000 / -$10,000
Alice and Carol agree on secret: +$5,000 / -$5,000
Bob and Carol agree on secret: +$3,000 / -$3,000

Alice sends: $50,000 + $10,000 + $5,000 = $65,000
Bob sends: $60,000 - $10,000 + $3,000 = $53,000
Carol sends: $70,000 - $5,000 - $3,000 = $62,000

Sum: $65,000 + $53,000 + $62,000 = $180,000 ✓
Average: $60,000 ✓

No one knows individual salaries! 😊
```

## Mathematical Foundation

### Additive Secret Sharing

```
Secret: s
Split into n shares: s₁, s₂, ..., sₙ

Property: s = s₁ + s₂ + ... + sₙ

Each party holds one share
No single share reveals anything about s
Only the sum reveals s
```

### Pairwise Masking

```
For n parties, generate n(n-1)/2 pairwise masks

Example with 3 parties:
Masks: m_AB, m_AC, m_BC

Party A: value_A + m_AB + m_AC
Party B: value_B - m_AB + m_BC
Party C: value_C - m_AC - m_BC

Sum: value_A + value_B + value_C
(all masks cancel)
```

## Advanced: Handling Dropouts

### Problem

```
What if Facility B drops out?

Facility A sent: weights_A + mask_AB + mask_AC
Facility C sent: weights_C - mask_AC - mask_BC

Sum: weights_A + weights_C + mask_AB - mask_BC

Masks don't cancel! Sum is corrupted.
```

### Solution: Threshold Secret Sharing

```
Use Shamir's Secret Sharing:
- Generate masks that can be reconstructed if t out of n parties participate
- If Facility B drops, others can reconstruct mask_AB and mask_BC
- Remove the masks from the sum
- Get correct aggregate

Requires: At least t parties participate
```

## Security Properties

### What Secure Aggregation Protects

✅ **Individual Privacy:**
```
Server cannot see individual contributions
Even if server is compromised
```

✅ **Collusion Resistance:**
```
Server + some facilities cannot learn about others
Requires threshold number of colluders
```

✅ **Verifiability:**
```
Can verify that aggregation was done correctly
Detect if server cheats
```

### What It Doesn't Protect

❌ **Aggregate Information:**
```
Server sees the sum/average
This is intentional (needed for federated learning)
```

❌ **Inference Attacks:**
```
If only 2 facilities, and you know one, you can infer the other
Requires sufficient number of participants
```

❌ **Malicious Inputs:**
```
Doesn't prevent facilities from sending garbage
Need Byzantine-robust aggregation for that
```

## Implementation

### Pseudocode

```python
class SecureAggregation:
    def __init__(self, facilities):
        self.facilities = facilities
        self.pairwise_keys = {}
    
    def setup_phase(self):
        """Facilities exchange pairwise keys"""
        for i in range(len(self.facilities)):
            for j in range(i+1, len(self.facilities)):
                key = generate_shared_key(facilities[i], facilities[j])
                self.pairwise_keys[(i, j)] = key
    
    def mask_weights(self, facility_id, weights):
        """Generate masked weights for a facility"""
        masked = weights.copy()
        
        for other_id in range(len(self.facilities)):
            if other_id == facility_id:
                continue
            
            # Get pairwise key
            if facility_id < other_id:
                key = self.pairwise_keys[(facility_id, other_id)]
                sign = +1
            else:
                key = self.pairwise_keys[(other_id, facility_id)]
                sign = -1
            
            # Generate mask from key
            mask = generate_mask(key, len(weights))
            
            # Apply mask
            masked += sign * mask
        
        return masked
    
    def aggregate(self, masked_weights_list):
        """Server aggregates masked weights"""
        # Simply sum all masked weights
        # Masks cancel out automatically
        return sum(masked_weights_list)
```

### Using Cryptographic Libraries

```python
from cryptography.hazmat.primitives import hashes
from cryptography.hazmat.primitives.kdf.hkdf import HKDF

def generate_mask(shared_key, size):
    """Generate deterministic mask from shared key"""
    # Use key derivation function
    kdf = HKDF(
        algorithm=hashes.SHA256(),
        length=size * 4,  # 4 bytes per float
        salt=None,
        info=b'secure aggregation mask'
    )
    
    # Derive mask bytes
    mask_bytes = kdf.derive(shared_key)
    
    # Convert to float array
    mask = np.frombuffer(mask_bytes, dtype=np.float32)
    
    return mask
```

## Integration with Other Privacy Techniques

### Layered Privacy Protection

```
Layer 1: Data Isolation
- Raw data never leaves facility

Layer 2: Model Weights Only
- Only send model parameters, not data

Layer 3: Differential Privacy
- Add calibrated noise to weights

Layer 4: Secure Aggregation
- Server cannot see individual noisy weights

Layer 5: Byzantine-Robust
- Exclude malicious updates

Result: Maximum privacy with useful collaboration
```

### Differential Privacy + Secure Aggregation

```
Facility A:
1. Compute weights: w_A
2. Add DP noise: w_A' = w_A + noise
3. Apply secure aggregation mask: w_A'' = w_A' + masks
4. Send w_A'' to server

Server:
- Receives masked, noisy weights
- Cannot see individual weights (secure aggregation)
- Cannot reverse-engineer data (differential privacy)
- Can compute aggregate for federated learning
```

## Performance Considerations

### Communication Overhead

```
Without Secure Aggregation:
- Each facility sends: 10 MB

With Secure Aggregation:
- Each facility sends: 10 MB (same!)
- Additional: Key exchange (one-time, small)

Overhead: Minimal
```

### Computation Overhead

```
Mask Generation:
- Deterministic from shared keys
- Fast (milliseconds)

Aggregation:
- Same as regular sum
- No overhead

Total: Negligible overhead
```

### Setup Cost

```
Key Exchange:
- One-time setup
- O(n²) pairwise keys for n facilities
- Can be done offline

For 10 facilities:
- 45 pairwise keys
- Setup time: Seconds
```

## Why It Matters for ICS

### 1. Regulatory Compliance

```
NERC CIP, IEC 62443, GDPR:
- Data must not leave facility
- Even processed data has restrictions

Secure Aggregation:
- Server never sees individual facility data
- Meets compliance requirements
```

### 2. Competitive Protection

```
Facilities don't want to reveal:
- Operational patterns
- Equipment configurations
- Production schedules

Secure Aggregation:
- Protects competitive information
- Enables collaboration without exposure
```

### 3. Trust Building

```
Facilities more willing to participate:
- "Even the server operator can't see my data"
- Mathematical guarantee, not just policy
- Enables broader industry collaboration
```

### 4. Defense in Depth

```
Even if other protections fail:
- Differential privacy noise removed
- Byzantine attacker on server

Secure Aggregation still protects:
- Individual contributions remain hidden
- Last line of defense
```

## Key Takeaways

1. **Secure aggregation hides individual contributions**
2. **Server only sees the sum/average**
3. **Uses cryptographic masking** - masks cancel in aggregate
4. **Minimal overhead** - same communication cost
5. **Complements differential privacy** - layered protection
6. **Essential for trust** - enables participation
7. **Project uses it** - core privacy component

## Further Reading

- Paper: "Practical Secure Aggregation for Privacy-Preserving Machine Learning" (Bonawitz et al., 2017)
- Paper: "Secure Aggregation for Federated Learning" (Google, 2017)
- Implementation: TensorFlow Federated secure aggregation
- Tutorial: "Cryptographic Techniques for Federated Learning"
