# Kaspa's Pruning Depth (Anticone Finalization Depth)

## What is Pruning Depth?

Pruning Depth, formally known as Anticone Finalization Depth, is the minimum safe depth at which a block's anticone becomes permanently closed. At 10 BPS, this depth is 591,194 blocks (approximately 16.4 hours). It represents the mathematical guarantee that no future blocks can be added to a block's anticone beyond this point, enabling safe data pruning.

## The Challenge: Determining When Data Can Be Safely Deleted

When you run a Kaspa node, the blockchain continuously grows as new blocks are added. Without pruning, nodes would need to store the entire history forever, making the network unsustainable.

Kaspa's DAG structure allows parallel block production, which creates a complex challenge: determining when historical data is truly final and can be safely deleted. Unlike linear blockchains where finality is straightforward, DAGs must account for blocks that exist in parallel (the anticone) and ensure these parallel structures are permanently closed before pruning.

**Premature Pruning Risk**: If the network prunes data too early, before a block's anticone is fully closed, new blocks could still be added that reference the pruned data. This would break consensus and make it impossible to validate the chain.

**Unbounded Storage Growth**: Without a calculable pruning depth, nodes cannot determine when it's safe to delete old data. This would force all nodes to maintain complete history indefinitely, making full node operation impractical as the network grows.

**Security vs Efficiency Trade-off**: The pruning depth must be deep enough to guarantee safety (no anticone can reopen) while being shallow enough to allow practical data management. Too conservative means excessive storage requirements; too aggressive risks consensus failures.

Pruning Depth solves these problems by providing a mathematically derived formula that accounts for all consensus parameters. This formula guarantees that once a block reaches this depth, its anticone is permanently closed and the data can be safely pruned without any risk to consensus.

## How Pruning Depth Works

### The Anticone Finalization Formula

Pruning Depth is calculated using a formula that combines all the key consensus parameters to ensure complete anticone closure:

anticone_finalization_depth = finality_depth + merge_depth + (4 × mergeset_size_limit × k) + (2 × k) + 2

At 10 BPS, this calculates to: 432,000 + 36,000 + (4 × 248 × 124) + (2 × 124) + 2 = 591,194 blocks.

- **Finality Depth (432,000 blocks)**: The base security parameter ensuring probabilistic finality
- **Merge Depth (36,000 blocks)**: The maximum time window for merging parallel blocks
- **4 × mergeset_size_limit × k (123,008 blocks)**: Accounts for the maximum anticone size based on GHOSTDAG parameters
- **2 × k + 2 (250 blocks)**: Additional safety margin for edge cases in anticone closure

This formula is derived from the prunality analysis which proves that beyond this depth, no new blocks can be added to the anticone.

### Fork-Dependent Calculation

The pruning depth has different values before and after the Crescendo fork activation, because all component parameters scale with BPS.

Pre-Crescendo (1 BPS): anticone_finalization_depth = 86,400 + 3,600 + (4 × 180 × 18) + (2 × 18) + 2 = 103,002 blocks (approximately 28.6 hours).

Post-Crescendo (10 BPS): anticone_finalization_depth = 432,000 + 36,000 + (4 × 248 × 124) + (2 × 124) + 2 = 591,194 blocks (approximately 16.4 hours).

The network uses DAA score to determine which parameter set applies when calculating the pruning depth for any given block.

### Relationship to Actual Pruning Depth

The anticone finalization depth represents the minimum safe depth. The actual pruning depth used by the network is the maximum of two values:

1. The anticone finalization depth (591,194 blocks at 10 BPS)
2. A retention period depth (BPS × 108,000 seconds = 1,080,000 blocks at 10 BPS for 30 hours)

The network uses whichever is larger to ensure both mathematical safety (anticone closure) and practical retention requirements. At 10 BPS, the retention period depth is larger, so the actual pruning depth is 1,080,000 blocks (30 hours).

## How the Network Uses This

### Validating Pruning Point Anticone

When the network calculates the pruning point anticone and trusted data, it verifies that the virtual state is at sufficient depth from the pruning point.

The check is: virtual_blue_score >= pruning_point_blue_score + anticone_finalization_depth

If this condition is not met, the network returns a PruningPointInsufficientDepth error, preventing premature pruning that could compromise consensus. This ensures the pruning point's anticone is truly final before any data is deleted.

### Determining What Data to Keep

Once a valid pruning point is established, the network uses the anticone finalization depth to determine data retention:

- **Full data**: Pruning point and its anticone (all blocks within anticone_finalization_depth)
- **Relations only**: DAA window blocks, GHOSTDAG chain blocks, and pruning proof blocks
- **Headers only**: Past pruning points (blocks older than the current pruning point)

This tiered approach minimizes storage while maintaining all data necessary for consensus validation and IBD (Initial Block Download) with headers proof.

### Enabling Headers-First Sync

The anticone finalization depth enables efficient headers-first synchronization. New nodes can download just the pruning proof (a compact representation of the chain) and the pruning point anticone, then validate and sync from there.

The mathematical guarantee that the anticone is closed means the pruning proof is sufficient to establish consensus without requiring the full historical DAG. This reduces sync time from hours to minutes for new nodes joining the network.

## Why This Matters

### Makes Full Nodes Practical

Without pruning depth, running a full Kaspa node would require storing the entire DAG history indefinitely. At 10 BPS producing ~864,000 blocks per day, this would quickly become impractical. The anticone finalization depth enables nodes to safely prune old data, keeping storage requirements manageable (typically under 100GB) while maintaining full consensus validation capabilities.

### Provides Mathematical Security Guarantees

The pruning depth formula is derived from rigorous mathematical analysis of the GHOSTDAG protocol and DAG structure. It accounts for all possible edge cases in anticone closure, including maximum mergeset sizes, parallel mining scenarios, and fork transitions. This mathematical foundation ensures pruning is provably safe, not just empirically safe.

### Enables Network Scalability

By allowing safe data pruning, the anticone finalization depth makes Kaspa's high-throughput design sustainable long-term. Nodes can participate in consensus without requiring enterprise-grade storage infrastructure. This keeps the network decentralized by ensuring individual users can run full nodes on consumer hardware.

## Key Takeaways

- **591,194 blocks at 10 BPS** - approximately 16.4 hours, the minimum safe depth for anticone closure
- **Mathematically derived formula** - combines finality depth, merge depth, mergeset limit, and GHOSTDAG K
- **Enables safe pruning** - guarantees no future blocks can be added to the anticone beyond this depth
- **Fork-aware calculation** - different values pre/post-Crescendo based on DAA score
- **Actual pruning depth is larger** - network uses max(anticone_finalization_depth, retention_period_depth)
- **Enables headers-first sync** - new nodes can sync efficiently using pruning proof and anticone data