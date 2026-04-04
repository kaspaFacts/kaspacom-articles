# DAA Score: Kaspa's Timing Metric for Difficulty Adjustment

## What is DAA Score?

DAA Score is a cumulative counter that tracks the total number of DAA-eligible blocks in a block's entire history, starting from Genesis. While the [Difficulty Adjustment Window](https://kaspa.com/learn-kaspa/post/daa-window) uses a subset of recent blocks for calculations, DAA Score itself accumulates all DAA-eligible blocks ever processed. serving as a timing mechanism for maintaining consistent block intervals. Unlike traditional blockchains that use wall-clock time for both timing and calculation, DAA Score uses block count to determine when to adjust difficulty, while still using timestamps from DAA-eligible blocks to calculate the adjustment magnitude based on actual block production rate.

## The Challenge: Timing Difficulty Adjustments in a DAG

In a [BlockDAG](https://kaspa.com/learn-kaspa/post/dag-and-kaspa) structure where multiple blocks are created simultaneously, traditional time-based difficulty adjustment breaks down because blocks don't have a clear linear ordering. This creates fundamental challenges for maintaining consistent block times in a decentralized network.

Kaspa's parallel block production means blocks can be created at different rates across multiple branches, making it impossible to rely on timestamps alone for difficulty calculations. The network needs a way to measure "network time" that doesn't depend on wall-clock time, which can vary between nodes and be manipulated by miners.

**Timing Ambiguity**: Without a reliable timing mechanism, nodes cannot agree on when to adjust difficulty, potentially leading to blocks that are too easy or too hard.

**Window Selection Complexity**: In a DAG with parallel blocks, how should the network identify which recent blocks should influence the next difficulty calculation when multiple valid paths exist?

**Consensus Breaks**: If nodes use different time windows for difficulty calculations, they'll compute different difficulty targets, potentially splitting the network and breaking Consensus.

DAA Score solves these problems by using [Blue Score](https://kaspa.com/learn-kaspa/post/blue-score-and-blue-work) as a proxy for time to create a sliding window of DAA-eligible blocks, providing a consistent timing mechanism that all nodes can agree on without relying on external time sources.

## How DAA Score Works

### The Blue Score Dependency

DAA Score relies on Blue Score to determine which blocks are recent enough to influence difficulty calculations. This dependency creates a sliding window that ensures only blocks with sufficiently high Blue Scores participate in DAA calculations.

- **Blue Score Threshold**: Calculated as current block's Blue Score minus the difficulty window size
- **DAA-Eligible Blocks**: All blocks in the [Mergeset](https://kaspa.com/learn-kaspa/post/parents-vs-mergeset) with Blue Scores above the threshold
- **Non-DAA Blocks**: Blocks with Blue Scores below the threshold, excluded from difficulty calculations

The threshold calculation ensures that as the network progresses, older blocks naturally fall out of the difficulty window, creating a time-based sliding effect without using actual timestamps.

### DAA Score Calculation

DAA Score is calculated as the [Selected Parent's](https://kaspa.com/learn-kaspa/post/ghostdag-simplified) DAA score plus the number of new DAA-eligible blocks in the current Mergeset. This cumulative approach means DAA Score always represents the total count of difficulty-relevant blocks in a block's history.

For example, if a block's Selected Parent has a DAA Score of 1000 and the current Mergeset contains 5 DAA-eligible blocks, the new block's DAA Score becomes 1005.

### Score Calculation Process

1. Calculate the Blue Score threshold using the current block's Blue Score
2. Identify non-DAA blocks by comparing each Mergeset block's Blue Score against the threshold
3. Count DAA-eligible blocks (total Mergeset Size minus non-DAA blocks)
4. Add this count to the Selected Parent's DAA Score to get the new DAA Score

## How the Network Uses This

### Difficulty Adjustment Timing

DAA Score serves as the network's primary timing mechanism for difficulty adjustments. The difficulty algorithm uses DAA Score to identify the appropriate window of blocks for calculating the next difficulty target.

When DAA Score reaches predetermined adjustment points, the network recalculates mining difficulty based on the actual block production rate within the DAA Window. This ensures blocks aren't too easy or too hard, maintaining [Target Block Time](https://kaspa.com/learn-kaspa/post/ghostdag-k-cluster).

This approach provides more stable difficulty adjustments than time-based systems because it uses block count for timing while still measuring actual block production rate through timestamps to calculate the adjustment magnitude.

### Transaction Lock Times

DAA Score provides a reliable timestamp alternative for transaction lock times. Instead of using wall-clock time, transactions can specify either a DAA Score or a timestamp at which they become valid.

This is particularly useful for time-locked transactions that should only be spendable after a certain network state and smart contracts that depend on network progression rather than absolute time.

### Network Health Monitoring

DAA Score growth rate serves as an indicator of network health and block production speed. A steadily increasing DAA Score indicates healthy network activity, while stagnation could signal network problems.

## Why This Matters

### Consistent Block Times

DAA Score enables Kaspa to maintain consistent block intervals regardless of network hash rate changes. By using block count rather than time for difficulty adjustments, the network automatically adapts to mining power fluctuations, ensuring predictable block production for users and applications.

### Resistance to Time Manipulation

Since DAA Score uses block count for timing decisions, miners cannot manipulate difficulty by reporting false times. The Blue Score dependency creates a timing mechanism that's tied to actual Network Consensus State, making it much more resistant to manipulation attacks.

### Decentralized Timekeeping

DAA Score provides a Decentralized way to measure network time that all nodes can agree on without central coordination or trusted time sources. This is essential for maintaining Consensus in a distributed system where nodes cannot rely on external time references.

## Key Takeaways

- **DAA Score Times Difficulty** - DAA Score is a block count used to determine when to adjust mining difficulty, not a measure of work
- **Blue Score Enables DAA** - Blue Score determines which blocks are DAA-eligible by creating a time-based threshold
- **Sliding Window Mechanism** - The Blue Score threshold creates a natural sliding window that excludes old blocks from difficulty calculations
- **Consensus Without Clocks** - DAA Score enables decentralized timekeeping without relying on wall-clock time for timing decisions
- **Practical Applications** - Beyond difficulty adjustment, DAA Score is used for transaction lock times and network health monitoring