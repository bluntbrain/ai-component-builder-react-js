# Class 12: Blockchain, Crypto & Smart Contracts -- The Complete Knowledge Guide

## What You'll Learn

- What blockchain actually is and why it was invented
- How hashing, immutability, and P2P networks make blockchain secure
- How mining and consensus mechanisms work (Proof of Work vs Proof of Stake)
- Everything about Bitcoin -- monetary policy, wallets, transactions, UTXOs, mempool
- Ethereum -- smart contracts, EVM, gas, accounts, DAOs, forks
- Solana -- why it's fast, Proof of History, and how it compares
- How tokens are created (ERC-20, ERC-721/NFTs, SPL tokens)
- Enough vocabulary and understanding to discuss blockchain confidently with anyone

## Who This Is For

You don't need any coding knowledge for this class. This is a **knowledge class** -- by the end, you should be able to explain blockchain, crypto, and smart contracts to a friend, hold your own in a conversation with a crypto developer, and understand what you're reading when you see blockchain news.

---

# Module 1: Blockchain Fundamentals

---

## 1. What is Blockchain?

A blockchain is a **chain of blocks**, where each block contains data, and every block is linked to the previous one using cryptography.

Think of it like a notebook where:
- Each page is a "block"
- Each page has a stamp (hash) at the bottom that's calculated from everything written on that page
- Each page also copies the stamp from the previous page
- If anyone changes a word on page 5, its stamp changes, which breaks the link to page 6, which breaks page 7... and so on

This makes the notebook **tamper-evident** -- you can't change the past without breaking everything that comes after.

```
     Page 1              Page 2              Page 3              Page 4
  +-----------+       +-----------+       +-----------+       +-----------+
  | Data...   |       | Data...   |       | Data...   |       | Data...   |
  |           |       |           |       |           |       |           |
  | Prev: 000 |       | Prev: #A  |       | Prev: #B  |       | Prev: #C  |
  | Stamp: #A |--+--->| Stamp: #B |--+--->| Stamp: #C |--+--->| Stamp: #D |
  +-----------+  |    +-----------+  |    +-----------+  |    +-----------+
                 |                   |                   |
           link (hash)         link (hash)         link (hash)

  Change anything on Page 2?
  -> Stamp #B changes -> Page 3's "Prev" no longer matches -> CHAIN BROKEN
```

### The technical definition

A blockchain is a **distributed, decentralized, immutable ledger** that records transactions across many computers.

Let's break each word down:

| Term | What it means |
|------|--------------|
| **Distributed** | Copies exist on thousands of computers worldwide, not one server |
| **Decentralized** | No single company or person controls it |
| **Immutable** | Once data is written, it cannot be changed or deleted |
| **Ledger** | A record of transactions (who sent what to whom) |

### What problem does it solve?

Before blockchain, every digital transaction needed a **trusted middleman**: a bank, PayPal, a government.

```
  BEFORE BLOCKCHAIN (with middleman):

  Alice ----> [ Bank / PayPal / Govt ] ----> Bob
                      |
               - Charges fees
               - Can censor
               - Single point of failure
               - Can deny service


  AFTER BLOCKCHAIN (peer-to-peer):

  Alice ---------> Bob
         |
    Verified by thousands
    of nodes worldwide
    (no single entity controls it)
```

These middlemen can:
- Charge fees
- Censor transactions
- Get hacked (single point of failure)
- Deny service to people in certain countries

Blockchain removes the middleman. Two strangers on the internet can transfer value directly, and the network itself guarantees the transaction is valid.

---

## 2. Why Study Blockchain?

- **$2+ trillion market cap** (as of 2024) -- this isn't a fad
- Jobs in blockchain development pay 30-50% more than equivalent web dev roles
- Every major bank (JP Morgan, Goldman Sachs), tech company (Google, Microsoft), and government is investing in blockchain
- Understanding blockchain is like understanding the internet in 1995 -- early knowledge compounds massively
- Even if you never write smart contracts, understanding the technology makes you a better engineer and more informed citizen

---

## 3. Applications of Blockchain

Blockchain isn't just cryptocurrency. Here are real applications:

| Application | Example | How blockchain helps |
|-------------|---------|---------------------|
| **Payments** | Bitcoin, UPI-like transfers without banks | No middleman, works across borders |
| **DeFi (Decentralized Finance)** | Uniswap, Aave | Lending, borrowing, trading without banks |
| **NFTs** | Digital art, gaming items | Proof of ownership for digital assets |
| **Supply chain** | Walmart tracking food origin | Every step recorded, can't be faked |
| **Identity** | Self-sovereign ID | You control your data, not Facebook |
| **Voting** | Transparent elections | Votes can't be altered after casting |
| **Healthcare** | Medical records | Patient controls access, records are immutable |
| **Real estate** | Property records on-chain | No forged documents, instant verification |

---

## 4. Hashing -- The Foundation of Blockchain Security

### What is hashing?

A **hash function** takes any input (a word, a file, an entire movie) and produces a **fixed-size output** called a hash.

```
Input: "hello"
SHA-256 Hash: 2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824

Input: "Hello" (capital H)
SHA-256 Hash: 185f8db32271fe25f561a6fc938b2e264306ec304eda518007d1764826381969
```

Notice:
- The output is always 64 characters (256 bits) regardless of input size
- Changing ONE character completely changes the output
- You cannot reverse-engineer the input from the output

```
  How a hash function works:

  ANY input                         Fixed-size output (256 bits)
  +------------------+             +----------------------------------+
  | "hello"          | -- SHA256 ->| 2cf24dba5fb0a30e26e83b2ac5b9e... |
  +------------------+             +----------------------------------+

  +------------------+             +----------------------------------+
  | "Hello"          | -- SHA256 ->| 185f8db32271fe25f561a6fc938b2... |  (completely different!)
  +------------------+             +----------------------------------+

  +------------------+             +----------------------------------+
  | entire 2GB movie | -- SHA256 ->| 9f86d081884c7d659a2feaa0c55ad... |  (still 64 chars)
  +------------------+             +----------------------------------+

  ONE character changes = ENTIRE hash changes (avalanche effect)
```

### Properties of a good hash function (SHA-256)

| Property | What it means |
|----------|--------------|
| **Deterministic** | Same input always gives same output |
| **Fast** | Computing a hash takes milliseconds |
| **One-way** | You can't get the input from the output |
| **Avalanche effect** | Tiny input change = completely different output |
| **Collision resistant** | Nearly impossible for two different inputs to produce the same hash |

### How blockchain uses hashing

Every block contains:
1. **Data** (list of transactions)
2. **Previous block's hash** (the link in the chain)
3. **Its own hash** (calculated from data + previous hash + nonce + timestamp)

```
Block 1                    Block 2                    Block 3
+-----------------+        +-----------------+        +-----------------+
| Prev: 0000...   |        | Prev: a1b2c3... |        | Prev: d4e5f6... |
| Data: "Genesis" |        | Data: "Alice ->  |        | Data: "Charlie  |
| Hash: a1b2c3... | -----> |  Bob: 5 BTC"    | -----> |  -> Dave: 2 BTC"|
+-----------------+        | Hash: d4e5f6... |        | Hash: 7g8h9i... |
                           +-----------------+        +-----------------+
```

If someone changes data in Block 2, its hash changes, which means Block 3's "Previous hash" no longer matches. The chain is broken. Every node on the network would reject this.

---

## 5. Immutable Ledger

**Immutable** means "cannot be changed." Once a block is added to the blockchain, the data inside it is permanent.

### Why is it immutable?

1. Each block's hash depends on its content AND the previous block's hash
2. Changing one block changes its hash
3. That breaks every block after it (because they reference the old hash)
4. To "fix" the chain, you'd need to recompute every block's hash
5. AND do it faster than the entire network is adding new blocks
6. This requires more than 50% of the network's computing power (practically impossible)

```
  ORIGINAL CHAIN (valid):

  Block 100          Block 101          Block 102          Block 103
  [Hash: 0000ab] --> [Prev: 0000ab] --> [Prev: 0000cd] --> [Prev: 0000ef]
                     [Hash: 0000cd]     [Hash: 0000ef]     [Hash: 0000gh]
                          OK                 OK                  OK


  AFTER TAMPERING Block 101:

  Block 100          Block 101          Block 102          Block 103
  [Hash: 0000ab] --> [Prev: 0000ab] --> [Prev: 0000cd] --> [Prev: 0000ef]
                     [Hash: 9f3x7z]     [Hash: 0000ef]     [Hash: 0000gh]
                       CHANGED!              |                   |
                                        0000cd != 9f3x7z    BROKEN TOO
                                          BROKEN!
```

### Why does immutability matter?

- **Trust**: You don't need to trust anyone -- the math guarantees the data hasn't been changed
- **Audit trail**: Every transaction ever made is permanently recorded
- **No fraud**: A bank employee can't secretly alter transaction records

---

## 6. P2P Network -- No Central Server

### Centralized vs P2P

**Centralized network** (how most apps work today):
```
         CENTRALIZED                              PEER-TO-PEER (P2P)
     (Banks, Instagram)                             (Blockchain)

       +----------+                          [Node]---[Node]---[Node]
       |  SERVER  |                            |  \   / |  \   / |
       | (single  |                            |   \ /  |   \ /  |
       |  owner)  |                            |   / \  |   / \  |
       +----+-----+                          [Node]---[Node]---[Node]
      / |   |   | \                            |  \   / |  \   / |
     /  |   |   |  \                           |   \ /  |   \ /  |
   [C] [C] [C] [C] [C]                      [Node]---[Node]---[Node]

   Server dies = ALL dead               Any node dies = network is fine
   Server censors = you're blocked       No one can censor anyone
   One copy of data                      Thousands of copies everywhere
```
- **Centralized**: One server controls everything. If it goes down, the whole system fails
- **P2P**: Every participant (node) is equal. No single point of failure
- Every participant (node) is equal
- No single point of failure
- No one can censor or shut it down
- Every node has a full copy of the blockchain

### How does the P2P network stay in sync?

When a new block is mined:
1. The miner broadcasts it to all connected nodes
2. Each node verifies the block (checks hash, checks transactions)
3. If valid, the node adds it to its copy and broadcasts to its peers
4. Within seconds, every node on the network has the same blockchain

```
  Step 1: Miner finds block        Step 2: Broadcast         Step 3: Propagation

      [Miner]                       [Miner]                   [Miner]
     "Found it!"                   / | | | \                  Everyone has
                             [Node] [N] [N] [Node]            the new block
                               |              |               within seconds
                             [Node]         [Node]
```

**Bitcoin has ~15,000+ nodes worldwide.** To shut it down, you'd need to shut down every single one simultaneously, across every country. That's effectively impossible.

---

## 7. Mining -- How New Blocks Are Created

### What is mining?

Mining is the process of:
1. Collecting pending transactions from the network
2. Bundling them into a block
3. Finding a special number (nonce) that makes the block's hash satisfy a difficulty requirement
4. Broadcasting the block to the network for a reward

```
  THE MINING PROCESS:

  +----------+      +-----------+      +-------------+      +------------+
  | Mempool  | ---> | Build     | ---> | Find valid  | ---> | Broadcast  |
  | (pending |      | candidate |      | nonce       |      | to network |
  |  txns)   |      | block     |      | (billions   |      | + collect  |
  +----------+      +-----------+      |  of tries)  |      | reward!    |
                                       +-------------+      +------------+
                                       (THIS is the hard part --
                                        takes ~10 min on avg)
```

### The nonce and difficulty target

The block's hash must start with a certain number of zeros. For example:

```
Target: hash must start with "0000"

Try nonce 1:    hash = 8a3f... (no)
Try nonce 2:    hash = b72c... (no)
Try nonce 3:    hash = 19e4... (no)
...
Try nonce 4521: hash = 00001c3a... (yes!)
```

The miner tries billions of nonces until one produces a hash below the target. This is called **Proof of Work** -- the valid nonce proves you did the computational work.

### Why does mining exist?

- **Security**: Makes it expensive to attack the network (you'd need massive computing power)
- **New coin distribution**: Miners receive newly created coins as a reward
- **Transaction processing**: Miners validate and include transactions in blocks

---

## 8. Byzantine Generals Problem

This is the fundamental problem that blockchain solves.

### The analogy

Imagine 10 generals surrounding a city. They need to attack simultaneously or all retreat -- a split decision means defeat. But:
- They can only communicate by messenger
- Some generals might be **traitors** sending fake messages
- How do the loyal generals agree on a plan?

```
                         CITY
                     +-----------+
                     |           |
          General A  |           |  General B
          "ATTACK!"  |           |  "ATTACK!"
               \     |           |     /
                \    |           |    /
                 \   +-----------+   /
                  \       |         /
         General C \      |        / General D
         "ATTACK!"  \     |       /  "RETREAT!" <-- TRAITOR!
                     \    |      /
                      \   |     /
                   General E
                   "ATTACK!"

  Problem: How do A, B, C, E agree on "ATTACK"
           when D is lying and saying "RETREAT"?

  Solution: CONSENSUS MECHANISM (PoW, PoS)
```

### How this maps to blockchain

- Generals = nodes on the network
- Messages = transactions and blocks
- Traitors = malicious nodes trying to cheat
- The attack plan = which transactions are valid

Blockchain solves this with **consensus mechanisms** -- rules that let honest nodes agree on the truth even when some nodes are lying.

---

## 9. Consensus Mechanisms

### Proof of Work (PoW) -- Used by Bitcoin

```
  PROOF OF WORK:

  Miner A: "nonce 1... nope. nonce 2... nope. nonce 3... nope..."
  Miner B: "nonce 1... nope. nonce 2... nope..."
  Miner C: "nonce 1... nope. nonce 2... YES! Hash starts with 0000!"
                                          |
                                          v
                                   Miner C broadcasts block
                                   and earns 3.125 BTC reward
                                          |
                                          v
                              All other nodes verify:
                              "Does nonce produce valid hash? YES."
                              (easy to verify, hard to find)
```

**How it works:**
1. Miners compete to solve the hash puzzle (find a valid nonce)
2. The first miner to solve it broadcasts the block
3. Other nodes verify the solution (easy to verify, hard to find)
4. The winning miner gets the block reward (currently 3.125 BTC)

**Why it's secure:** Cheating requires more computing power than all honest miners combined (51% attack). For Bitcoin, this would cost billions of dollars in hardware and electricity.

**Downside:** Enormous energy consumption. Bitcoin uses roughly as much electricity as a small country.

### Proof of Stake (PoS) -- Used by Ethereum (since 2022)

```
  PROOF OF STAKE:

  +------------------+     +------------------+     +------------------+
  | Validator A      |     | Validator B      |     | Validator C      |
  | Staked: 32 ETH   |     | Staked: 32 ETH   |     | Staked: 32 ETH   |
  +------------------+     +------------------+     +------------------+
           |                        |                        |
           v                        v                        v
     [Network randomly       [Not selected]           [Not selected]
      selects A to
      propose block]
           |
           v
     A proposes block -----> B and C verify and attest
           |                        |
           v                        v
     Block added             A earns rewards (~3-5% APR)
     to chain
                    If A cheats: staked ETH gets SLASHED (destroyed)
```

**How it works:**
1. Validators lock up (stake) their coins as collateral
2. The network randomly selects a validator to propose the next block
3. Other validators verify and attest to the block
4. The proposer earns rewards (transaction fees + small amount of new ETH)
5. If a validator cheats, their stake gets **slashed** (destroyed)

**Why it's secure:** Cheating means losing your staked coins. To attack Ethereum, you'd need to stake (and risk losing) billions of dollars worth of ETH.

**Upside:** 99.9% less energy than PoW. No need for mining hardware.

### Longest Chain Rule

When two miners find a valid block at almost the same time, the network temporarily has two versions of the chain. The rule: **whichever chain gets the next block first wins.** The other block becomes an "orphan" and is discarded. This is why exchanges wait for multiple "confirmations" before crediting your deposit.

```
  Two miners find Block 5 at almost the same time:

                        +---[Block 5a]---[Block 6a]---[Block 7a]  <-- THIS WINS (longer)
                       /
  ...[Block 3]---[Block 4]
                       \
                        +---[Block 5b]  <-- orphaned (shorter chain loses)

  Rule: whichever fork gets the next block first becomes the "real" chain.
  The other is discarded. This is why exchanges wait for 3-6 "confirmations."
```

---

## 10. Byzantine Fault Tolerance (BFT)

A system is **Byzantine Fault Tolerant** if it continues to work correctly even when some participants are malicious or faulty.

- Bitcoin (PoW) tolerates up to ~49% malicious miners
- Ethereum (PoS) tolerates up to ~33% malicious validators
- Solana uses a variant called **Tower BFT** (optimized for speed)

This is what makes blockchain "trustless" -- you don't trust any individual participant, you trust that the math prevents any minority from cheating.

---

# Module 2: Bitcoin & Cryptocurrency

---

## 11. What is Bitcoin?

Bitcoin is the **first cryptocurrency**, created in 2009 by the pseudonymous **Satoshi Nakamoto**. No one knows who Satoshi is (person or group).

### Bitcoin vs Blockchain

| | Bitcoin | Blockchain |
|---|---|---|
| **What it is** | A cryptocurrency (digital money) | A technology (data structure) |
| **Analogy** | Email | The internet |
| **Relationship** | Bitcoin USES blockchain | Blockchain can be used for many things beyond Bitcoin |

Bitcoin was the first application of blockchain, but blockchain technology has since been used for thousands of other projects.

### Key properties of Bitcoin

- **Digital**: Exists only on computers, no physical coins
- **Decentralized**: No company or government controls it
- **Pseudonymous**: Addresses are random strings, not names (but not fully anonymous)
- **Borderless**: Send money anywhere in the world in minutes
- **Censorship-resistant**: No one can freeze your Bitcoin or block a transaction
- **Scarce**: Only 21 million will ever exist

---

## 12. Bitcoin's Monetary Policy

### Why only 21 million?

Satoshi hardcoded a maximum supply of **21 million BTC** into the Bitcoin protocol. This makes Bitcoin **deflationary** (scarce), unlike government currencies that can be printed endlessly.

### Block rewards and halving

```
  BITCOIN HALVING TIMELINE:

  50 BTC ----+
             |
  25 BTC     +----+
                  |
  12.5 BTC        +----+
                       |
  6.25 BTC             +----+
                            |
  3.125 BTC                 +----+
                                 |
  1.5625 BTC                     +----+
                                      |       ...eventually 0 BTC
  0 BTC                               +-----------------------------> 2140
       2009   2012   2016   2020   2024   2028

  Every ~4 years (210,000 blocks), the reward cuts in half.
  Fewer new coins = increasing scarcity = historically drives price up.
```

Miners earn new Bitcoin for every block they mine. But the reward **halves** every 210,000 blocks (~4 years):

| Year | Block reward | Total BTC in circulation |
|------|-------------|------------------------|
| 2009 | 50 BTC | ~0 |
| 2012 | 25 BTC | ~10.5 million |
| 2016 | 12.5 BTC | ~15.75 million |
| 2020 | 6.25 BTC | ~18.375 million |
| 2024 | 3.125 BTC | ~19.6 million |
| 2028 | 1.5625 BTC | ~20.3 million |
| ~2140 | 0 BTC | 21 million (final) |

After 2140, miners will only earn transaction fees. The halving creates scarcity -- each cycle, fewer new coins enter circulation, which historically has driven price increases.

### Why does this matter?

Compare to USD:
- The US government printed ~40% of all dollars in existence during 2020-2021
- More printing = each dollar is worth less (inflation)
- Bitcoin can't be printed. 21 million. Forever. That's the whole point.

---

## 13. Mining in Detail -- Nonce, Target, Difficulty

### What miners actually do

1. Collect unconfirmed transactions from the **mempool**
2. Build a candidate block (header + transactions)
3. Hash the block header with different **nonce** values
4. Check if the hash is below the **target**
5. If yes, broadcast the block. If no, try another nonce.

### What is a nonce?

**Nonce** = "number used once." It's the variable miners change to get a different hash output.

```
Block header: [prev_hash + merkle_root + timestamp + target + nonce]

nonce = 0    -> hash = f391a8c2...  (too high)
nonce = 1    -> hash = e28b4d71...  (too high)
nonce = 2    -> hash = 193c7f08...  (too high)
...
nonce = 8391 -> hash = 00000a3f...  (below target -- valid!)
```

Miners try **billions** of nonces per second. A modern ASIC miner does ~100 TH/s (100 trillion hashes per second).

### Difficulty adjustment

Every 2,016 blocks (~2 weeks), Bitcoin automatically adjusts the difficulty:
- Blocks coming too fast? -> Increase difficulty (more leading zeros required)
- Blocks coming too slow? -> Decrease difficulty (fewer leading zeros)

```
  DIFFICULTY ADJUSTMENT (self-regulating):

  Target: 1 block every 10 minutes

  More miners join         Fewer miners leave
  (blocks too fast)        (blocks too slow)
        |                        |
        v                        v
  Difficulty UP              Difficulty DOWN
  (need more zeros)          (need fewer zeros)
        |                        |
        v                        v
  Target: 0000000...         Target: 0000...
  (harder to find)           (easier to find)
        |                        |
        v                        v
  Blocks slow back           Blocks speed back
  to ~10 min                 to ~10 min

  This automatic adjustment is why Bitcoin has maintained
  ~10 min blocks since 2009, despite mining power growing
  by 10,000,000,000x
```

This keeps block time at roughly **10 minutes** regardless of how much mining power joins or leaves the network.

---

## 14. CPU vs GPU vs ASIC

| Hardware | Speed | Cost | Used for |
|----------|-------|------|----------|
| **CPU** | ~10 MH/s | Already own it | Nothing (too slow for Bitcoin now) |
| **GPU** | ~30-100 MH/s | $300-2000 | Ethereum (before PoS), altcoins |
| **ASIC** | ~100 TH/s | $2000-15000 | Bitcoin exclusively |

**ASIC** = Application-Specific Integrated Circuit. A chip designed to do ONE thing: SHA-256 hashing. It's 1 million times faster than a CPU for this specific task, but can't do anything else.

This is why you can't mine Bitcoin on your laptop anymore. The difficulty has risen so high that only warehouses full of ASICs are profitable.

---

## 15. Mining Pools

A single miner finding a block is like winning the lottery. So miners join **pools** -- groups that combine their computing power and split the reward.

```
  SOLO MINING vs POOL MINING:

  Solo:                              Pool:
  +--------+                         +--------+ +--------+ +--------+
  | Miner  |                         | Miner  | | Miner  | | Miner  |
  | (you)  |                         |   A    | |   B    | |   C    |
  +--------+                         +---+----+ +---+----+ +---+----+
      |                                   \         |         /
  Chance of finding                        \        |        /
  block: 0.0001%                         +-----------+--------+
  (like winning lottery)                 |    MINING POOL     |
                                         | (combines power)   |
                                         +--------+-----------+
                                                  |
                                         Finds blocks regularly
                                         Splits reward by contribution
                                         (like a lottery syndicate)
```

**How it works:**
1. Pool assigns each miner a portion of the nonce range
2. All miners hash simultaneously
3. When any miner finds a valid block, the pool submits it
4. The reward is split proportionally to each miner's contribution

**Major pools:** Foundry USA, AntPool, F2Pool, ViaBTC

**Risk:** If one pool gets >50% of the network hashrate, it could theoretically execute a 51% attack. This has never happened with Bitcoin, but it's a concern the community watches closely.

---

## 16. Mempool -- The Waiting Room

The **mempool** (memory pool) is where unconfirmed transactions wait before being included in a block.

```
  THE MEMPOOL -- Transaction Lifecycle:

  You send BTC                    Mempool (waiting room)
  +--------+                    +---------------------------+
  | "Send  | --- broadcast ---> | Tx: Alice->Bob  fee: $2   |
  |  1 BTC |     to network     | Tx: Carol->Dan  fee: $5   | <-- higher fee = priority
  |  to    |                    | Tx: Eve->Frank  fee: $0.5 |
  |  Bob"  |                    | Tx: Grace->Hank fee: $8   | <-- miners pick this first
  +--------+                    +---------------------------+
                                           |
                                    Miner picks txns
                                    (highest fee first)
                                           |
                                           v
                                    +-------------+
                                    | New Block   |
                                    | [Grace->Hank|  confirmed!
                                    |  Carol->Dan |  confirmed!
                                    |  Alice->Bob |  confirmed!
                                    +-------------+
                                    Eve's tx left behind (fee too low)
```

### How it works

1. You send 1 BTC to a friend
2. Your transaction is broadcast to the network
3. Every node adds it to their mempool
4. Miners pick transactions from the mempool to include in their next block
5. **Miners prioritize transactions with higher fees** (they keep the fees)
6. Once included in a mined block, the transaction is "confirmed"

### Why transactions get stuck

If the network is busy and you set a low fee, your transaction sits in the mempool for hours or even days. During the 2017 and 2021 bull runs, Bitcoin fees reached $50-60 per transaction because the mempool was overloaded.

---

## 17. Transactions and UTXOs

### What is a UTXO?

**UTXO** = Unspent Transaction Output. Bitcoin doesn't track "balances" like a bank. Instead, it tracks individual chunks of Bitcoin that haven't been spent yet.

### Example

Alice has 0.7 BTC (from a previous transaction). She wants to send 0.3 BTC to Bob.

```
  UTXO MODEL -- like paying with cash, you get change back:

  REAL WORLD ANALOGY:                     BITCOIN:

  You have a $7 bill                      Alice has 1 UTXO worth 0.7 BTC
  You want to pay $3                      She wants to send 0.3 BTC to Bob
       |                                        |
       v                                        v
  +----------+                            +------------------+
  | $7 bill  |                            | 0.7 BTC (input)  |
  +----+-----+                            +----+-------------+
       |                                        |
       +---> $3 to shopkeeper                   +---> 0.3   BTC -> Bob
       |                                        |
       +---> $4 change back to you              +---> 0.399 BTC -> Alice (change)
                                                |
                                                +---> 0.001 BTC -> Miner (fee)

  Total in = Total out                    0.7 = 0.3 + 0.399 + 0.001
```

Alice doesn't "send 0.3 from her balance." She spends the entire 0.7 UTXO and gets change back, just like paying for a $3 item with a $7 bill.

### Why UTXOs?

- Every UTXO can be independently verified
- No need for a central balance database
- Enables parallel transaction verification

---

## 18. Transaction Fees

- Fees are **not fixed** -- you choose how much to pay
- Higher fee = faster confirmation (miners prioritize you)
- Fee is calculated per byte of transaction data, not per amount sent
- Sending $10 and sending $10 million costs the **same fee** (same data size)
- During low-traffic periods, fees can be as low as $0.10-0.50

---

## 19. Wallets

A crypto wallet doesn't "store" your coins. Your coins are always on the blockchain. The wallet stores your **private keys** -- the passwords that prove you own those coins.

```
  COMMON MISCONCEPTION:

  What people think:                      What actually happens:

  +----------+                            +------ BLOCKCHAIN ------+
  | Wallet   |                            |                        |
  | [coins]  |  <-- coins live            |  Address 1: 0.5 BTC   |
  | [coins]  |      in the wallet?        |  Address 2: 2.3 BTC   |
  | [coins]  |      NO!                   |  Address 3: 0.1 BTC   |
  +----------+                            +------------------------+
                                                     ^
  Your wallet only                          +--------+--------+
  holds KEYS, not coins                     |     Wallet      |
                                            | [private key 1] |  <-- keys that
                                            | [private key 2] |     PROVE you own
                                            | [private key 3] |     those addresses
                                            +-----------------+
```

### Types of wallets

| Type | Examples | Security | Convenience |
|------|----------|----------|-------------|
| **Hardware wallet (cold)** | Ledger, Trezor | Highest (offline) | Low (need physical device) |
| **Software wallet (hot)** | MetaMask, Phantom, Trust Wallet | Medium (online) | High (always accessible) |
| **Exchange wallet** | Coinbase, Binance | Low (they control keys) | Highest (built into exchange) |
| **Paper wallet** | QR code printed on paper | High (offline) | Very low |

**"Not your keys, not your coins"** -- if you leave crypto on an exchange, the exchange controls it. They can freeze your account, get hacked, or go bankrupt (see: FTX collapse in 2022).

---

## 20. Public Key and Private Key

### The key pair

Every wallet is based on a **key pair**:

| | Private key | Public key |
|---|---|---|
| **What it is** | A random 256-bit number | Derived from the private key using elliptic curve cryptography |
| **Who knows it** | Only you. NEVER share it. | Everyone. It's your "address" for receiving funds. |
| **Analogy** | The password to your bank account | Your bank account number |
| **If you lose it** | Your coins are gone forever | Just generate a new receiving address |

### How transactions are signed

```
  DIGITAL SIGNATURE FLOW:

  Step 1: Create transaction
  +----------------------------------+
  | "Send 0.5 BTC to Bob"           |
  +----------------------------------+
                |
  Step 2: Sign with PRIVATE key (only you have this)
                |
                v
  +----------------------------------+
  | Transaction + Digital Signature  |  --- broadcast to network --->
  +----------------------------------+
                                                    |
  Step 3: Anyone can verify with your PUBLIC key    |
                                                    v
                                          +-------------------+
                                          | Nodes verify:     |
                                          | "Does signature   |
                                          |  match public     |
                                          |  key?" -> YES     |
                                          | "Transaction is   |
                                          |  authorized"      |
                                          +-------------------+

  Private key NEVER leaves your wallet.
  The signature proves ownership without revealing the key.
```

1. You create a transaction: "Send 0.5 BTC to Bob"
2. Your wallet signs it with your **private key** (digital signature)
3. Anyone can verify the signature using your **public key**
4. This proves YOU authorized the transaction without revealing your private key

This is why you never, ever share your private key or seed phrase with anyone. There is no "forgot password" in crypto. No customer support. No recovery. Lost key = lost coins.

### Public key vs Bitcoin address

Your Bitcoin address is not exactly your public key -- it's a **hashed version** of your public key (shorter, with a checksum for error detection):

```
  KEY DERIVATION (one-way, can't go backwards):

  Private Key                      Public Key                       BTC Address
  (256-bit random number)          (derived via elliptic curve)     (hashed + checksum)

  5HueCGU8rMjxEXxiP...   --->     04d0de0aaeaefad02b...   --->     1GAehh7TsJAHuU...
  (64 hex chars)                   (130 hex chars)                  (34 chars, starts with 1)

       SECRET                      SHAREABLE                        SHAREABLE
    (never share)               (for verification)              (give this to receive BTC)

  You CAN go:   Private --> Public --> Address
  You CANNOT:   Address --> Public --> Private   (mathematically impossible)
```

---

## 21. Seed Phrase (Mnemonic)

A **seed phrase** is 12 or 24 random words that can regenerate all your private keys:

```
abandon ability able about above absent absorb abstract absurd abuse access accident
```

This is the ULTIMATE backup. If your phone breaks, your laptop is stolen, or your hardware wallet is destroyed -- these 12/24 words can recover everything.

**Rules:**
- Write it on paper. Never store it digitally (not in Notes, not in a photo, not in email)
- Store it in a safe, ideally in two separate physical locations
- Anyone with your seed phrase has full control of your coins

---

## 22. HD Wallets (Hierarchical Deterministic)

Modern wallets are **HD wallets** -- from a single seed phrase, they can generate an unlimited number of key pairs.

```
Seed phrase
  |
  +--> Account 1
  |      +--> Address 1 (for receiving)
  |      +--> Address 2 (for receiving)
  |      +--> Address 3 (change address)
  |
  +--> Account 2
         +--> Address 1
         +--> Address 2
```

**Why multiple addresses?** Privacy. If you use the same address for every transaction, anyone can trace your entire financial history on the public blockchain. HD wallets generate a fresh address for each transaction.

---

## 23. SegWit (Segregated Witness)

**SegWit** was a 2017 Bitcoin upgrade that separated (segregated) the transaction signature (witness) data from the main transaction data.

```
  BEFORE SegWit:                          AFTER SegWit:

  +------------------------+              +------------------+  +----------+
  | Transaction Data       |              | Transaction Data |  | Witness  |
  |                        |              |                  |  | (sigs)   |
  | - sender               |              | - sender         |  |          |
  | - receiver             |              | - receiver       |  | Separated|
  | - amount               |              | - amount         |  | out!     |
  | - SIGNATURE (big!)     |              +------------------+  +----------+
  +------------------------+
                                          Transaction is ~40% smaller
  Takes up a lot of space                 = more txns fit per block
  in each block                           = lower fees for everyone
```

**Why it matters:**
- Reduced transaction size by ~40% (more transactions per block)
- Lower fees
- Fixed a bug called "transaction malleability"
- Enabled future upgrades like the Lightning Network

**How to spot it:** SegWit addresses start with `bc1` (e.g., `bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh`). Legacy addresses start with `1` or `3`.

---

# Module 3: Ethereum

---

## 24. What is Ethereum?

Ethereum is a blockchain that can run **programs** (smart contracts), not just transfer money.

| | Bitcoin | Ethereum |
|---|---|---|
| **Created** | 2009 by Satoshi Nakamoto | 2015 by Vitalik Buterin |
| **Purpose** | Digital money | Programmable blockchain |
| **Language** | Script (very limited) | Solidity (Turing-complete) |
| **Block time** | ~10 minutes | ~12 seconds |
| **Consensus** | Proof of Work | Proof of Stake (since Sep 2022) |
| **Supply** | 21 million cap | No hard cap (but ~120 million ETH exist) |

```
  BITCOIN vs ETHEREUM:

  Bitcoin = Calculator                    Ethereum = Smartphone
  +------------------+                   +---------------------------+
  |                  |                   |  [DeFi] [NFTs] [Games]   |
  |   7 + 5 = 12    |                   |  [DAOs] [DEXs] [Tokens]  |
  |                  |                   |  [Identity] [Voting]     |
  |  Does ONE thing  |                   |  [Insurance] [Lending]   |
  |  really well:    |                   |                           |
  |  send money      |                   |  Runs ANY application     |
  +------------------+                   +---------------------------+
```

**Analogy:** If Bitcoin is a calculator (does one thing well: transfers), Ethereum is a smartphone (can run any app).

---

## 25. Nodes in Ethereum

A **node** is a computer running Ethereum software that stores a copy of the blockchain and validates transactions.

| Type | What it stores | Purpose |
|------|---------------|---------|
| **Full node** | Current state + recent blocks | Validates transactions independently |
| **Archive node** | Complete history of every state change | Historical queries, block explorers |
| **Light node** | Only block headers | Low-resource devices, quick verification |

Anyone can run a node. You don't need special hardware (unlike Bitcoin mining). Running a node makes you a first-class participant in the network.

---

## 26. Accounts in Ethereum

Ethereum has TWO types of accounts (Bitcoin only has one):

```
  TWO TYPES OF ETHEREUM ACCOUNTS:

  Externally Owned Account (EOA)          Contract Account
  +-------------------------+             +-------------------------+
  |                         |             |                         |
  | Controlled by: YOU      |             | Controlled by: CODE     |
  | (private key)           |             | (smart contract)        |
  |                         |             |                         |
  | Has: ETH balance        |             | Has: ETH balance + CODE |
  |                         |             |                         |
  | Can start txns: YES     |    calls    | Can start txns: NO      |
  | (you click "send")      | ----------> | (only REACTS to calls)  |
  |                         |             |                         |
  | Example: MetaMask       |             | Example: Uniswap        |
  +-------------------------+             +-------------------------+

  You (EOA) ---> Smart Contract ---> does something ---> sends result back
                 (Contract Account)
```

### Externally Owned Account (EOA)
- Controlled by a private key (a person)
- Can send transactions
- Has an ETH balance
- Example: your MetaMask wallet

### Contract Account
- Controlled by code (a smart contract)
- Cannot initiate transactions (only responds to them)
- Has an ETH balance AND code
- Example: Uniswap's swap contract

| | EOA | Contract Account |
|---|---|---|
| **Controlled by** | Private key | Smart contract code |
| **Has code?** | No | Yes |
| **Can initiate transactions?** | Yes | No (only reacts) |
| **Creation** | Free (just generate a key pair) | Costs gas (deploying code) |

---

## 27. Smart Contracts

A **smart contract** is a program stored on the blockchain that executes automatically when conditions are met.

### Simple analogy

```
  SMART CONTRACT = VENDING MACHINE

  +-------------------------------------------+
  |              VENDING MACHINE               |
  |                                           |
  |   1. Insert $2.00          [$$]           |
  |                              |            |
  |   2. Select item B3         [B3]          |
  |                              |            |
  |   3. Machine checks:                      |
  |      - Enough money? YES                  |
  |      - Item in stock? YES                 |
  |                              |            |
  |   4. Machine dispenses item  [item]       |
  |      + returns change        [$0.25]      |
  |                                           |
  |   No human cashier needed.                |
  |   The machine IS the rules.               |
  +-------------------------------------------+

  Smart contract = same thing, but on the blockchain.
  The CODE is the rules. It executes automatically.
  No company, no middleman, no trust needed.
```

A vending machine is a smart contract:
1. You insert money (input)
2. You select an item (condition)
3. The machine gives you the item (automatic execution)
4. No human needed. No trust needed. The machine enforces the rules.

### On Ethereum

Smart contracts are written in **Solidity** (most common) and deployed to the blockchain. Once deployed, the code **cannot be changed** (immutable, just like blockchain data).

```
Example smart contract logic (in English):

IF user sends 0.1 ETH to this contract
AND the crowdfunding goal has been reached
THEN send all collected ETH to the project owner

IF the deadline passes AND goal is NOT reached
THEN allow all contributors to withdraw their ETH
```

This is a crowdfunding contract. No Kickstarter needed. No company taking 10% fees. The code IS the middleman.

### Turing Complete vs Non-Turing Complete

| | Bitcoin Script | Solidity (Ethereum) |
|---|---|---|
| **Turing complete?** | No | Yes |
| **What it means** | Can only do basic operations (verify signatures, time locks) | Can compute anything a computer can compute |
| **Loops?** | No | Yes |
| **Complex logic?** | No | Yes (DeFi, games, DAOs, NFTs) |

**Turing complete** means the language can solve any computational problem given enough time and resources. This is what makes Ethereum so much more powerful than Bitcoin for applications beyond simple payments.

---

## 28. DApps (Decentralized Applications)

A **DApp** is an application where the backend runs on a blockchain (smart contracts) instead of a company's servers.

### Centralized app vs DApp

| | Instagram (centralized) | Uniswap (DApp) |
|---|---|---|
| **Backend** | Meta's servers | Ethereum smart contracts |
| **Who controls it** | Meta | No one (code is law) |
| **Can they ban you?** | Yes | No |
| **Can they change rules?** | Yes, anytime | Only if contract allows it |
| **Data ownership** | Meta owns your data | You own your data |
| **Downtime** | Server goes down = app dies | As long as Ethereum runs, Uniswap runs |

### Popular DApps

| DApp | What it does | Chain |
|------|-------------|-------|
| **Uniswap** | Decentralized exchange (swap tokens) | Ethereum |
| **Aave** | Lending and borrowing | Ethereum |
| **OpenSea** | NFT marketplace | Ethereum |
| **Lido** | Liquid staking | Ethereum |
| **Raydium** | DEX | Solana |
| **Magic Eden** | NFT marketplace | Solana, Bitcoin |

---

## 29. EVM (Ethereum Virtual Machine)

The **EVM** is the runtime environment where smart contracts execute. Think of it as a virtual computer that runs on every Ethereum node simultaneously.

```
  HOW THE EVM WORKS:

  Your Solidity Code            Compiled to Bytecode         Runs on EVM
  +------------------+         +------------------+         +------------------+
  | function swap()  |  --->   | 0x60806040...    |  --->   | Every node runs  |
  | { ... }          |         | (machine code)   |         | the SAME code    |
  +------------------+         +------------------+         | gets SAME result |
                                                            +------------------+
                                                                    |
                                                            Each operation costs
                                                            GAS (prevents abuse)

  Node in USA:    runs bytecode -> result: X
  Node in Japan:  runs bytecode -> result: X    (deterministic -- always same)
  Node in India:  runs bytecode -> result: X
```

**Key properties:**
- **Sandboxed**: Contract code can't access the node's file system, network, or other contracts (unless explicitly called)
- **Deterministic**: Same input always produces same output on every node
- **Metered**: Every operation costs gas (prevents infinite loops and abuse)

**EVM-compatible chains**: Many other blockchains copy the EVM so developers can deploy the same Solidity code:
- Polygon, BNB Chain (Binance), Avalanche, Arbitrum, Optimism, Base

This is why Solidity skills are so valuable -- one language works across dozens of chains.

---

## 30. Gas -- The Cost of Computation

**Gas** is the unit measuring computational work on Ethereum. Every operation costs gas:

| Operation | Gas cost |
|-----------|----------|
| Sending ETH (simple transfer) | 21,000 gas |
| Storing a new value | 20,000 gas |
| Simple addition | 3 gas |
| Deploying a contract | 100,000 - 1,000,000+ gas |

### Gas Price

You choose how much to pay per unit of gas (in **gwei**, where 1 gwei = 0.000000001 ETH):

```
Transaction cost = Gas used x Gas price

Example:
Simple transfer:  21,000 gas x 20 gwei = 420,000 gwei = 0.00042 ETH (~$1.50)
Complex DeFi swap: 200,000 gas x 20 gwei = 4,000,000 gwei = 0.004 ETH (~$14)
```

### Gas Limit

The **gas limit** is the maximum gas you're willing to spend on a transaction:
- Set it too low -> transaction fails (but you still pay for the gas used up to the failure)
- Set it too high -> no problem, unused gas is refunded

### Why gas exists

Without gas, anyone could deploy a contract with an infinite loop and freeze the entire network. Gas ensures every computation has a cost, preventing abuse.

### EIP-1559 (since Aug 2021)

```
  EIP-1559 FEE MODEL:

  Your transaction fee = Base Fee + Priority Fee (tip)

  +------------------+     +------------------+
  | BASE FEE         |     | PRIORITY FEE     |
  | (set by network) |     | (you choose)     |
  |                  |     |                  |
  | Network busy?    |     | Want it fast?    |
  |  -> fee goes UP  |     |  -> tip more     |
  |                  |     |                  |
  | Network quiet?   |     | Can wait?        |
  |  -> fee goes DOWN|     |  -> tip less     |
  +--------+---------+     +--------+---------+
           |                         |
           v                         v
       BURNED                   Goes to
    (destroyed forever!)       validator

  Because base fee is burned, ETH can be DEFLATIONARY:
  If more ETH burned > new ETH created = total supply shrinks
```

Ethereum reformed its fee system:
- **Base fee**: Automatically set by the network (burned, destroyed forever)
- **Priority fee (tip)**: You set this -- goes to the validator
- When the network is busy, base fee increases. When quiet, it decreases.
- The burning mechanism means ETH can be **deflationary** (more burned than created)

---

## 31. DAO -- Decentralized Autonomous Organization

A **DAO** is an organization governed by smart contracts and token holder votes instead of a CEO and board of directors.

```
  TRADITIONAL COMPANY vs DAO:

  Traditional Company:                    DAO:

  +-------------+                         +------------------+
  |    CEO      |  decisions              | Token Holders    |  vote on
  +------+------+  flow down              | (thousands of    |  proposals
         |                                |  people)         |
  +------+------+                         +--------+---------+
  |    Board    |                                  |
  +------+------+                           +------+------+
         |                                  | Smart       |
  +------+------+                           | Contract    |
  | Employees   |                           | (executes   |
  +-------------+                           |  automatically)|
                                            +-------------+
  Centralized                               Decentralized
  CEO can be corrupt                        Code can't be bribed
  Opaque decisions                          All votes are public
```

**How it works:**
1. Members buy governance tokens
2. Anyone can submit a proposal (e.g., "Fund this project with $500K from the treasury")
3. Token holders vote (1 token = 1 vote, usually)
4. If the proposal passes, the smart contract executes it automatically
5. No CEO, no board, no legal entity needed

**Examples:** MakerDAO (manages the DAI stablecoin), Uniswap DAO, Aave DAO

---

## 32. The DAO Attack (2016)

**The DAO** was one of the first DAOs on Ethereum -- a decentralized venture capital fund. It raised $150 million in ETH.

```
  THE DAO ATTACK TIMELINE:

  April 2016         June 2016              July 2016
  DAO launches       Hacker exploits        Community votes:
  raises $150M       re-entrancy bug        "Do we reverse it?"
       |                  |                       |
       v                  v                       v
  +----------+      +----------+           +-------------+
  | $150M in |      | $60M     |           | YES: 87%    |
  | the DAO  |      | drained! |           | (hard fork) |
  +----------+      +----------+           +------+------+
                                                  |
                                           +------+------+
                                          /               \
                             Ethereum (ETH)          Ethereum Classic (ETC)
                             reversed the hack       kept the hack
                             "community > code"      "code is law"
                             (what we use today)     (still exists)
```

**What happened:**
1. A hacker found a bug in the smart contract (re-entrancy vulnerability)
2. They drained $60 million worth of ETH
3. The Ethereum community was split: do we reverse the hack or respect "code is law"?
4. Vitalik and the majority voted to **hard fork** -- rewrite history to reverse the theft

This led to the chain splitting into:
- **Ethereum (ETH)** -- the fork that reversed the hack (what we use today)
- **Ethereum Classic (ETC)** -- the original chain that refused to reverse it ("code is law")

This event shaped the entire blockchain philosophy debate: should immutability be absolute, or should the community be able to fix catastrophic bugs?

---

## 33. Hard Fork vs Soft Fork

### Hard fork

A **backwards-incompatible** upgrade. Old nodes can't validate new blocks.

```
               +-- New chain (Ethereum)
              /
... Block N --
              \
               +-- Old chain (Ethereum Classic)
```

- Creates two separate blockchains
- All nodes must upgrade or get left behind
- Examples: Ethereum/Ethereum Classic split, Bitcoin/Bitcoin Cash split

### Soft fork

A **backwards-compatible** upgrade. Old nodes still accept new blocks (they just don't understand the new features).

- No chain split
- Old nodes work but can't use new features
- Examples: Bitcoin's SegWit upgrade

---

## 34. ICO -- Initial Coin Offering

An **ICO** is how crypto projects raise money by selling tokens to early investors.

```
  HOW AN ICO WORKS:

  Project Team                Smart Contract              Investors
  +-------------+           +----------------+           +-------------+
  | 1. Write    |           |                |           |             |
  |    whitepaper|          |  3. Investors   |           | 2. Read     |
  | 2. Create   | -------> |     send ETH    | <-------- |    whitepaper|
  |    ERC-20   |  deploy  |     receive     |  send ETH |    decide   |
  |    token    |          |     tokens      |           |    to invest|
  +-------------+           +----------------+           +-------------+
                                    |
                             4. If project succeeds:
                                token value goes UP
                             4. If project fails/scam:
                                token value goes to ZERO
```

**How it works:**
1. Project publishes a whitepaper explaining their idea
2. They create a token on Ethereum (usually ERC-20)
3. Investors send ETH to a smart contract and receive tokens in return
4. If the project succeeds, the token value increases

**ICO vs IPO:**

| | ICO | IPO |
|---|---|---|
| **What you get** | Tokens (may or may not have utility) | Shares (ownership in a company) |
| **Regulation** | Mostly unregulated (risky) | Heavily regulated by SEC |
| **Who can invest** | Anyone with a crypto wallet | Usually accredited investors first |
| **Cost to launch** | A few thousand dollars | Millions in legal/banking fees |

The 2017 ICO boom saw thousands of projects raise billions of dollars. Many were scams. This led to increased regulation and the shift toward more legitimate fundraising methods (IDOs, airdrops, fair launches).

---

## 35. Ethereum 2.0 -- The Merge (Proof of Stake)

In September 2022, Ethereum completed "The Merge" -- switching from Proof of Work to **Proof of Stake**.

### How PoS works on Ethereum

1. **Become a validator**: Deposit 32 ETH (~$100K at current prices) into the staking contract
2. **Propose blocks**: The network randomly selects a validator to propose the next block
3. **Attest**: Other validators verify the block and vote on it
4. **Earn rewards**: Validators earn ~3-5% APR on their staked ETH
5. **Get slashed**: If you cheat or go offline, your stake is partially destroyed

### Why the switch?

| | PoW (old Ethereum) | PoS (new Ethereum) |
|---|---|---|
| **Energy** | ~78 TWh/year (like a small country) | ~0.01 TWh/year (99.95% reduction) |
| **Hardware** | Expensive GPUs | Regular computer |
| **Barrier to entry** | Buying mining rigs | Staking 32 ETH |
| **Security model** | Cost of electricity to attack | Cost of staked ETH to attack |

---

## 36. Sharding

**Sharding** is Ethereum's plan to scale by splitting the network into multiple parallel chains (shards).

```
  SHARDING -- parallel processing:

  WITHOUT SHARDING:                       WITH SHARDING:

  Every node does ALL work                Work is split across shards

  Node 1: [tx1][tx2][tx3]...[tx1000]     Shard A: [tx1][tx2]...[tx250]
  Node 2: [tx1][tx2][tx3]...[tx1000]     Shard B: [tx251]...[tx500]
  Node 3: [tx1][tx2][tx3]...[tx1000]     Shard C: [tx501]...[tx750]
                                          Shard D: [tx751]...[tx1000]

  Throughput: 1x                          Throughput: 4x (or 64x with 64 shards)
```

**Current state**: Every node processes every transaction. If 1000 transactions happen, all nodes do all 1000.

**With sharding**: The network splits into 64 shards. Each shard processes a subset of transactions. Total throughput = 64x current capacity.

**Status**: Ethereum's roadmap has shifted toward **Danksharding** (focused on making Layer 2 rollups cheaper) rather than full execution sharding. Layer 2 solutions like Arbitrum and Optimism already handle much of the scaling load.

---

# Module 4: Solana

---

## 37. What is Solana?

Solana is a high-performance blockchain launched in 2020 by **Anatoly Yakovenko** (former Qualcomm engineer).

### Solana vs Ethereum

| | Ethereum | Solana |
|---|---|---|
| **TPS (transactions per second)** | ~15-30 (L1), 1000+ (with L2) | ~4,000-65,000 |
| **Block time** | ~12 seconds | ~400 milliseconds |
| **Average fee** | $0.50 - $50+ (varies wildly) | ~$0.00025 (fraction of a cent) |
| **Consensus** | Proof of Stake | Proof of Stake + Proof of History |
| **Smart contract language** | Solidity | Rust (and C/C++) |
| **VM** | EVM | SVM (Sealevel Virtual Machine) |
| **Trade-off** | More decentralized, battle-tested | Faster and cheaper, but less decentralized |

### Why is Solana so fast?

Three key innovations:

**1. Proof of History (PoH)**

The biggest bottleneck in blockchain is getting nodes to agree on the **order and timing** of events. Bitcoin and Ethereum solve this by waiting for blocks (10 min and 12 sec respectively).

Solana uses a **cryptographic clock** -- a chain of SHA-256 hashes that proves time has passed between events. Each hash depends on the previous one, creating an unbreakable sequence.

```
  PROOF OF HISTORY -- a cryptographic clock:

  hash_1 = SHA256("start")
  hash_2 = SHA256(hash_1)        <-- tx from Alice inserted here (timestamped!)
  hash_3 = SHA256(hash_2)
  hash_4 = SHA256(hash_3)        <-- tx from Bob inserted here
  hash_5 = SHA256(hash_4)
  ...
  hash_1000 = SHA256(hash_999)

  Each hash MUST come after the previous one (can't be computed in parallel).
  This creates an unbreakable timeline:

  time ----[hash_1]----[hash_2]----[hash_3]----[hash_4]----[hash_5]---->
                         ^                       ^
                     Alice's tx              Bob's tx
                   (provably BEFORE Bob)   (provably AFTER Alice)

  Traditional blockchain: "Wait for a block to order transactions" (slow)
  Solana: "Transactions are already ordered by PoH" (fast)
```

If you see hash_1000, you know 999 hashing operations happened before it. This proves time passed without needing all nodes to communicate. Nodes can order transactions BEFORE consensus, making everything faster.

**2. Tower BFT**

Solana's consensus mechanism, built on top of Proof of History. Validators vote on blocks, and PoH timestamps make it easy to verify vote ordering.

**3. Gulf Stream**

```
  TRADITIONAL MEMPOOL vs SOLANA GULF STREAM:

  Traditional (Bitcoin/Ethereum):        Solana (Gulf Stream):

  Tx --> [  MEMPOOL  ] --> Miner         Tx --> [Next block producer]
         (waiting room)    picks it             (sent directly!)
         Can get backed up                      No waiting room needed

  Like waiting in line at                Like skipping the line --
  a restaurant                          your order goes straight
                                        to the chef
```

Transactions are forwarded to the expected next block producer BEFORE the current block is finalized. This eliminates the mempool bottleneck that slows Bitcoin and Ethereum.

### Solana's trade-offs

| Pro | Con |
|-----|-----|
| Extremely fast (~400ms blocks) | Has had multiple network outages (downtime) |
| Near-zero fees | Requires beefy hardware to run a validator |
| Growing ecosystem (DeFi, NFTs, DePIN) | Less decentralized than Ethereum (~1,800 validators vs Ethereum's ~900,000) |
| Great developer experience | Younger ecosystem, less battle-tested |

### Major Solana projects

- **Jupiter** -- DEX aggregator (largest on Solana)
- **Marinade Finance** -- Liquid staking
- **Magic Eden** -- NFT marketplace
- **Helium** -- Decentralized wireless network (DePIN)
- **Tensor** -- NFT trading platform

---

# Module 5: Tokens -- Creating Digital Assets

---

## 38. What is a Token?

A **token** is a digital asset created on an existing blockchain using a smart contract. It's different from a **coin**:

```
  COIN vs TOKEN:

  COIN (has its own blockchain)           TOKEN (lives on another blockchain)

  +------ Bitcoin Blockchain ------+      +------ Ethereum Blockchain ------+
  |                                |      |                                 |
  |    BTC (native coin)           |      |    ETH (native coin)            |
  |                                |      |    USDT (token)                 |
  +--------------------------------+      |    UNI  (token)                 |
                                          |    SHIB (token)                 |
  +------ Ethereum Blockchain -----+      |    LINK (token)                 |
  |                                |      |    ... thousands more           |
  |    ETH (native coin)           |      +---------------------------------+
  |                                |
  +--------------------------------+      All tokens are smart contracts
                                          deployed ON the Ethereum blockchain.
  +------ Solana Blockchain -------+      They don't have their own network.
  |                                |
  |    SOL (native coin)           |
  |                                |
  +--------------------------------+
```

| | Coin | Token |
|---|---|---|
| **Has its own blockchain?** | Yes | No (lives on another chain) |
| **Examples** | BTC, ETH, SOL | USDT, LINK, UNI, SHIB |
| **How it's created** | Build an entire blockchain | Deploy a smart contract (minutes) |
| **Cost to create** | Millions of dollars | $5-50 in gas fees |

Anyone can create a token. This is both powerful (democratizes finance) and dangerous (enables scam tokens).

---

## 39. Token Standards

### ERC-20 (Ethereum -- Fungible Tokens)

The most common token standard. **Fungible** means every token is identical and interchangeable (like dollars -- one $1 bill equals any other $1 bill).

**Examples:** USDT, USDC, LINK, UNI, SHIB, DAI

**What the smart contract defines:**
- `name` -- "Tether USD"
- `symbol` -- "USDT"
- `totalSupply` -- how many tokens exist
- `balanceOf(address)` -- check anyone's balance
- `transfer(to, amount)` -- send tokens
- `approve(spender, amount)` -- allow another contract to spend your tokens

**How to create one:** You write a Solidity smart contract implementing the ERC-20 interface and deploy it to Ethereum. The contract tracks who owns how many tokens. It can be done in ~50 lines of code.

### ERC-721 (Ethereum -- Non-Fungible Tokens / NFTs)

```
  FUNGIBLE (ERC-20) vs NON-FUNGIBLE (ERC-721):

  FUNGIBLE (every unit is identical):     NON-FUNGIBLE (every unit is unique):

  +-----+  +-----+  +-----+              +--------+  +--------+  +--------+
  |  $1 |  |  $1 |  |  $1 |              | Ape    |  | Ape    |  | Ape    |
  |     |  |     |  |     |              | #1234  |  | #5678  |  | #9999  |
  +-----+  +-----+  +-----+              | Gold   |  | Laser  |  | Crown  |
                                          | fur    |  | eyes   |  | hat    |
  Any $1 = any other $1                   +--------+  +--------+  +--------+
  (interchangeable)
                                          Each one is different and
  Like dollars, USDT, UNI                 has different value.
                                          Like art, real estate, tickets.
```

**Non-fungible** means each token is unique. Token #1 is different from Token #2.

**Examples:** Bored Ape Yacht Club, CryptoPunks, Art Blocks

**Use cases beyond art:**
- Real estate deeds
- Event tickets
- Domain names (ENS)
- Gaming items
- Academic certificates

**What makes it different from ERC-20:**
- Each token has a unique `tokenId`
- Each token can point to different metadata (image, attributes)
- You can't send "0.5 of an NFT" -- it's indivisible

### ERC-1155 (Multi-Token Standard)

Combines fungible and non-fungible tokens in a single contract. Used heavily in gaming where you might have:
- 1000 identical gold coins (fungible)
- 1 unique legendary sword (non-fungible)

### SPL Tokens (Solana)

```
  ETHEREUM TOKENS vs SOLANA TOKENS:

  Ethereum:                               Solana:
  Each token = separate contract          All tokens = ONE shared program

  +----------+  +----------+              +---------------------------+
  | USDT     |  | UNI      |              | SPL Token Program         |
  | Contract |  | Contract |              | (pre-deployed on Solana)  |
  +----------+  +----------+              |                           |
  +----------+  +----------+              | Mint A (USDC)             |
  | LINK     |  | SHIB     |              | Mint B (BONK)             |
  | Contract |  | Contract |              | Mint C (your token)       |
  +----------+  +----------+              +---------------------------+

  Deploy new contract each time            Just create a new "mint"
  (~$50-500 in gas)                        (~$0.01)
```

Solana's equivalent of ERC-20. The **SPL Token Program** is a pre-deployed program on Solana that handles token creation and transfers.

**Key difference from Ethereum:** On Ethereum, every token is a separate smart contract. On Solana, all tokens share the same program -- you just create a new "mint" (token definition).

**How to create an SPL token:**
1. Use the `spl-token` CLI or a tool like Metaplex
2. Create a mint (defines the token)
3. Create token accounts for holders
4. Mint tokens to those accounts

It costs less than $0.01 to create a token on Solana.

---

## 40. Stablecoins

**Stablecoins** are tokens pegged to a real-world asset (usually $1 USD).

| Stablecoin | Peg | How it maintains peg | Market cap |
|-----------|-----|---------------------|------------|
| **USDT (Tether)** | $1 | Backed by reserves (cash, bonds, etc.) | ~$100B+ |
| **USDC (Circle)** | $1 | Backed by cash and US treasuries (audited) | ~$30B+ |
| **DAI (MakerDAO)** | $1 | Backed by crypto collateral (over-collateralized) | ~$5B |
| **UST (Terra)** | $1 | Algorithmic (no real backing) | **Collapsed to $0 in May 2022** |

**Why stablecoins matter:**
- Bridge between crypto and fiat (real money)
- Used for trading (buy the dip without converting to USD)
- Payments and remittances (send $1000 across borders for $0.01)
- DeFi (lending, borrowing, yield farming)

**The UST/Luna collapse** (May 2022) destroyed $40 billion in value and proved that algorithmic stablecoins without real backing are extremely risky.

---

## 41. Altcoins

**Altcoin** = any cryptocurrency that's not Bitcoin. "Alternative coin."

### Categories

```
  THE BLOCKCHAIN ECOSYSTEM (Layer 1 vs Layer 2):

  +================================================================+
  |                        LAYER 1 (Base chains)                    |
  |                                                                |
  |  +----------+    +-----------+    +--------+    +----------+   |
  |  | Bitcoin   |    | Ethereum  |    | Solana |    | Avalanche|   |
  |  | (BTC)    |    | (ETH)     |    | (SOL)  |    | (AVAX)   |   |
  |  +----------+    +-----+-----+    +--------+    +----------+   |
  |                        |                                        |
  +========================|========================================+
                           |
  +========================|========================================+
  |                  LAYER 2 (Built on top of Ethereum)             |
  |                        |                                        |
  |  +-----------+   +-----+-----+   +-----------+   +-----------+ |
  |  | Arbitrum  |   | Optimism  |   | Base      |   | Polygon   | |
  |  | (ARB)     |   | (OP)      |   | (Coinbase)|   | (MATIC)   | |
  |  +-----------+   +-----------+   +-----------+   +-----------+ |
  |                                                                |
  |  Same security as Ethereum, but 10-100x cheaper and faster     |
  +================================================================+
```

| Category | Examples | What they do |
|----------|---------|-------------|
| **Smart contract platforms** | ETH, SOL, ADA, AVAX | Run decentralized applications |
| **DeFi tokens** | UNI, AAVE, CRV | Governance for DeFi protocols |
| **Stablecoins** | USDT, USDC, DAI | Pegged to $1 |
| **Meme coins** | DOGE, SHIB, PEPE, BONK | Community/hype driven, no utility |
| **Layer 2** | ARB, OP, MATIC | Scale Ethereum with cheaper transactions |
| **Infrastructure** | LINK, GRT, FIL | Oracles, indexing, storage |
| **DePIN** | HNT, RNDR, AKT | Decentralized physical infrastructure |

---

# Module 6: Essential Vocabulary

---

## 42. Terms You Must Know

This is your quick-reference glossary. If you can explain all of these, you can hold your own in any blockchain conversation.

| Term | Definition |
|------|-----------|
| **Blockchain** | A distributed, immutable ledger of transactions linked by cryptographic hashes |
| **Block** | A batch of transactions bundled together with a hash and timestamp |
| **Hash** | A fixed-size fingerprint of any data, produced by a one-way function (SHA-256) |
| **Nonce** | A number miners change to find a valid block hash |
| **Consensus** | How nodes agree on the state of the blockchain |
| **PoW (Proof of Work)** | Consensus via computational puzzle solving (Bitcoin) |
| **PoS (Proof of Stake)** | Consensus via staking coins as collateral (Ethereum) |
| **PoH (Proof of History)** | Cryptographic clock for ordering events (Solana) |
| **Mining** | Using computing power to find valid blocks (PoW) |
| **Staking** | Locking coins to become a validator (PoS) |
| **Validator** | A node that proposes and verifies blocks in PoS |
| **Node** | A computer running blockchain software |
| **Wallet** | Software that stores your private keys |
| **Private key** | The secret that controls your crypto (never share) |
| **Public key** | Your address for receiving crypto |
| **Seed phrase** | 12/24 words that can recover all your keys |
| **Smart contract** | A program on the blockchain that executes automatically |
| **DApp** | An app with its backend on a blockchain |
| **DAO** | An organization governed by smart contracts and token votes |
| **EVM** | Ethereum's virtual machine that runs smart contracts |
| **Gas** | The fee for computation on Ethereum |
| **Gwei** | 0.000000001 ETH (unit for gas prices) |
| **Token** | A digital asset created on an existing blockchain |
| **ERC-20** | Ethereum standard for fungible tokens |
| **ERC-721** | Ethereum standard for NFTs |
| **SPL Token** | Solana's token standard |
| **NFT** | A unique, non-fungible token (digital ownership proof) |
| **DeFi** | Decentralized Finance (banking without banks) |
| **DEX** | Decentralized exchange (trade without a company) |
| **CEX** | Centralized exchange (Coinbase, Binance) |
| **TVL** | Total Value Locked (how much money is in a DeFi protocol) |
| **Stablecoin** | A token pegged to a real-world asset (usually $1) |
| **Liquidity** | How easily an asset can be bought/sold |
| **Slippage** | Price change between when you submit a trade and when it executes |
| **Yield farming** | Earning rewards by providing liquidity to DeFi protocols |
| **Airdrop** | Free tokens distributed to wallet addresses (marketing/reward) |
| **Whitepaper** | A document explaining a crypto project's technology and vision |
| **Mainnet** | The live, production blockchain |
| **Testnet** | A test version of a blockchain (fake money, for development) |
| **Layer 1 (L1)** | The base blockchain (Ethereum, Solana, Bitcoin) |
| **Layer 2 (L2)** | A scaling solution built on top of L1 (Arbitrum, Optimism) |
| **Rollup** | An L2 that bundles transactions and posts proof to L1 |
| **Bridge** | A tool to move assets between different blockchains |
| **Oracle** | A service that brings real-world data to smart contracts (Chainlink) |
| **Mempool** | Where unconfirmed transactions wait |
| **UTXO** | Unspent Transaction Output (Bitcoin's accounting model) |
| **Hard fork** | A backwards-incompatible blockchain upgrade (can split the chain) |
| **Soft fork** | A backwards-compatible blockchain upgrade |
| **51% attack** | When one entity controls majority of mining/staking power |
| **Rug pull** | A scam where developers abandon a project and take investor funds |
| **HODL** | "Hold On for Dear Life" -- holding crypto long-term despite volatility |
| **WAGMI** | "We're All Gonna Make It" -- community optimism |
| **DYOR** | "Do Your Own Research" -- don't blindly trust crypto advice |
| **GM** | "Good Morning" -- crypto community greeting |
| **Whale** | Someone holding a very large amount of crypto |
| **FUD** | "Fear, Uncertainty, and Doubt" -- negative sentiment (sometimes manufactured) |
| **ATH** | "All-Time High" -- the highest price an asset has ever reached |
| **Market cap** | Total value of all coins in circulation (price x supply) |

---

## Key Concepts Recap

- **Blockchain** -- a chain of blocks linked by hashes, distributed across thousands of computers, making data tamper-proof without needing trust in any single party.

- **Hashing (SHA-256)** -- the cryptographic foundation. One-way, deterministic, and any tiny change in input produces a completely different output. This makes tampering detectable.

- **Consensus** -- how thousands of strangers agree on truth. PoW uses electricity (Bitcoin), PoS uses economic stakes (Ethereum), PoH uses cryptographic time (Solana).

- **Bitcoin** -- digital money with a fixed supply of 21 million, secured by Proof of Work. Transactions use the UTXO model and are processed through the mempool.

- **Ethereum** -- a programmable blockchain where smart contracts enable DApps, DeFi, DAOs, and NFTs. The EVM runs code, and gas prevents abuse.

- **Solana** -- a high-speed blockchain using Proof of History for sub-second block times and near-zero fees, trading some decentralization for performance.

- **Tokens** -- digital assets created on existing blockchains. ERC-20 for fungible tokens, ERC-721 for NFTs, SPL for Solana tokens. Anyone can create one in minutes.

- **Security** -- your private key and seed phrase are everything. "Not your keys, not your coins." Never share them. There is no password reset in crypto.

---

## What's Next

If this class made you curious about the coding side:
- **Solidity** -- learn to write Ethereum smart contracts (Code Eater has a full course)
- **Rust** -- the language for Solana development
- **Hardhat / Foundry** -- development frameworks for testing and deploying contracts
- **Ethers.js / web3.js** -- JavaScript libraries to interact with Ethereum from your React apps

The knowledge you learned today is the foundation. Every blockchain developer needs this mental model before writing their first line of contract code.
