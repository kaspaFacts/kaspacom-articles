# Intro Level - Transaction Selection and Weighting

Frontier, Feerate Weighting, Random Sampling - what does it all mean?

Frontier and Feerate Weighting are mempool terms, while Random Sampling is used for block template creation.

Transaction selection describes how Kaspa chooses which transactions to include in blocks.  Kaspa uses two distinct strategies: when the mempool has **≤** one block's worth of transactions, it takes everything.  When the mempool has > one block's worth of transactions, it switches to weighted random selection to prevent collisions.

## Traditional Blockchain Transaction Selection

Traditional blockchains like Bitcoin use deterministic fee-rate ordering where transactions are selected by fee-per-byte ratio in descending order.  Miners simply pick the highest-paying transactions first until the block is full, creating predictable and straightforward inclusion logic.  When blocks reach capacity, lower fee-rate transactions wait for the next block opportunity.

## Kaspa's Mempool Architecture

Kaspa's mempool organizes transactions into different pools based on their readiness for inclusion.

### Frontier - Ready Transactions

The Frontier contains transactions that have no mempool dependencies and are ready for immediate block inclusion.  Only transactions in the Frontier can be selected for blocks - if your transaction depends on another unconfirmed transaction, it must wait until that transaction is included first.

### Feerate Weighting Formula

Kaspa calculates transaction weights using (feerate^3) where feerate equals (fee/mass).  This cubic weighting creates exponential bias toward higher-feerate transactions.  A transaction paying 2x the feerate gets 8x the selection weight (2^3 = 8).

### Dynamic Selection Strategies

Kaspa uses different selection methods based on Frontier size:

**Small Frontier (≤ block capacity): Complete Selection** - includes every transaction since they all fit.

**Large Frontier (> block capacity): Weighted Random Selection** - uses probability-based sampling with (feerate^3) weighting.  Higher feerate transactions have exponentially higher selection probability, and selection becomes probabilistic rather than deterministic.

### Probability Mathematics

The selection probability for any transaction equals: (transaction weight / total weight) where (total weight) is the sum of all Frontier transaction weights.

**Small mempool example**: With 1000 transactions, doubling your feerate significantly increases your probability since the denominator (total_weight) is relatively small.

**Large mempool example**: With 100,000 transactions, the total_weight becomes enormous, so you need much higher feerates to achieve the same probability improvement.

### Why Probabilistic Selection is Necessary

In Kaspa's DAG structure, multiple miners can work on blocks simultaneously.  When different miners include the same transaction in their blocks, this creates a "collision" - this has negative consequences for everyone.

**The Collision Problem**: If all miners used identical deterministic selection (picking transactions by feerate), they would constantly include the same transactions as other miners.  This means most miners would waste computational effort including transactions they won't get paid for, users would receive a lower quality of service, and the DAG's capacity would be underutilized since the same transactions appear in multiple blocks instead of unique transactions.

**Why Probabilistic Selection Helps**: By using weighted random sampling, different miners are likely to select different sets of transactions.  This reduces collisions between miners, improves miner revenue (since they're more likely to get unique transactions), improves the quality of service received by users, and increases effective DAG throughput by filling blocks with diverse transaction sets rather than duplicates.

## Simplified Definitions

**Frontier** - Transactions ready for immediate block inclusion with no mempool dependencies.

**Feerate** - Fee per unit mass ratio (fee/mass) measured in sompi/gram.

**Transaction Weight** - Feerate raised to the power of 3 (feerate^3).

**Selection Probability** - Transaction weight divided by total Frontier weight.

**Mempool** - The pool of unconfirmed transactions waiting to be included in blocks.

**Block Capacity** - The maximum mass limit that can fit in a single block.

**Mass** - Transaction size measurement in Kaspa (replaces "bytes" from Bitcoin).

**Sompi** - Smallest unit of KAS currency (like satoshi for Bitcoin).

**DAG** - Directed Acyclic Graph structure that allows multiple concurrent blocks.

**Collision** - When multiple miners include the same transaction in their blocks, reducing miner revenue, user quality of service, and network efficiency.

## Bitcoin and Kaspa

**Bitcoin** - Uses simple feerate ordering where higher-fee transactions get priority.  Selection is deterministic based on fee-per-byte ratios.  No sophisticated weighting or probability calculations.

**Kaspa** - Uses dynamic selection strategies, cubic feerate weighting, and probabilistic sampling.  The system adapts its selection method based on mempool size and maintains efficiency even with millions of transactions.  Kaspa's approach creates steeper competition curves where small feerate differences become amplified in crowded mempools.