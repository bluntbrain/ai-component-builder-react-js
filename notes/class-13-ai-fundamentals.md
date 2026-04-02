# Class 13: Artificial Intelligence -- The Complete Knowledge Guide

## What You'll Learn

- What AI actually is and the difference between AI, ML, and Deep Learning
- How neural networks work (weights, biases, layers, activation functions)
- The Transformer architecture -- the breakthrough behind ChatGPT, Claude, and Gemini
- How Large Language Models (LLMs) are trained -- pre-training, fine-tuning, RLHF
- Tokens, embeddings, and how AI "understands" language
- Prompt engineering -- how to talk to AI effectively
- RAG, AI agents, and function calling
- How image generation works (diffusion models)
- Open source vs closed source models
- GPUs, NVIDIA, and why hardware matters
- AI safety, alignment, and hallucinations
- Key companies and models you should know
- Enough vocabulary to discuss AI confidently with anyone

## Who This Is For

You don't need any math or coding background. This is a **knowledge class** -- by the end, you should understand what's happening under the hood when you use ChatGPT, why everyone is talking about AI, and be able to hold an intelligent conversation about the technology that's reshaping every industry.

---

# Module 1: Foundations

---

## 1. What is Artificial Intelligence?

**AI** is the field of computer science that aims to create machines that can perform tasks that normally require human intelligence -- like understanding language, recognizing images, making decisions, and generating creative content.

```
  THE AI UMBRELLA -- each layer is a subset of the one above:

  +===========================================================+
  |                                                           |
  |   ARTIFICIAL INTELLIGENCE (AI)                            |
  |   Any technique that enables machines to mimic            |
  |   human-like intelligence                                 |
  |                                                           |
  |   +===================================================+   |
  |   |                                                   |   |
  |   |   MACHINE LEARNING (ML)                           |   |
  |   |   Machines learn from data instead of             |   |
  |   |   being explicitly programmed                     |   |
  |   |                                                   |   |
  |   |   +============================================+  |   |
  |   |   |                                            |  |   |
  |   |   |   DEEP LEARNING                            |  |   |
  |   |   |   ML using neural networks with            |  |   |
  |   |   |   many layers (GPT, DALL-E, etc.)          |  |   |
  |   |   |                                            |  |   |
  |   |   +============================================+  |   |
  |   |                                                   |   |
  |   +===================================================+   |
  |                                                           |
  +===========================================================+

  AI > ML > Deep Learning
  (broadest)          (most specific -- what powers ChatGPT)
```

### Three types of AI

| Type | What it means | Example | Status |
|------|--------------|---------|--------|
| **Narrow AI (ANI)** | Good at ONE specific task | ChatGPT, Siri, spam filters | Exists today |
| **General AI (AGI)** | Human-level intelligence across ALL tasks | A machine that can learn anything a human can | Does not exist yet |
| **Super AI (ASI)** | Surpasses human intelligence in every way | Science fiction (for now) | Theoretical |

Everything we have today -- ChatGPT, Claude, Midjourney, self-driving cars -- is **Narrow AI**. Extremely capable at specific tasks, but can't generalize the way humans do. The race toward AGI is what's driving billions in investment.

---

## 2. Brief History of AI

```
  AI TIMELINE:

  1950        1956        1970s-80s     1997         2012          2017
   |           |             |            |            |             |
   v           v             v            v            v             v
  Turing     "AI" term    AI Winters   Deep Blue    AlexNet      Transformer
  Test       coined at    (hype died,  beats chess  wins image   paper:
  proposed   Dartmouth    funding cut) champion     competition  "Attention Is
             conference                Kasparov     (deep        All You Need"
                                                   learning
                                                   takes off)
                                                                     |
  2018        2020         2022          2023         2024            |
   |           |             |            |            |              |
   v           v             v            v            v              |
  GPT-1      GPT-3        ChatGPT      GPT-4       Claude 3,     <--+
  (OpenAI)   (few-shot    launches     multimodal   Gemini,       This paper
              learning)   (Nov 30)     (text+image) Llama 3,      changed
                          AI goes                   open source   EVERYTHING
                          mainstream                explosion
```

The key turning point: **2017's Transformer paper** by Google researchers. Everything since -- GPT, BERT, Claude, Gemini, Llama, Stable Diffusion -- is built on the Transformer architecture.

---

## 3. Traditional Programming vs Machine Learning

```
  TRADITIONAL PROGRAMMING:

  +--------+
  | Rules  |  +--------+     +---------+
  | (code) |  | Input  | --> | Output  |
  | (if/   |  | (data) |     | (answer)|
  |  else) |  +--------+     +---------+
  +--------+
  Human writes the rules explicitly.
  Example: "IF temperature > 30, show 'hot'"


  MACHINE LEARNING:

  +--------+
  | Input  |  +----------+     +---------+
  | (data) |  | Expected | --> | Rules   |
  |        |  | Output   |     | (model  |
  +--------+  | (labels) |     |  learns |
              +----------+     |  them!) |
                               +---------+
  Machine DISCOVERS the rules from examples.
  Example: Show 1 million photos of cats and dogs.
           Model learns to tell them apart on its own.


  KEY DIFFERENCE:
  Traditional: Human writes rules --> computer follows them
  ML: Human provides examples --> computer discovers rules
```

---

## 4. Types of Machine Learning

### Supervised Learning

The model learns from **labeled data** (input-output pairs).

```
  SUPERVISED LEARNING -- learning from examples with answers:

  Training data (labeled):
  +------------------+----------+
  | Photo            | Label    |
  +------------------+----------+
  | [image of cat]   | "cat"    |
  | [image of dog]   | "dog"    |
  | [image of cat]   | "cat"    |
  | [image of dog]   | "dog"    |
  | ... millions     | ...      |
  +------------------+----------+
           |
           v
     Model trains on these examples
           |
           v
     New photo: [image of cat]
     Model predicts: "cat" (97% confident)
```

**Examples:** Spam detection, image classification, price prediction, medical diagnosis

### Unsupervised Learning

The model finds **patterns in unlabeled data** -- no one tells it what to look for.

```
  UNSUPERVISED LEARNING -- finding patterns without labels:

  Input: 10,000 customer purchase records (no labels)

  +--------+  +--------+  +--------+
  | Cust.  |  | Cust.  |  | Cust.  |
  | buys   |  | buys   |  | buys   |
  | tech,  |  | food,  |  | luxury,|
  | gaming |  | health |  | travel |
  +--------+  +--------+  +--------+
       \          |          /
        \         |         /
         v        v        v
     Model discovers 3 clusters:
     "Tech enthusiasts", "Health-conscious", "Big spenders"
     (nobody told it these categories exist)
```

**Examples:** Customer segmentation, anomaly detection, recommendation systems

### Reinforcement Learning

The model learns by **trial and error**, receiving rewards or penalties.

```
  REINFORCEMENT LEARNING -- learning by doing:

  +-------+     action      +-------------+
  | Agent | -------------> | Environment |
  |       | <------------- |             |
  +-------+  reward/penalty +-------------+

  Example: Training a robot to walk

  Step 1: Robot falls  --> penalty (-1)
  Step 2: Robot wobbles --> small reward (+0.1)
  Step 3: Robot takes a step --> big reward (+1)
  ...
  Step 10000: Robot walks smoothly --> maximum reward

  The agent learns to maximize rewards over time.
```

**Examples:** Game-playing AI (AlphaGo, chess), robotics, self-driving cars, RLHF for ChatGPT

---

# Module 2: Neural Networks & Deep Learning

---

## 5. What is a Neural Network?

A neural network is a computing system inspired by (but not identical to) the human brain. It's made of layers of connected "neurons" that process information.

```
  A SIMPLE NEURAL NETWORK:

  INPUT LAYER        HIDDEN LAYER        OUTPUT LAYER
  (raw data)         (learns patterns)   (prediction)

    [x1] ----\       /--- [h1] ---\
               \     /              \
    [x2] ------[w]--+--- [h2] ------[w]--- [output]
               /     \              /
    [x3] ----/       \--- [h3] ---/

  x1, x2, x3 = input features (e.g., pixels, words, numbers)
  h1, h2, h3 = hidden neurons (learn intermediate patterns)
  w = weights (how strongly each connection matters)
  output = prediction (e.g., "cat" or "dog")

  Each connection has a WEIGHT (a number).
  The network LEARNS by adjusting these weights.
```

### How a single neuron works

```
  ONE NEURON:

  inputs        weights          sum + activation        output
  +----+
  | x1 | ---(w1 = 0.5)---\
  +----+                   \
  +----+                    +---> [SUM] ---> [Activation] ---> output
  | x2 | ---(w2 = -0.3)---/        |          Function
  +----+                    /       |
  +----+                   /        |
  | x3 | ---(w3 = 0.8)---/    + bias (b)
  +----+

  output = activation(w1*x1 + w2*x2 + w3*x3 + b)

  Weight = how important each input is
  Bias = adjusts the threshold for activation
  Activation function = decides whether the neuron "fires"
```

### What is an activation function?

Without activation functions, a neural network is just linear math (straight lines). Activation functions add **non-linearity** -- the ability to learn curves, patterns, and complex relationships.

| Function | What it does | Used in |
|----------|-------------|---------|
| **ReLU** | If positive, keep it. If negative, make it 0. | Most hidden layers |
| **Sigmoid** | Squashes output to 0-1 range | Binary classification |
| **Softmax** | Converts to probability distribution (sums to 1) | Multi-class output |
| **GELU** | Smooth version of ReLU | Transformers (GPT, BERT) |

---

## 6. How Neural Networks Learn (Training)

```
  THE TRAINING LOOP:

  +--------+     +--------+     +----------+     +--------+
  | 1.     | --> | 2.     | --> | 3.       | --> | 4.     |
  | Forward |     | Calculate|   | Backward |     | Update |
  | Pass    |     | Loss    |     | Pass     |     | Weights|
  +--------+     +--------+     +----------+     +--------+
       ^                                               |
       |                                               |
       +----------- REPEAT for millions of examples ---+

  Step 1 - FORWARD PASS:
    Feed input through network, get a prediction

  Step 2 - CALCULATE LOSS:
    Compare prediction to correct answer
    Loss = how wrong the prediction was
    (e.g., predicted "dog" but answer was "cat" = high loss)

  Step 3 - BACKWARD PASS (Backpropagation):
    Figure out which weights caused the error
    Calculate how to adjust each weight to reduce the error

  Step 4 - UPDATE WEIGHTS:
    Nudge each weight slightly in the right direction
    (using an "optimizer" like Adam or SGD)

  Repeat millions of times until loss is minimized.
```

### Key training concepts

| Term | What it means |
|------|--------------|
| **Epoch** | One complete pass through the entire training dataset |
| **Batch size** | Number of examples processed at once (e.g., 32, 64, 256) |
| **Learning rate** | How big of a step to take when adjusting weights (too big = overshoot, too small = slow) |
| **Loss function** | Measures how wrong the prediction is (lower = better) |
| **Overfitting** | Model memorizes training data but fails on new data |
| **Underfitting** | Model is too simple to capture the patterns |

```
  OVERFITTING vs UNDERFITTING vs JUST RIGHT:

  Underfitting          Just Right            Overfitting
  (too simple)          (generalizes well)    (memorized data)

      .  .                  .  .                  .  .
    .      .              .      .              .~    .~
   .        .            .   ---  .            ./ \  / \.
  .     ---  .          . --/    \--.         ./   \/   \.
  .  ---      .        .-/         \-.       /     /\    \
  .--          .      ./             \.     .     .  .    .

  Straight line:       Smooth curve:         Wiggly line:
  misses the pattern   captures the trend    fits noise, not signal
```

---

## 7. Deep Learning -- Why "Deep"?

**Deep learning** = neural networks with **many layers** (deep = many layers stacked).

```
  SHALLOW vs DEEP:

  Shallow (1-2 hidden layers):       Deep (many hidden layers):

  Input -> [H] -> Output              Input -> [H] -> [H] -> [H] -> [H] -> [H] -> Output
                                                |       |       |       |       |
  Can learn simple patterns            Layer 1: edges
  (spam/not spam)                      Layer 2: shapes
                                       Layer 3: parts (eyes, ears)
                                       Layer 4: faces
                                       Layer 5: specific person

  Each layer learns HIGHER-LEVEL features from the previous layer.
  Early layers = simple patterns. Deeper layers = complex concepts.
```

### Why deep networks are powerful

```
  EXAMPLE: Image recognition (what each layer "sees")

  Original Image: Photo of a cat

  Layer 1 (edges):       Layer 2 (textures):    Layer 3 (parts):
  +---+---+---+          +---+---+---+           +---+---+---+
  | / | | | \ |          | ~ | ^ | ~ |           |ear|eye|   |
  | - | + | | |          | . | . | ^ |           |   |nose|  |
  | \ | | | / |          | ~ | ~ | . |           |   |mouth| |
  +---+---+---+          +---+---+---+           +---+---+---+

  Layer 4 (objects):     Layer 5 (classification):
  +---+---+---+          
  |   cat     |          "This is a cat"  (98.7% confidence)
  |   face    |          "This is a dog"  (0.8% confidence)
  |           |          "This is a bird" (0.5% confidence)
  +---+---+---+          
```

---

## 8. Types of Neural Network Architectures

```
  MAIN ARCHITECTURES:

  CNN (Convolutional Neural Network)     RNN (Recurrent Neural Network)
  Best for: IMAGES                       Best for: SEQUENCES (text, audio)

  [Image] --> [Conv] --> [Conv] --> [FC]     [word1] -> [H] -> [word2] -> [H] -> [word3]
               |           |                              |                 |
           detect       detect                       remembers          remembers
           edges        objects                      context             context

  Used by: Image classification,         Used by: (Mostly replaced by
  object detection, face recognition     Transformers now, but historically
                                         used for translation, speech)


  Transformer
  Best for: EVERYTHING (text, images, audio, video, code)

  [Input tokens] --> [Self-Attention] --> [Feed-Forward] --> [Output]
                          |
                    "Which parts of the
                     input are relevant
                     to each other?"

  Used by: GPT, Claude, Gemini, BERT, Llama, DALL-E, Whisper
  (This is THE architecture of modern AI -- covered in detail next)
```

| Architecture | Year | Best for | Examples |
|-------------|------|----------|----------|
| **CNN** | 1998 (LeNet) | Images, video | ResNet, YOLO, EfficientNet |
| **RNN/LSTM** | 1997 | Sequential data | (largely replaced by Transformers) |
| **Transformer** | 2017 | Everything | GPT, Claude, Gemini, BERT, Llama, DALL-E |
| **Diffusion** | 2020 | Image generation | Stable Diffusion, DALL-E, Midjourney |

---

# Module 3: The Transformer -- The Architecture Behind Modern AI

---

## 9. What is a Transformer?

The **Transformer** is a neural network architecture introduced in the 2017 paper "Attention Is All You Need" by Google researchers. It's the foundation of virtually every modern AI system.

```
  WHY TRANSFORMERS REPLACED RNNs:

  RNN (old way) -- processes words ONE AT A TIME:

  "The cat sat on the mat"
   [1] -> [2] -> [3] -> [4] -> [5] -> [6]
   The    cat    sat    on     the    mat

   Slow: must process sequentially (can't parallelize)
   Forgetful: by word 100, it's forgotten word 1


  TRANSFORMER (new way) -- processes ALL WORDS AT ONCE:

  "The cat sat on the mat"
   [1]   [2]   [3]   [4]   [5]   [6]
    |     |     |     |     |     |
    +--+--+--+--+--+--+--+--+--+--+
    |       SELF-ATTENTION          |
    | Every word looks at EVERY     |
    | other word simultaneously     |
    +--+--+--+--+--+--+--+--+--+--+
    |     |     |     |     |     |

   Fast: processes all words in parallel (GPU-friendly)
   Full context: every word can attend to every other word
```

### The key innovation: Self-Attention

**Self-attention** lets the model decide which parts of the input are relevant to each other.

```
  SELF-ATTENTION EXAMPLE:

  Sentence: "The animal didn't cross the street because it was too tired"

  What does "it" refer to?

  Without attention: ambiguous -- "it" could be the animal or the street
  With self-attention: the model calculates attention scores:

  "it" pays attention to:
    "animal"  -> HIGH score (0.82)   <-- "it" refers to the animal
    "street"  -> low score  (0.05)
    "cross"   -> low score  (0.03)
    "tired"   -> medium     (0.10)   <-- supports "animal" interpretation

  The model learns WHICH words matter for understanding each word.
```

### Transformer architecture (simplified)

```
  THE TRANSFORMER (simplified):

  Input text: "Translate: the cat is black"

  +----------------------------------------------+
  |              ENCODER                          |
  |                                              |
  | [Tokenize] -> [Embed] -> [Position] ->       |
  |                                              |
  | +------------------------------------------+ |
  | | Self-Attention (words attend to each     | |
  | | other to understand context)             | |
  | +------------------------------------------+ |
  | | Feed-Forward (process each position)     | |
  | +------------------------------------------+ |
  |        (repeat N times -- "layers")          |
  +---------------------+------------------------+
                        |
                   encoded meaning
                        |
  +---------------------v------------------------+
  |              DECODER                          |
  |                                              |
  | +------------------------------------------+ |
  | | Masked Self-Attention (can only look at  | |
  | | words generated so far, not future)      | |
  | +------------------------------------------+ |
  | | Cross-Attention (looks at encoder output)| |
  | +------------------------------------------+ |
  | | Feed-Forward                             | |
  | +------------------------------------------+ |
  |        (repeat N times)                      |
  +---------------------+------------------------+
                        |
                        v
  Output: "le chat est noir"
  (generated one token at a time)
```

**GPT-style models** (GPT, Claude, Llama) use only the **decoder** part -- they predict the next token given all previous tokens. This is called a "decoder-only" or "autoregressive" model.

**BERT-style models** use only the **encoder** part -- they understand text bidirectionally but don't generate new text. Used for classification, search, and embeddings.

---

## 10. Tokens and Tokenization

LLMs don't read text character by character or word by word. They break text into **tokens** -- chunks that can be whole words, parts of words, or single characters.

```
  TOKENIZATION EXAMPLE:

  Input:  "ChatGPT is unbelievably good at coding!"

  Tokens: ["Chat", "G", "PT", " is", " un", "believ", "ably",
           " good", " at", " coding", "!"]

  Token count: 11 tokens

  Common patterns:
  - Common words = 1 token:     "the" "is" "good"
  - Uncommon words get split:   "unbelievably" = "un" + "believ" + "ably"
  - Numbers get split:          "2024" = "20" + "24"
  - Code:                       "useState" = "use" + "State"
```

### Why tokens matter

```
  TOKEN LIMITS AND PRICING:

  Every model has a CONTEXT WINDOW (max tokens it can handle):

  +--------------------------------------------------+
  | GPT-4o:          128,000 tokens (~300 pages)     |
  | Claude Opus:   1,000,000 tokens (~2,500 pages)   |
  | Gemini 1.5 Pro:1,000,000 tokens (~2,500 pages)   |
  | Llama 3:         128,000 tokens                   |
  +--------------------------------------------------+

  Context window includes BOTH your input AND the model's output.

  +==========================================+
  | [System prompt] [Your message] [Response]|
  |    500 tokens    2000 tokens   1500 tokens|
  |                                          |
  | Total: 4,000 tokens used of 128K limit   |
  +==========================================+

  API PRICING (pay per token):
  GPT-4o:   $2.50 / 1M input tokens,  $10 / 1M output tokens
  Claude:   $3    / 1M input tokens,  $15 / 1M output tokens

  ~1 token = ~4 characters in English
  ~750 words = ~1000 tokens
```

---

## 11. Embeddings -- How AI Understands Meaning

An **embedding** is a list of numbers (a vector) that represents the meaning of a word, sentence, or document. Words with similar meanings have similar numbers.

```
  EMBEDDINGS -- meaning as numbers:

  "king"    -> [0.2, 0.8, 0.1, 0.9, ...]   (1536 numbers)
  "queen"   -> [0.3, 0.8, 0.2, 0.8, ...]   (similar to king!)
  "banana"  -> [0.9, 0.1, 0.7, 0.2, ...]   (very different)

  In "embedding space" (imagine a 3D map):

              king *     * queen
                    \   /
                     \ /
                      |
                      |  (far away)
                      |
                banana *

  Words with similar meanings are CLOSE together.
  Words with different meanings are FAR apart.


  THE FAMOUS EXAMPLE:

  king - man + woman = queen

  [0.2, 0.8, 0.1, 0.9]     (king)
  - [0.1, 0.9, 0.0, 0.5]   (man)
  + [0.2, 0.7, 0.1, 0.4]   (woman)
  = [0.3, 0.6, 0.2, 0.8]   (closest match: queen!)

  The model learned gender relationships from data alone.
```

### Vector databases

```
  HOW VECTOR SEARCH WORKS:

  Traditional database:                   Vector database:
  "Find rows where                        "Find items SIMILAR to
   name = 'shoes'"                         this meaning"

  SQL:                                    Query: "comfortable footwear"
  SELECT * FROM products                  Embedding: [0.3, 0.7, 0.2, ...]
  WHERE name = 'shoes'                            |
       |                                          v
       v                                    Search by DISTANCE in vector space:
  Exact match only                          - "running shoes"    (distance: 0.1) -- close!
  Misses "sneakers",                        - "hiking boots"     (distance: 0.15) -- close!
  "running shoes", etc.                     - "leather jacket"   (distance: 0.8) -- far

  Examples: Pinecone, Weaviate, Chroma, pgvector
```

---

## 12. How LLMs Are Trained

Training a Large Language Model happens in stages:

```
  THE THREE STAGES OF LLM TRAINING:

  Stage 1: PRE-TRAINING                  Stage 2: FINE-TUNING
  (learn language from the internet)      (learn to follow instructions)

  +---------------------------+           +---------------------------+
  | Training data:            |           | Training data:            |
  | - Wikipedia               |           | - Human-written Q&A pairs |
  | - Books                   |           | - Instruction examples    |
  | - Websites                |           | - "Be helpful, harmless"  |
  | - Code (GitHub)           |           |                           |
  | - Research papers         |           | "What is the capital      |
  |                           |           |  of France?"              |
  | Trillions of tokens       |           | "The capital of France    |
  | Months of training        |           |  is Paris."               |
  | Millions of dollars       |           +---------------------------+
  +---------------------------+                      |
             |                                       v
             v                              Model learns to be a
  Model learns language,                   helpful assistant
  facts, reasoning, code                   (not just predict text)
  (but has no "personality"
   -- just predicts next word)

  Stage 3: RLHF (Reinforcement Learning from Human Feedback)

  +--------------------------------------------------+
  | 1. Model generates multiple responses             |
  | 2. Human raters rank them (best to worst)         |
  | 3. A "reward model" learns human preferences      |
  | 4. The LLM is trained to maximize the reward      |
  +--------------------------------------------------+
             |
             v
  Model becomes helpful, harmless, and honest
  (this is what makes ChatGPT feel "aligned")
```

### Pre-training: next token prediction

The core of LLM training is surprisingly simple: **predict the next word**.

```
  NEXT TOKEN PREDICTION:

  Training text: "The cat sat on the"

  Model sees:     "The cat sat on the"
  Model predicts: "mat" (probability: 0.23)
                  "floor" (probability: 0.18)
                  "chair" (probability: 0.12)
                  "dog" (probability: 0.01)

  Actual next word: "mat"
  Loss: -log(0.23)  (model was somewhat right, small loss)

  If model predicted "mat" with probability 0.01:
  Loss: -log(0.01)  (model was very wrong, big loss)

  Repeat for TRILLIONS of tokens.
  The model gradually learns grammar, facts, reasoning,
  common sense -- all from predicting the next word.
```

### Training costs

| Model | Training cost (estimated) | Training time | GPUs used |
|-------|--------------------------|--------------|-----------|
| GPT-3 (2020) | ~$5 million | Months | Thousands of V100s |
| GPT-4 (2023) | ~$100+ million | ~6 months | Tens of thousands of A100s |
| Llama 3 405B (2024) | ~$50+ million | Months | 16,000+ H100s |
| Claude, Gemini | Not disclosed | Months | Massive GPU clusters |

---

## 13. How LLMs Generate Text (Inference)

When you send a prompt to ChatGPT or Claude, here's what happens:

```
  GENERATION (Inference) -- one token at a time:

  Your prompt: "Write a haiku about coding"

  Step 1: Model sees "Write a haiku about coding"
          Predicts next token: "Lines"

  Step 2: Model sees "Write a haiku about coding Lines"
          Predicts next token: " of"

  Step 3: Model sees "Write a haiku about coding Lines of"
          Predicts next token: " code"

  Step 4: Model sees "Write a haiku about coding Lines of code"
          Predicts next token: " flow"

  ... continues until it generates a stop token or hits the limit.

  Final output: "Lines of code flow
                 Through logic's winding pathways
                 Bugs hide in the dark"

  Each token is generated ONE AT A TIME.
  This is why responses "stream" in letter by letter.
```

### Temperature -- controlling randomness

```
  TEMPERATURE SETTING:

  Model predicts next word after "The cat sat on the":
    "mat"   -> 0.40
    "floor" -> 0.25
    "table" -> 0.15
    "moon"  -> 0.01

  Temperature = 0 (deterministic):        Temperature = 1.0 (creative):
  ALWAYS picks "mat" (highest prob)        Samples from distribution:
  Same input = same output every time      sometimes "mat", sometimes "floor"
                                           rarely "moon"

  Temperature = 0.0     --> factual, consistent, boring
  Temperature = 0.3-0.7 --> balanced (good for most tasks)
  Temperature = 1.0+    --> creative, surprising, may hallucinate

  Low temp = exam answers     High temp = poetry
```

---

## 14. Parameters -- What Makes a Model "Large"

When people say "GPT-4 has trillions of parameters" or "Llama 3 has 70 billion parameters" -- what does that mean?

```
  PARAMETERS = the learned weights and biases in the neural network

  A parameter is a single number that was learned during training.

  Small model (7B):
  7,000,000,000 numbers that encode everything the model knows
  about language, facts, reasoning, and code.

  Large model (70B):
  70,000,000,000 numbers. More parameters = can store more knowledge
  and nuance, but needs more compute to run.

  MODEL SIZE COMPARISON:

  +--------+----------+-------------+------------------+
  | Model  | Params   | RAM needed  | Quality          |
  +--------+----------+-------------+------------------+
  | Tiny   | 1B       | ~2 GB       | Basic tasks      |
  | Small  | 7B       | ~14 GB      | Good for simple  |
  | Medium | 13B      | ~26 GB      | Solid all-around |
  | Large  | 70B      | ~140 GB     | Very capable     |
  | Huge   | 405B+    | ~800 GB     | Frontier quality |
  +--------+----------+-------------+------------------+

  Rule of thumb: each parameter = ~2 bytes (FP16)
  70B model = ~140 GB of GPU memory just to LOAD it
```

---

# Module 4: Prompt Engineering & Using AI

---

## 15. Prompt Engineering

**Prompt engineering** is the skill of crafting inputs to get the best outputs from an LLM. The same question asked differently can produce vastly different results.

### Techniques

```
  ZERO-SHOT (just ask):
  +----------------------------------------------+
  | "Is this review positive or negative?         |
  |  'The food was amazing but service was slow'" |
  +----------------------------------------------+
  Model uses its training to answer.
  Works for simple tasks.


  FEW-SHOT (give examples first):
  +----------------------------------------------+
  | "Classify these reviews:                      |
  |                                              |
  |  'Loved it!' -> Positive                     |
  |  'Terrible experience' -> Negative           |
  |  'It was okay' -> Neutral                    |
  |                                              |
  |  'The food was amazing but service was slow'" |
  +----------------------------------------------+
  Model learns the pattern from examples.
  Much better for specific tasks.


  CHAIN OF THOUGHT (think step by step):
  +----------------------------------------------+
  | "Solve this step by step:                     |
  |  If a shirt costs $25 and is 20% off,        |
  |  how much do you pay?"                        |
  |                                              |
  | Model: "Step 1: 20% of $25 = $5              |
  |         Step 2: $25 - $5 = $20               |
  |         Answer: $20"                          |
  +----------------------------------------------+
  Forcing the model to "show its work" dramatically
  improves accuracy on reasoning tasks.


  SYSTEM PROMPT (set behavior):
  +----------------------------------------------+
  | System: "You are a senior React developer.    |
  |  Always use TypeScript. Explain your code.    |
  |  Follow best practices."                      |
  |                                              |
  | User: "Build me a todo app"                   |
  +----------------------------------------------+
  System prompts define the model's persona,
  constraints, and output format.
```

### Tips for better prompts

| Technique | Bad prompt | Good prompt |
|-----------|-----------|-------------|
| **Be specific** | "Write code" | "Write a TypeScript function that validates email addresses using regex" |
| **Give context** | "Fix this bug" | "This React component re-renders infinitely. Here's the code: ..." |
| **Define format** | "List some ideas" | "List 5 ideas in bullet points, each under 20 words" |
| **Set constraints** | "Write an essay" | "Write a 200-word essay for a 10th grader about climate change" |

---

## 16. Hallucinations -- When AI Makes Things Up

**Hallucinations** are confident-sounding but factually wrong outputs. The model doesn't "know" facts -- it predicts probable-sounding text.

```
  WHY HALLUCINATIONS HAPPEN:

  Model's job: predict the most PROBABLE next token
  NOT: look up the CORRECT answer

  Prompt: "Who won the 2027 Cricket World Cup?"

  Model thinks: "This looks like a sports question.
   Based on patterns in my training data,
   a probable answer would be..."

  Model outputs: "India won the 2027 Cricket World Cup,
   defeating Australia in the final."

  Problem: The model has no training data from 2027.
   It GENERATED a plausible-sounding answer.
   It sounds confident because confidence is
   what got rewarded during training.

  HALLUCINATION != lying (no intent to deceive)
  HALLUCINATION = pattern matching without verification
```

### How to reduce hallucinations

1. **Ask for sources** -- "Cite your sources" (model may still hallucinate sources)
2. **RAG** -- Give the model real data to reference (see next section)
3. **Temperature 0** -- Reduces creativity/randomness
4. **Verify** -- Always fact-check important claims
5. **Constrain output** -- "Only answer based on the document I provided"

---

## 17. RAG -- Retrieval Augmented Generation

**RAG** solves the hallucination problem by giving the model real, up-to-date data to reference.

```
  WITHOUT RAG:                            WITH RAG:

  User: "What is our                      User: "What is our
   refund policy?"                          refund policy?"
        |                                       |
        v                                       v
  +----------+                            +-----------+
  | LLM      |                            | 1. Search |
  | (guesses  |                           |    vector  |
  |  based on |                           |    database|
  |  training)|                           +-----+-----+
  +----------+                                  |
        |                                  Retrieved: "Refund policy:
        v                                  Full refund within 30 days.
  "Our refund policy                       After 30 days, store credit
   is typically 14 days..."               only. Contact support@..."
  (HALLUCINATED -- wrong!)                      |
                                                v
                                          +-----------+
                                          | 2. LLM    |
                                          | answers   |
                                          | USING the |
                                          | retrieved |
                                          | document  |
                                          +-----+-----+
                                                |
                                                v
                                          "According to our policy,
                                           you can get a full refund
                                           within 30 days..."
                                          (ACCURATE -- grounded in real data)
```

### How RAG works step by step

```
  RAG PIPELINE:

  SETUP (one time):
  +----------+     +----------+     +----------------+
  | Your     | --> | Split    | --> | Create         |
  | documents|     | into     |     | embeddings     |
  | (PDFs,   |     | chunks   |     | for each chunk |
  | docs,    |     | (500     |     | and store in   |
  | website) |     |  words)  |     | vector DB      |
  +----------+     +----------+     +----------------+

  QUERY TIME (every question):
  +----------+     +----------+     +----------+     +----------+
  | User     | --> | Embed    | --> | Search   | --> | Send to  |
  | question |     | the      |     | vector   |     | LLM with |
  |          |     | question |     | DB for   |     | retrieved |
  |          |     |          |     | similar  |     | context  |
  +----------+     +----------+     | chunks   |     +----------+
                                    +----------+          |
                                                          v
                                                    Answer grounded
                                                    in YOUR data
```

**Examples:** Customer support bots, internal knowledge bases, legal document search, medical diagnosis assistants.

---

## 18. AI Agents

An **AI agent** is an LLM that can take **actions** -- not just generate text, but actually DO things like search the web, run code, call APIs, and make decisions.

```
  REGULAR LLM:                            AI AGENT:

  User: "What's the weather              User: "What's the weather
   in Mumbai?"                             in Mumbai?"
        |                                       |
        v                                       v
  +----------+                            +----------+
  | LLM      |                            | Agent    |
  | "Based   |                            | thinks:  |
  |  on my   |                            | "I need  |
  |  training|                            |  real-   |
  |  data..."|                            |  time    |
  +----------+                            |  data"   |
        |                                 +----+-----+
        v                                      |
  "Mumbai is generally                    +----v-----------+
   hot and humid..."                      | Tool: Weather  |
  (generic, may be wrong)                | API call to     |
                                          | api.weather.com |
                                          +----+-----------+
                                               |
                                          +----v-----+
                                          | Agent:   |
                                          | "It's    |
                                          | 34C and |
                                          | humid in |
                                          | Mumbai   |
                                          | right now"|
                                          +----------+
                                          (REAL-TIME, accurate)
```

### Function calling / Tool use

```
  HOW FUNCTION CALLING WORKS:

  You define tools the model can use:

  Available tools:
  +------------------+  +------------------+  +------------------+
  | get_weather()    |  | search_web()     |  | run_code()       |
  | - city: string   |  | - query: string  |  | - code: string   |
  +------------------+  +------------------+  +------------------+

  User: "What's 2+2 and what's the weather in Delhi?"

  Agent thinks:
    1. "2+2 is simple math, I know this: 4"
    2. "Weather needs real data, let me use a tool"
       -> calls get_weather(city="Delhi")
       -> gets back: { temp: 38, condition: "sunny" }
    3. Combines: "2+2 is 4, and it's currently 38C
       and sunny in Delhi."

  The model DECIDES which tools to use and when.
  This is what makes Claude Code, ChatGPT plugins,
  and AI coding assistants possible.
```

### Agent loop

```
  THE AGENT LOOP:

  +--------+     +---------+     +-------+     +---------+
  | Think  | --> | Decide  | --> | Act   | --> | Observe |
  | about  |     | what to |     | (use  |     | result  |---+
  | task   |     | do next |     | tool) |     |         |   |
  +--------+     +---------+     +-------+     +---------+   |
       ^                                                      |
       |                                                      |
       +---------- Loop until task is complete ---------------+

  Example: "Find the cheapest flight from Delhi to Goa next week"

  Think: "I need flight data. Let me search."
  Act:   search_flights(from="DEL", to="GOI", date="next week")
  Observe: [list of flights with prices]
  Think: "Let me find the cheapest."
  Act:   sort by price
  Observe: "IndiGo, April 8, Rs 3,200"
  Output: "The cheapest flight is IndiGo on April 8 at Rs 3,200"
```

---

# Module 5: Image Generation & Multimodal AI

---

## 19. How Image Generation Works (Diffusion Models)

Most modern image generators (DALL-E, Midjourney, Stable Diffusion) use **diffusion models**.

```
  DIFFUSION -- learning to remove noise:

  TRAINING (forward process -- add noise):

  Clear image     Slightly noisy    Very noisy       Pure noise
  +----------+    +----------+     +----------+     +----------+
  |  [cat]   | -> | [cat~~]  | ->  | [~~~~]   | ->  | [static] |
  +----------+    +----------+     +----------+     +----------+
  Step 0          Step 100          Step 500          Step 1000

  The model learns: "What does each noise level look like?"


  GENERATION (reverse process -- remove noise):

  Pure noise      Less noisy        Almost clear     Clear image!
  +----------+    +----------+     +----------+     +----------+
  | [static] | -> | [~~~~]   | ->  | [cat~~]  | ->  | [cat!]   |
  +----------+    +----------+     +----------+     +----------+
  Step 1000       Step 500          Step 100          Step 0

  Start with random noise.
  Gradually remove noise, guided by your text prompt.
  The model learned "what removing noise looks like"
  during training, so it can generate NEW images.
```

### Text-to-image pipeline

```
  FROM PROMPT TO IMAGE:

  "A cat wearing sunglasses on a beach, digital art"
        |
        v
  +----------------+
  | Text Encoder   |  (converts text to embeddings --
  | (CLIP model)   |   numbers that capture meaning)
  +-------+--------+
          |
          v
  +-------+--------+
  | Diffusion      |  (starts from noise, gradually
  | Model          |   "denoises" guided by the
  | (U-Net)        |   text embeddings)
  +-------+--------+
          |
          v
  +-------+--------+
  | Image Decoder  |  (converts from latent space
  | (VAE)          |   to actual pixels)
  +-------+--------+
          |
          v
  [Generated image of a cat with sunglasses on a beach]
```

### Major image generation models

| Model | Company | Type | Access |
|-------|---------|------|--------|
| **DALL-E 3** | OpenAI | Closed | API, ChatGPT |
| **Midjourney** | Midjourney | Closed | Discord bot |
| **Stable Diffusion** | Stability AI | Open source | Run locally |
| **Imagen 3** | Google | Closed | Gemini |
| **Flux** | Black Forest Labs | Open source | Run locally |

---

## 20. Multimodal AI

**Multimodal** models can process and generate multiple types of data -- text, images, audio, video.

```
  EVOLUTION OF AI MODELS:

  2020: Text only          2023: Text + Image         2024+: Everything
  +----------+             +----------+               +----------+
  | GPT-3    |             | GPT-4V   |               | GPT-4o   |
  |          |             | Claude 3 |               | Gemini   |
  | Text     |             |          |               |          |
  | in/out   |             | Text in  |               | Text     |
  |          |             | Image in |               | Image    |
  +----------+             | Text out |               | Audio    |
                           +----------+               | Video    |
                                                      | Code     |
                                                      | in/out   |
                                                      +----------+

  GPT-4o can:
  - See images and describe them
  - Listen to audio and respond with voice
  - Read code and explain it
  - Generate images from text

  This is why it's called "omni" (everything)
```

---

# Module 6: The AI Industry

---

## 21. Key Companies and Models

```
  THE AI LANDSCAPE (2024-2025):

  CLOSED SOURCE (API access only):

  +------------------+  +------------------+  +------------------+
  | OpenAI           |  | Anthropic        |  | Google DeepMind  |
  | GPT-4o, o1, o3   |  | Claude 4         |  | Gemini 2.5      |
  | DALL-E 3         |  | Claude Opus/     |  | Imagen 3         |
  | Whisper (open)   |  |  Sonnet/Haiku    |  | AlphaFold        |
  | ChatGPT          |  | claude.ai        |  | Bard/Gemini app  |
  +------------------+  +------------------+  +------------------+

  OPEN SOURCE (download and run yourself):

  +------------------+  +------------------+  +------------------+
  | Meta             |  | Mistral          |  | Others           |
  | Llama 3 (8B,    |  | Mistral Large    |  | Qwen (Alibaba)   |
  |  70B, 405B)      |  | Mixtral (MoE)    |  | DeepSeek         |
  | Free to use      |  | Open weights     |  | Phi (Microsoft)  |
  +------------------+  +------------------+  +------------------+
```

### Closed vs Open source

| | Closed source | Open source |
|---|---|---|
| **Examples** | GPT-4, Claude, Gemini | Llama 3, Mistral, DeepSeek |
| **Can you see the code?** | No | Yes (weights are downloadable) |
| **Can you run it locally?** | No (API only) | Yes (with enough GPU) |
| **Privacy** | Your data goes to their servers | Data stays on your machine |
| **Cost** | Pay per token | Free (but need hardware) |
| **Quality** | Generally best (more $$ for training) | Catching up fast |
| **Customization** | Limited (fine-tuning APIs) | Full control |

---

## 22. GPUs -- Why Hardware Matters

AI runs on **GPUs** (Graphics Processing Units), not CPUs. Here's why:

```
  CPU vs GPU:

  CPU (Central Processing Unit):          GPU (Graphics Processing Unit):
  +--+--+--+--+                           +--+--+--+--+--+--+--+--+
  |C1|C2|C3|C4|  4-16 powerful cores      |c |c |c |c |c |c |c |c |
  +--+--+--+--+                           +--+--+--+--+--+--+--+--+
                                          |c |c |c |c |c |c |c |c |
  Each core: VERY fast                    +--+--+--+--+--+--+--+--+
  at complex tasks                        |c |c |c |c |c |c |c |c |
                                          +--+--+--+--+--+--+--+--+
  Good for: running your OS,              ... thousands of small cores
  web browsing, single tasks
                                          Each core: slower individually
                                          But THOUSANDS working together

                                          Good for: matrix multiplication
                                          (which is ALL neural networks do)

  Analogy:
  CPU = 4 math professors solving problems one at a time (fast but few)
  GPU = 10,000 students each doing simple multiplication (slow but parallel)

  Neural networks = billions of multiplications.
  GPU wins massively.
```

### The NVIDIA dominance

```
  NVIDIA GPU TIMELINE FOR AI:

  2012: GTX 680           Used for early deep learning experiments
  2016: P100              First "AI-specific" datacenter GPU
  2020: A100              The workhorse that trained GPT-3
  2023: H100              80GB, trained GPT-4 and most current models
  2024: H200              141GB memory, 2x H100 performance
  2024: B200 (Blackwell)  Next generation, even faster

  NVIDIA has ~80-90% market share in AI training GPUs.
  This is why NVIDIA became a $3 trillion company.

  H100 GPU:
  - Price: ~$30,000-40,000 each
  - Training GPT-4 required: ~25,000 of them
  - That's ~$750 million in GPUs alone (not counting electricity)
```

### Alternatives to NVIDIA

| Company | Chip | Used by |
|---------|------|---------|
| **NVIDIA** | H100, H200, B200 | Everyone |
| **AMD** | MI300X | Some datacenters |
| **Google** | TPU v5 | Google's own models (Gemini) |
| **Amazon** | Trainium | AWS customers |
| **Apple** | M-series (Neural Engine) | On-device AI (Siri, photos) |

---

## 23. How Developers Use AI (APIs)

```
  HOW YOUR APP TALKS TO AN LLM:

  Your React App               API Request              AI Provider
  +-------------+             +----------------+        +-------------+
  |             |  -- POST -> | {              |  --->   | OpenAI /    |
  | User types  |             |   "model":     |        | Anthropic / |
  | a prompt    |             |   "gpt-4o",    |        | Google      |
  |             |             |   "messages":  |        |             |
  +-------------+             |   [{role:      |        | Runs model  |
                              |     "user",    |        | on GPU      |
  Your App                    |     content:   |        | cluster     |
  +-------------+             |     "Hello"}]  |        +------+------+
  |             | <-- JSON -- | }              |  <---         |
  | Shows       |             +----------------+        Response JSON
  | response    |                                       with generated
  | to user     |                                       text
  +-------------+

  This is EXACTLY how our AI Component Builder works!
  (See App.tsx -- the openai.chat.completions.create() call)
```

### Key API concepts

| Term | What it means |
|------|--------------|
| **System message** | Sets the AI's behavior ("You are a helpful coding assistant") |
| **User message** | The human's input |
| **Assistant message** | The AI's response |
| **Temperature** | Controls randomness (0 = deterministic, 1 = creative) |
| **Max tokens** | Maximum length of the response |
| **Streaming** | Receive tokens as they're generated (not all at once) |
| **Context window** | Total tokens the model can handle (input + output) |
| **Rate limiting** | Maximum requests per minute your API key allows |

---

# Module 7: Advanced Topics

---

## 24. Fine-Tuning vs RAG vs Prompting

Three ways to customize AI behavior:

```
  WHEN TO USE WHAT:

  +----------------------------------------------------------+
  |                                                          |
  |  PROMPTING (easiest)                                     |
  |  "You are a customer support agent for Acme Corp..."     |
  |                                                          |
  |  Use when: you need quick customization                  |
  |  Cost: $0 (just words)                                   |
  |  Limitation: context window limit                        |
  |                                                          |
  +----------------------------------------------------------+
  |                                                          |
  |  RAG (medium effort)                                     |
  |  Feed the model your documents at query time             |
  |                                                          |
  |  Use when: model needs access to YOUR data               |
  |  Cost: vector DB hosting + more tokens per query         |
  |  Limitation: retrieval quality depends on embeddings     |
  |                                                          |
  +----------------------------------------------------------+
  |                                                          |
  |  FINE-TUNING (most effort)                               |
  |  Retrain the model on your specific data                 |
  |                                                          |
  |  Use when: you need the model to BEHAVE differently      |
  |  (specific tone, format, domain expertise)               |
  |  Cost: $100-10,000+ for training                         |
  |  Limitation: needs labeled training data                 |
  |                                                          |
  +----------------------------------------------------------+

  Decision tree:
  Need specific data? -> RAG
  Need specific behavior/style? -> Fine-tune
  Need quick customization? -> Prompt engineering
```

---

## 25. AI Safety and Alignment

**Alignment** = making sure AI systems do what humans actually want, not just what they're literally told.

```
  THE ALIGNMENT PROBLEM:

  What you said:           What you meant:          What AI did:
  "Make the house          "Clean up the house      AI hires a demolition
   spotless"                and make it tidy"        crew and rebuilds from
                                                     scratch (technically
                                                     spotless!)

  "Maximize                "Get reasonable          AI signs every human up
   user engagement"         engagement"              for addictive infinite
                                                     scrolling

  "Cure cancer"            "Cure cancer safely      AI experiments on humans
                            and ethically"           without consent
                                                     (most efficient path)

  The problem: AI optimizes for EXACTLY what you measure,
  not what you actually want. This is called "reward hacking"
  or "specification gaming."
```

### Key safety concepts

| Concept | What it means |
|---------|--------------|
| **Alignment** | Making AI do what humans actually want |
| **RLHF** | Training AI to prefer outputs humans rate highly |
| **Constitutional AI** | Anthropic's approach -- AI follows a set of principles |
| **Jailbreaking** | Tricking AI into ignoring safety guidelines |
| **Red teaming** | Deliberately trying to make AI behave badly (to find and fix vulnerabilities) |
| **Guardrails** | Rules that prevent harmful outputs |
| **Hallucination** | AI generating false but confident-sounding information |
| **Existential risk** | Concern that superintelligent AI could pose risks to humanity |

### Why Anthropic was founded

Anthropic was founded in 2021 by former OpenAI researchers (Dario and Daniela Amodei) who wanted to focus on AI safety. Their approach: **Constitutional AI** -- training Claude to follow a set of principles about being helpful, harmless, and honest, rather than relying solely on human raters.

---

## 26. The Compute Scaling Debate

```
  SCALING LAWS -- bigger model + more data = better performance:

  Performance
  (quality)
     ^
     |                                          * (GPT-4)
     |                                    *
     |                              *
     |                       *
     |                *
     |          *
     |     *
     | *
     +----------------------------------------->
                  Model size (parameters)
                  + Training data
                  + Compute (GPU hours)

  Key finding: performance improves PREDICTABLY
  with more compute. This is why companies are
  spending billions on bigger models.

  BUT: There's a debate:

  Camp 1: "Keep scaling"          Camp 2: "We need new ideas"
  - Just make models bigger       - Bigger models = diminishing returns
  - More data, more GPUs          - Need architectural breakthroughs
  - OpenAI, Anthropic approach    - Efficiency matters (Mistral, DeepSeek)

  The truth is probably both: scale PLUS new techniques.
```

---

## 27. Local AI vs Cloud AI

```
  CLOUD AI (API):                         LOCAL AI (on your machine):

  Your device                             Your device
  +----------+                            +------------------+
  | Send     | ---internet--->            | Model runs HERE  |
  | request  |               |            | (on your GPU)    |
  +----------+     +---------+-------+    +------------------+
                   | Cloud GPU       |
  +----------+     | cluster         |    Pros:
  | Receive  | <---|                 |    + Complete privacy
  | response |     | Runs model      |    + No internet needed
  +----------+     +-----------------+    + No API costs
                                          + Full control
  Pros:
  + Best models available                 Cons:
  + No hardware needed                    - Need expensive GPU
  + Always up to date                     - Smaller models (7B-70B)
  + Easy to start                         - You manage everything
                                          - Lower quality than
  Cons:                                     frontier models
  - Data goes to their servers
  - Costs per token
  - Internet required
  - Rate limits

  TOOLS FOR LOCAL AI:
  - Ollama (easiest -- run LLMs with one command)
  - llama.cpp (C++ inference, very efficient)
  - vLLM (high-throughput serving)
  - LM Studio (GUI for running local models)
```

---

# Module 8: Essential Vocabulary

---

## 28. Terms You Must Know

| Term | Definition |
|------|-----------|
| **AI** | Artificial Intelligence -- machines that perform tasks requiring human-like intelligence |
| **ML** | Machine Learning -- subset of AI where machines learn from data |
| **Deep Learning** | ML using neural networks with many layers |
| **Neural Network** | Computing system inspired by the brain, made of connected layers of neurons |
| **Neuron** | A single unit that receives inputs, applies weights, and produces an output |
| **Weight** | A learned number that determines how important an input is |
| **Bias** | A learned offset that adjusts a neuron's activation threshold |
| **Layer** | A group of neurons at the same depth in the network |
| **Activation function** | Function that adds non-linearity (ReLU, sigmoid, softmax) |
| **Backpropagation** | Algorithm for calculating how to adjust weights to reduce error |
| **Loss function** | Measures how wrong the model's prediction is |
| **Epoch** | One complete pass through the training data |
| **Batch size** | Number of examples processed at once |
| **Learning rate** | How big of a step to take when updating weights |
| **Overfitting** | Model memorizes training data, fails on new data |
| **Transformer** | THE neural network architecture behind modern AI (2017) |
| **Self-attention** | Mechanism that lets each token "look at" every other token |
| **Token** | A chunk of text (word, sub-word, or character) that the model processes |
| **Tokenizer** | Algorithm that splits text into tokens |
| **Embedding** | A list of numbers representing the meaning of a word/sentence |
| **Vector** | A list of numbers (embeddings are vectors) |
| **Vector database** | Database optimized for searching by similarity (Pinecone, Chroma) |
| **LLM** | Large Language Model -- a transformer trained on massive text data |
| **Parameter** | A single learned value (weight or bias) in the model |
| **Context window** | Maximum number of tokens the model can process at once |
| **Inference** | Running a trained model to get predictions (vs. training) |
| **Latency** | Time between sending a request and getting a response |
| **Throughput** | Number of requests processed per second |
| **Temperature** | Controls randomness in output (0 = deterministic, 1 = creative) |
| **Top-p (nucleus sampling)** | Only consider tokens whose cumulative probability exceeds p |
| **Prompt** | Input text sent to the model |
| **System prompt** | Instructions that set the model's behavior |
| **Few-shot** | Giving examples in the prompt to guide output |
| **Zero-shot** | Asking the model without examples |
| **Chain of thought** | Prompting the model to reason step by step |
| **Fine-tuning** | Further training a pre-trained model on specific data |
| **RLHF** | Reinforcement Learning from Human Feedback -- aligning models with human preferences |
| **RAG** | Retrieval Augmented Generation -- giving models external data at query time |
| **Hallucination** | Model generating confident but factually wrong information |
| **Grounding** | Connecting model outputs to verified data sources |
| **Agent** | An LLM that can take actions (search, call APIs, run code) |
| **Function calling** | LLM deciding to call a specific tool/function with arguments |
| **Tool use** | Same as function calling |
| **MCP** | Model Context Protocol -- standard for connecting AI to external tools |
| **Streaming** | Receiving model output token by token as it's generated |
| **Diffusion model** | Image generation by learning to remove noise |
| **GAN** | Generative Adversarial Network -- older image generation approach |
| **VAE** | Variational Autoencoder -- compresses data to latent space |
| **CLIP** | Model that understands both text and images (used in image generation) |
| **Multimodal** | Model that handles multiple data types (text + image + audio) |
| **GPU** | Graphics Processing Unit -- hardware that runs AI workloads |
| **TPU** | Tensor Processing Unit -- Google's custom AI chip |
| **FLOPS** | Floating Point Operations Per Second -- measure of compute power |
| **Quantization** | Reducing model precision (FP32 -> FP16 -> INT8) to run on less hardware |
| **LoRA** | Low-Rank Adaptation -- efficient fine-tuning that only trains a small number of parameters |
| **MoE** | Mixture of Experts -- architecture where only some parameters activate per input (Mixtral) |
| **Distillation** | Training a small model to mimic a large model |
| **Benchmark** | Standardized test to compare model performance (MMLU, HumanEval) |
| **Open weights** | Model weights are downloadable (Llama, Mistral) |
| **Closed source** | Model only accessible via API (GPT-4, Claude) |
| **Alignment** | Making AI systems do what humans actually want |
| **Guardrails** | Safety constraints preventing harmful outputs |
| **Jailbreak** | Tricking AI into bypassing safety guidelines |
| **Red teaming** | Testing AI by trying to make it fail |
| **AGI** | Artificial General Intelligence -- human-level AI (doesn't exist yet) |
| **Scaling laws** | Performance improves predictably with more data, compute, and parameters |

---

## Key Concepts Recap

- **AI > ML > Deep Learning** -- Each is a subset of the one above. Deep learning (neural networks with many layers) is what powers all modern AI.

- **Transformer** -- The 2017 architecture that changed everything. Uses self-attention to process all tokens in parallel. Foundation of GPT, Claude, Gemini, BERT, and Llama.

- **Training** -- LLMs learn by predicting the next token on trillions of words. Pre-training gives language ability, fine-tuning adds instruction following, RLHF adds alignment.

- **Tokens** -- The unit AI thinks in. Not words, not characters, but sub-word chunks. Every model has a context window (max tokens) and pricing is per token.

- **Embeddings** -- Numbers that capture meaning. Similar concepts are close in vector space. This is how search, RAG, and recommendation systems work.

- **Hallucination** -- AI doesn't "know" facts. It predicts probable text. RAG (giving it real data) and verification are the main defenses.

- **Agents** -- LLMs that can take actions via function calling. The future of AI is not just chat, but autonomous systems that can search, code, and interact with the world.

- **GPUs** -- AI runs on GPUs because neural networks are fundamentally matrix multiplication, which GPUs do thousands of times faster than CPUs. NVIDIA dominates.

- **Safety** -- The biggest challenge isn't making AI smarter, it's making it aligned with human values. RLHF, Constitutional AI, and red teaming are active research areas.

---

## What's Next

The AI field moves faster than any technology in history. By the time you read these notes, there may be new models and techniques. What won't change:

- **Transformers** will remain the foundation (even if improved)
- **Scale** will continue to matter (more data, more compute)
- **Understanding fundamentals** (how attention works, what tokens are, why training matters) will let you understand every new development

If you want to go deeper:
- **3Blue1Brown's neural network series** on YouTube -- the best visual explanations
- **Andrej Karpathy's "Let's build GPT"** -- build a transformer from scratch
- **fast.ai** -- free course on practical deep learning
- **Anthropic's research blog** -- papers on alignment and safety
- **Build with APIs** -- the best way to learn is to build (like our AI Component Builder!)
