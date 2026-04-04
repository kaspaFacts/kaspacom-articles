## What is Past Median Time?

Past Median Time is a Kaspa Consensus Mechanism that calculates a Representative Timestamp from a Window of historical blocks to [validate new block timestamps](https://kaspa.com/learn-kaspa/post/timestamp-validation). It uses a Sampled Window approach, ensuring blocks cannot have timestamps too far in the past while maintaining consistent behavior regardless of network [block rate](https://kaspa.com/learn-kaspa/post/blocks-per-second).

## The Challenge: Timestamp Validation in High-Throughput Networks

In a [BlockChain](https://kaspa.com/learn-kaspa/post/dag-terminology) network, proper Timestamp Validation is essential for maintaining Security and preventing timestamp manipulation attacks. Kaspa's high block rate creates specific challenges for Timestamp Management.

Traditional Median Time calculations Scale Linearly with block rate, causing inconsistent behavior across different network configurations. This becomes problematic as the Kaspa protocol supports different [parameter configurations](https://kaspa.com/learn-kaspa/post/ghostdag-k-cluster) for various deployment scenarios.

**Timestamp Manipulation**: Without proper Median Time Validation, malicious actors could submit blocks with significantly skewed timestamps, potentially disrupting Difficulty Adjustment and Network Synchronization.

**Configuration Inconsistency**: Different parameter configurations operating at various block rates make it difficult to maintain consistent Timestamp Validation Rules across all deployment scenarios.

**Performance Bottlenecks**: Calculating Median Time from large windows of historical blocks can become Computationally Expensive, especially at high block rates where windows would contain [thousands of blocks](https://kaspa.com/learn-kaspa/post/kaspa-and-the-bitcoin-scaling-problem).

Past Median Time solves these problems by using a Sampled Window approach that maintains constant window size regardless of block rate, providing consistent validation rules while optimizing performance.

## How Past Median Time Works

### Sampled Window Construction

The system builds windows by traversing the [Selected Parent Chain](https://kaspa.com/learn-kaspa/post/parent-chain) backwards and sampling blocks at regular intervals based on their [DAA Score](https://kaspa.com/learn-kaspa/post/daa-score).

- **Window Size**: A fixed-size window that maintains consistent behavior across different network configurations
- **Sample Rate**: Blocks sampled at regular intervals based on network block rate
- **Sample Interval**: Consistent time periods between samples to ensure even distribution

Blocks are sampled at regular positions within the network's difficulty sequence, ensuring even distribution across the window.

### Median Calculation Algorithm

The implementation averages a small group of central values from sorted timestamps to create a representative timestamp, providing smoother results than a true median.

For example, with timestamps [100, 105, 110, 115, 120, 125, 130, 135, 140, 145, 150], the algorithm would average the central values to get a representative timestamp.

### Window Building Process

The window construction follows a systematic approach to gather representative historical timestamps.

1. Start from the Selected Parent and traverse backwards through the chain
2. Sample blocks at regular positions within the network's difficulty sequence
3. Collect blocks organized by their chain importance, ensuring the window contains the most significant blocks
4. Stop when window reaches its target size, maintaining consistent validation across all network configurations
5. Optionally merge with cached parent windows for efficiency to optimize performance

## How the Network Uses This

### Header Validation

Past Median Time is validated during header processing to ensure new blocks have timestamps greater than the calculated median.

Blocks with timestamps less than or equal to the Past Median Time are rejected, preventing timestamp manipulation attacks.

This validation is essential for maintaining network Security and preventing Difficulty Adjustment Manipulation.

### Virtual State Calculation

The Virtual Processor calculates Past Median Time when computing the [Virtual State](https://kaspa.com/learn-kaspa/post/virtual-block), which represents the current tip of the [BlockDAG](https://kaspa.com/learn-kaspa/post/dag-and-kaspa).

The calculation uses a smoothed average of central timestamps from the sorted window values

This ensures the Virtual State always has a valid timestamp reference for subsequent Block Validation.

### Network Parameter Configuration

Different Kaspa deployment configurations use different parameters based on their block rate configuration.

Each configuration uses appropriate Window Sizes and Sample Intervals to maintain consistent Security Guarantees.

## Why This Matters

### Consistent Security Across Networks

The Sampled Window approach provides identical Security Guarantees regardless of the network's block rate configuration.

### Performance Optimization

By limiting windows to a fixed size instead of thousands, the system reduces computational overhead while maintaining accurate Timestamp Validation.

### Attack Prevention

Strict Timestamp Validation prevents attackers from manipulating block timestamps to influence Difficulty Adjustment or disrupt Network Synchronization.

## Key Takeaways

- **Sampled Window Approach** - Uses sampled windows instead of full historical windows for constant size regardless of block rate
- **Representative Average** - Calculates average of central timestamps to create a representative value rather than using true median
- **Ancestor Traversal** - Builds windows by walking backward through Selected Parent Chain with position-based sampling
- **Strict Validation** - Block timestamps must be strictly greater than past median time (not equal)
- **Caching Optimization** - Uses aggressive caching to avoid redundant window calculations across related blocks