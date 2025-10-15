# LSTM for Temporal Patterns

## What is LSTM?

**LSTM = Long Short-Term Memory**

A type of Recurrent Neural Network (RNN) designed to learn patterns in sequential data and remember information over long periods.

## The Problem: Memory in Neural Networks

### Regular Neural Networks

```
Input → Process → Output
(forgets everything immediately)

Example:
Input: Temperature = 85°C
Output: Normal or Anomaly?

Problem: No context! Is 85°C normal?
- During startup: Yes
- During shutdown: No
- During normal operation: Maybe
```

### Need for Memory

In ICS, context matters:
- **Sequences:** Attacks unfold over multiple steps
- **Trends:** Gradual changes indicate problems
- **Patterns:** Normal behavior varies by time/state
- **History:** Current state depends on past states

## How LSTM Works

### Basic Architecture

```
Input Sequence → [LSTM Cell] → Output Sequence
                      ↓
                 Memory State
```

### LSTM Cell Components

**1. Cell State (Long-term Memory)**
```
Carries information through the sequence
Like a conveyor belt running through the chain
```

**2. Hidden State (Short-term Memory)**
```
Current output and working memory
Passed to next time step
```

**3. Gates (Control Information Flow)**

**Forget Gate:**
```
Decides what to forget from cell state
"Should I remember the startup phase? No, we're in normal operation now"
```

**Input Gate:**
```
Decides what new information to store
"This unusual command sequence is important, remember it"
```

**Output Gate:**
```
Decides what to output
"Based on history and current input, output: ANOMALY"
```

### Visual Representation

```
Time Step 1:
Input₁ → [LSTM] → Output₁
           ↓
        Memory₁

Time Step 2:
Input₂ + Memory₁ → [LSTM] → Output₂
                      ↓
                   Memory₂

Time Step 3:
Input₃ + Memory₂ → [LSTM] → Output₃
                      ↓
                   Memory₃
```

## LSTM vs Regular RNN

### Regular RNN Problem: Vanishing Gradients

```
Long sequence: Input₁ → Input₂ → ... → Input₁₀₀

Problem: Information from Input₁ is lost by Input₁₀₀
Cannot learn long-term dependencies
```

### LSTM Solution

```
Cell state acts as highway for information
Can remember information for 100+ time steps
Learns what to remember and what to forget
```

## LSTM Autoencoder

### What is an Autoencoder?

```
Input → [Encoder] → Compressed → [Decoder] → Reconstructed Output

Goal: Reconstructed ≈ Input

If reconstruction is good → Input is "normal" (seen before)
If reconstruction is poor → Input is "anomaly" (never seen)
```

### LSTM Autoencoder Architecture

```
Input Sequence → [LSTM Encoder] → Latent Vector → [LSTM Decoder] → Reconstructed Sequence

Example:
Input: [temp₁, temp₂, temp₃, temp₄, temp₅]
Encoder: Compresses to latent vector [0.5, -0.3, 0.8]
Decoder: Reconstructs [temp₁', temp₂', temp₃', temp₄', temp₅']

If input ≈ reconstructed → Normal
If input ≠ reconstructed → Anomaly
```

### Training Process

**Phase 1: Training on Normal Data**
```
Feed normal operational sequences:
- Normal startup procedures
- Normal operator actions  
- Normal process variations
- Normal sensor readings

LSTM learns: "This is what normal looks like"
```

**Phase 2: Anomaly Detection**
```
New sequence arrives:
1. LSTM tries to reconstruct it
2. Compute reconstruction error
3. If error > threshold → ANOMALY

Reconstruction Error = |Input - Reconstructed|
```

## Why LSTM for ICS Security?

### 1. Time-Series Data

ICS generates continuous streams:
```
Sensor readings every second:
08:00:00 - Temp: 45°C, Pressure: 100 PSI
08:00:01 - Temp: 45.1°C, Pressure: 100 PSI
08:00:02 - Temp: 45.2°C, Pressure: 101 PSI
...

LSTM naturally handles this temporal data
```

### 2. Sequential Attacks

Attacks have multiple steps:
```
Normal Sequence:
Login → Read sensors → Adjust setpoint → Logout

Attack Sequence:
Login → Read ALL programs → Download config → Modify program → Logout
         ↑ Unusual step

LSTM detects: "This sequence doesn't match learned patterns"
```

### 3. Context Awareness

```
Scenario: Temperature rises to 80°C

Context 1 (Startup):
Previous: 20°C → 40°C → 60°C → 80°C
LSTM: "Normal startup pattern" ✓

Context 2 (Normal Operation):
Previous: 50°C → 50°C → 50°C → 80°C
LSTM: "Abnormal jump!" ✗

Same value, different context, different decision
```

### 4. Gradual Attacks

```
Attacker slowly changes setpoint:
Day 1: 50°C (normal)
Day 2: 51°C (normal)
Day 3: 52°C (normal)
...
Day 30: 80°C (dangerous!)

Each individual change looks normal
But the trend is abnormal

LSTM detects: "This gradual drift is unusual"
```

## Practical Example

### Normal Operation Pattern

```
Time Series (10 seconds):
[45, 45, 46, 46, 45, 45, 46, 45, 45, 46]

LSTM Autoencoder:
Input: [45, 45, 46, 46, 45, 45, 46, 45, 45, 46]
Reconstructed: [45.1, 45.0, 45.9, 46.1, 45.0, 45.1, 45.9, 45.0, 45.1, 46.0]
Error: 0.15 (low) → NORMAL ✓
```

### Attack Pattern

```
Time Series (10 seconds):
[45, 45, 46, 85, 85, 85, 85, 85, 85, 85]
                ↑ Sudden jump (attack)

LSTM Autoencoder:
Input: [45, 45, 46, 85, 85, 85, 85, 85, 85, 85]
Reconstructed: [45.1, 45.0, 45.9, 46.2, 46.5, 47.0, 47.5, 48.0, 48.5, 49.0]
                                ↑ Cannot reconstruct the jump
Error: 35.8 (high) → ANOMALY ✗
```

## Architecture Details

### Encoder

```python
Input: Sequence of length T
       Shape: (batch_size, T, features)

LSTM Layer 1: 128 units
LSTM Layer 2: 64 units
Output: Latent vector
        Shape: (batch_size, 64)
```

### Decoder

```python
Input: Latent vector
       Shape: (batch_size, 64)

LSTM Layer 1: 64 units
LSTM Layer 2: 128 units
Output: Reconstructed sequence
        Shape: (batch_size, T, features)
```

### Loss Function

```python
# Mean Squared Error between input and reconstruction
loss = mean((input - reconstructed)²)

# During training, minimize this loss
# During detection, if loss > threshold → anomaly
```

## Training in Federated Learning

### Local Training

```
Each facility:
1. Collects normal operational data
2. Trains LSTM Autoencoder locally (5 epochs)
3. Computes weight updates
4. Sends updates to FL server
```

### Benefits of Federation

```
Facility A learns: Power plant startup patterns
Facility B learns: Water utility patterns
Facility C learns: Manufacturing patterns

Combined model learns:
- General attack patterns (common across facilities)
- Diverse normal behaviors (reduces false positives)
- Industry-wide threat intelligence
```

### Why It Works

```
Attack patterns generalize:
- Reconnaissance → Lateral Movement → Execution
- Similar across different facilities
- LSTM learns these universal patterns

Normal patterns vary:
- Each facility has unique baselines
- Local physics rules handle facility-specific behavior
- LSTM handles general behavioral patterns
```

## Hyperparameters

### Sequence Length
```
Too short (e.g., 5 seconds):
- Misses long-term patterns
- Cannot detect slow attacks

Too long (e.g., 1 hour):
- Too much data
- Slow processing
- Overfitting

Optimal: 30-60 seconds
- Captures attack sequences
- Fast processing
- Good generalization
```

### Number of LSTM Units
```
Too few (e.g., 16):
- Cannot learn complex patterns
- Underfitting

Too many (e.g., 512):
- Overfitting
- Slow training
- Large model size

Optimal: 64-128 units
- Good capacity
- Reasonable speed
- Generalizes well
```

### Threshold for Anomaly
```
Too low (e.g., 0.1):
- Many false positives
- Alert fatigue

Too high (e.g., 10.0):
- Misses attacks
- False negatives

Optimal: Tune on validation data
- Balance precision and recall
- Typically 95th percentile of normal errors
```

## Implementation Example

```python
import tensorflow as tf

# Encoder
encoder_inputs = tf.keras.Input(shape=(sequence_length, n_features))
encoder_lstm1 = tf.keras.layers.LSTM(128, return_sequences=True)(encoder_inputs)
encoder_lstm2 = tf.keras.layers.LSTM(64)(encoder_lstm1)

# Decoder
decoder_lstm1 = tf.keras.layers.RepeatVector(sequence_length)(encoder_lstm2)
decoder_lstm2 = tf.keras.layers.LSTM(64, return_sequences=True)(decoder_lstm1)
decoder_lstm3 = tf.keras.layers.LSTM(128, return_sequences=True)(decoder_lstm2)
decoder_outputs = tf.keras.layers.TimeDistributed(
    tf.keras.layers.Dense(n_features)
)(decoder_lstm3)

# Model
autoencoder = tf.keras.Model(encoder_inputs, decoder_outputs)
autoencoder.compile(optimizer='adam', loss='mse')

# Training
autoencoder.fit(normal_sequences, normal_sequences, epochs=50)

# Detection
reconstruction = autoencoder.predict(new_sequence)
error = np.mean((new_sequence - reconstruction) ** 2)
is_anomaly = error > threshold
```

## Advantages for ICS

✅ **Temporal Understanding:** Captures time-based patterns
✅ **Context Awareness:** Considers history
✅ **Unsupervised Learning:** No need for labeled attack data
✅ **Adaptable:** Learns new normal patterns
✅ **Federated:** Benefits from multi-facility learning
✅ **Real-time:** Fast inference (<100ms)

## Limitations

❌ **Training Data:** Needs clean normal data
❌ **Threshold Tuning:** Requires careful calibration
❌ **Novel Attacks:** May miss completely new attack types
❌ **Computational Cost:** More expensive than simple rules
❌ **Interpretability:** Hard to explain why it flagged something

## Key Takeaways

1. **LSTM has memory** - Remembers past inputs
2. **Perfect for sequences** - ICS data is temporal
3. **Autoencoder detects anomalies** - Reconstruction error
4. **Context-aware** - Same value, different meaning
5. **Federated learning improves it** - Learns from multiple facilities
6. **Complements other methods** - Use with Isolation Forest and physics rules
7. **Project uses it for behavioral detection** - Core component

## Further Reading

- Paper: "LSTM Autoencoder for Anomaly Detection" (Malhotra et al., 2016)
- Tutorial: "Understanding LSTM Networks" (Colah's blog)
- Book: "Deep Learning" (Goodfellow et al.) - Chapter on RNNs
- Implementation: TensorFlow/Keras LSTM documentation
