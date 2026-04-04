# Intermediate Level - Transaction Selection Strategies in Kaspa

Transaction selection in Kaspa is about choosing the right algorithm for the right congestion level. The system dynamically adapts its selection strategy based on matching the selection complexity to the actual need.

## The Three-Strategy Framework

Kaspa uses three distinct transaction selection strategies, each optimized for different mempool congestion scenarios:

### 1. Complete Selection (Low Congestion)

**When**: Mempool mass ≤ block capacity<br>**Strategy**: Take all available transactions<br>**Reasoning**: When you can fit everything, selection becomes unnecessary overhead. The optimal strategy is simply to include every transaction, maximizing throughput without any computational cost for prioritization.

### 2. Probabilistic Weighted Selection (Moderate Congestion)

**When:** Mempool mass is 1-4x block capacity<br>**Strategy:** Random weighted selection using (feerate ^ 3) weighting, performed outside the mempool lock<br>**Reasoning:** With moderate transaction volumes, the system uses a selection strategy to perform probability-based sampling.  This strategy copies all frontier transactions out of the mempool first, then performs weighted random sampling without holding the lock, optimizing for both fairness and performance.  Higher-feerate transactions have exponentially higher selection probability, but selection remains probabilistic rather than deterministic.

### 3. Probabilistic Weighted Sampling (Heavy Congestion)

**When:** Mempool mass > 4x block capacity<br>**Strategy:** In-place random sampling with (feerate ^ 3) probability weighting, performed inside the mempool lock<br>**Reasoning:** With massive transaction volumes, the system uses a selection strategy to perform weighted random sampling directly on the locked mempool data structure.  This in-place approach is efficient because collision rates are low when there are many transactions relative to the sample size.  The algorithm handles collisions by narrowing the sampling space and uses sophisticated collision detection to maintain performance even with millions of transactions.

## The Economic Logic

All strategies use (feerate ^ 3) weighting, creating exponential preference for higher-paying transactions.  This ensures that:

- **Economic incentives remain strong**: Users who pay more get prioritized
- **Network efficiency is maintained**: Higher fees signal higher transaction urgency
- **Miner revenue is optimized**: Fee-maximizing selection benefits network security

## The Performance Principle

The 4x threshold represents an inflection point where collision probability mathematics favor different approaches:

- **Below 4x**: Selection overhead > sampling efficiency gains
- **Above 4x**: Sampling efficiency gains > selection overhead

This ensures sub-150μs selection times regardless of mempool size.

## Why This Approach Works

**Adaptive Complexity**: The system only uses complex algorithms when necessary, avoiding over-engineering for simple scenarios.

**Economic Consistency**: All strategies maintain feerate-based prioritization, ensuring users' economic incentives remain predictable.

**Performance Scalability**: Selection time remains constant even as the network grows from thousands to millions of transactions.

**Fairness Balance**: Deterministic selection provides fairness in normal conditions, while probabilistic selection prevents system degradation under extreme load.

## The Result

This three-strategy framework ensures that Kaspa's transaction selection remains both economically rational and efficient across all network conditions.  Users paying higher fees get prioritized, miners fill blocks with the highest chance of getting paid the most, and the system is optimized so transaction selection is not a bottleneck.

### **Definitions and Concepts**

### Core Transaction Selection Terms:

- **Frontier** - The set of transactions with no mempool dependencies, ready for block inclusion
- **Feerate** - Fee per unit mass ratio (fee/mass) measured in sompi/gram
- **Mempool** - The pool of unconfirmed transactions waiting to be included in blocks
- **Mempool Lock** - Read/write lock protecting concurrent access to mempool data structures
- **Block Capacity** - The maximum mass limit that can fit in a single block, used as the threshold for strategy selection
- **Block Template** - The structure containing selected transactions and metadata used for mining new blocks

### Technical Implementation Terms:

- **Transaction Weight** - Feerate raised to the power of 3 (feerate^3)
- **Selection Probability** - Transaction weight divided by total frontier weight
- **Mass** - Transaction size measurement in Kaspa (replaces "bytes" from Bitcoin)
- **Sompi** - Smallest unit of KAS currency (like satoshi for Bitcoin)
- **In-Place Sampling** - Weighted random selection performed directly within the locked mempool
- **Collision** - When the same transaction gets randomly selected multiple times during sampling
- **Collision Detection** - The algorithm that identifies when the same transaction is randomly selected multiple times
- **Sampling Space** - The range of weighted transactions from which random selection occurs
- **Selection Overhead** - The computational cost of choosing which transactions to include
- **Consensus Rejections** - When transactions fail validation during block template creation, requiring the system to sample extra transactions

### Strategy-Specific Terms:

- **TakeAllSelector** - Deterministic strategy that includes all transactions when they fit
- **RebalancingWeightedTransactionSelector** - Medium congestion strategy using copied transactions
- **SequenceSelector** - Heavy congestion strategy using in-place sampling
- **Collision Factor** - The 4x threshold determining strategy selection
- **Template Transaction Selector** - The interface that different selection strategies implement to provide transactions for block templates