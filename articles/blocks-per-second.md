# Kaspa's BPS Configuration

## What is BPS Configuration?

BPS (Blocks Per Second) Configuration is Kaspa's system for scaling all consensus parameters based on the network's block production rate. It ensures that security parameters like finality depth, merge depth, and GHOSTDAG K scale proportionally when the network transitions from 1 block per second to 10 blocks per second.

## The Challenge: Maintaining Security Across Different Block Rates

When you increase a blockchain's block production rate, you can't simply speed up block creation without adjusting security parameters.

Kaspa's Crescendo hardfork increased the block rate from 1 BPS to 10 BPS. Without proper parameter scaling, this would create severe security and operational problems.

**Broken Time-Based Security**: Security parameters like finality depth (12 hours) and merge depth (1 hour) are defined in time units, not block counts. At 1 BPS, 12 hours = 43,200 blocks. At 10 BPS, 12 hours = 432,000 blocks. Without scaling, the network would use the wrong depth values.

**Insufficient Anticone Tolerance**: The GHOSTDAG K parameter must scale with block rate to maintain the same probability of anticones exceeding K. At 1 BPS, K=18 provides 99% confidence. At 10 BPS, K must increase to 124 to maintain the same security level.

**Incompatible DAA Windows**: The difficulty adjustment window must cover the same time duration regardless of BPS. At 1 BPS, this requires 2,641 blocks. At 10 BPS with sampling, this requires 661 sampled blocks (every 40 blocks).

BPS Configuration solves all these problems by providing a unified system that calculates all consensus parameters as functions of the BPS value, ensuring security properties remain constant across different block rates.

## How BPS Configuration Works

### Time-Based Parameter Scaling

BPS Configuration converts time-based security parameters into block counts by multiplying the time duration (in seconds) by the BPS value.

- **Finality Depth**: BPS × 43,200 seconds (12 hours) = 432,000 blocks at 10 BPS
- **Merge Depth**: BPS × 3,600 seconds (1 hour) = 36,000 blocks at 10 BPS
- **Target Time Per Block**: 1000 milliseconds / BPS = 100ms at 10 BPS

This ensures that security guarantees remain constant in real-world time, regardless of how fast blocks are produced.

### GHOSTDAG K Calculation

The K parameter is calculated using the formula from the GHOSTDAG paper to ensure anticones larger than K occur with less than 1% probability.

The calculation uses x = 2Dλ where D is network delay (5 seconds) and λ is the block rate. At 10 BPS: x = 2 × 5 × 10 = 100, which yields K=124.

### Derived Parameter Calculation

Once BPS and K are known, other parameters are calculated as functions of these values.

1. Mergeset Size Limit = 2 × K (capped between 180-512) = 248 at 10 BPS
2. Max Block Parents = K / 2 (capped between 10-16) = 16 at 10 BPS
3. Pruning Depth = max(anticone_finalization_depth, BPS × 108,000 seconds)
4. Sample Rates = BPS × sample_interval for DAA and median time windows

## How the Network Uses This

### Fork Activation and Parameter Selection

The network uses ForkedParam structures to maintain both pre-Crescendo (1 BPS) and post-Crescendo (10 BPS) parameter values.

When processing a block, the consensus system checks the DAA score to determine which parameter set to use. If the DAA score is above the Crescendo activation threshold, 10 BPS parameters apply; otherwise, 1 BPS parameters apply.

This allows the network to transition smoothly from 1 BPS to 10 BPS at a specific block without requiring all nodes to upgrade simultaneously.

### Anticone Finalization Depth Formula

BPS Configuration enables the anticone finalization depth calculation, which determines when blocks can be safely pruned.

anticone_finalization_depth = finality_depth + merge_depth + (4 × mergeset_size_limit × k) + (2 × k) + 2

At 10 BPS: 432,000 + 36,000 + (4 × 248 × 124) + (2 × 124) + 2 = 591,194 blocks. This represents approximately 16.4 hours of blocks, ensuring sufficient depth for safe pruning.

### Simulation and Testing

The simpa simulator uses BPS Configuration to test consensus at different block rates. It can simulate any BPS value by calculating appropriate K values and scaling all time-based parameters accordingly.

For example, simpa can test 8 BPS by calculating K=102 and scaling finality depth to 345,600 blocks. This allows developers to verify that security properties hold at different block rates.

## Why This Matters

### Enables the Crescendo Hardfork

BPS Configuration is what made the 10x throughput increase possible. Without it, the network couldn't safely transition from 1 BPS to 10 BPS while maintaining the same security guarantees. All security parameters scale automatically, ensuring 12-hour finality and 1-hour merge depth remain constant in real time.

### Future-Proofs the Protocol

The BPS Configuration system supports block rates from 1 BPS to 32 BPS with pre-computed K values. This means future hardforks could increase throughput further without requiring fundamental protocol changes. The same formulas and scaling logic would apply at any supported BPS value.

### Maintains Security Guarantees

By scaling all parameters proportionally, BPS Configuration ensures that security properties remain constant regardless of block rate. The probability of anticones exceeding K stays below 1%, finality depth remains 12 hours, and merge depth remains 1 hour. Users experience the same security level at 10 BPS as they did at 1 BPS.

## Key Takeaways

- **Scales all parameters with BPS** - finality depth, merge depth, K, mergeset limit all scale proportionally
- **Time-based security** - converts time durations (hours) into block counts based on BPS
- **Fork-aware** - maintains separate parameter sets for pre/post-Crescendo using DAA score
- **Mathematically derived** - K calculated from GHOSTDAG paper formula, other parameters derived from K
- **Enables 10 BPS** - made the Crescendo hardfork possible while maintaining security guarantees