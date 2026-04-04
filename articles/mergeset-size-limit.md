# Kaspa's Mergeset Size Limit

## What is Mergeset Size Limit?

The Mergeset Size Limit caps the total number of blocks that can be in a single block's mergeset. At 10 BPS, this limit is 248 blocks. It balances DAG parallelism with storage efficiency and computational feasibility.

## The Challenge: Balancing Unlimited Parallelism with Practical Constraints

Kaspa's DAG structure allows miners to produce blocks in parallel, creating a web of blocks rather than a single chain. This parallelism enables high throughput - but without limits, it creates severe practical problems.

When you mine a block, you reference multiple parent blocks. The Mergeset is built from these parents and includes all the blocks they reference. The more parallel mining occurs, the larger these Mergesets can grow. Without a size limit, several critical problems emerge:

**Storage Explosion**: Every block requires GHOSTDAG and Reachability Data Structures that scale with its Mergeset size. Block headers store only direct parents, but the Consensus System must maintain additional data for all blocks in each Mergeset. Without a limit, storage requirements would multiply rapidly as each block's associated data structures grow. At high block rates, this would quickly become unsustainable.

**Computational Overhead**: Processing each block requires validating all blocks in its Mergeset, calculating Reachability Relationships, and updating GHOSTDAG Data. Unbounded Mergesets would make block validation prohibitively expensive, creating a denial-of-service vector where attackers could force nodes to process massive Mergesets.

**Anticone Uncertainty**: The Mergeset Size Limit is a key component in calculating when a block's Anticone permanently closes. The Anticone Finalization Depth formula includes a term ( 4 × mergeset_size_limit × k ) that accounts for the maximum Anticone size. Without this limit, the network couldn't calculate a bounded Anticone Finalization depth, making it impossible to determine when blocks achieve Finality or when data can be Safely Pruned.

The Mergeset Size Limit solves these problems by capping the total Mergeset at 248 blocks, ensuring the network can handle realistic parallel mining scenarios without overwhelming node resources.

## How Mergeset Size Limit Works

### Calculation

The mergeset size limit is calculated as 2 × K, where K is the GHOSTDAG Anticone tolerance parameter. At 10 BPS, K=124, giving us a limit of 248 blocks.

The limit is bounded between 180 and 512 blocks to ensure the network remains practical across different configurations. The minimum ensures sufficient parallelism, while the maximum prevents storage complexity from becoming unmanageable.

### Enforcement

When a node receives a new block, it validates that the block's Mergeset does not exceed the limit. Blocks violating this limit are rejected as invalid, preventing them from being added to the DAG.

## How the Network Uses This

### Enables Anticone Finalization

The Mergeset Size Limit is a critical component of the Anticone Finalization Depth formula. This formula calculates how far ahead virtual must be from a Pruning Point before its Anticone is considered finalized for Pruning:

anticone_finalization_depth = (finality_depth) + (merge_depth_bound) + (4 × mergeset_size_limit × k) + (2 × k) + 2

The (4 × mergeset_size_limit × k) term accounts for the maximum Anticone size. Without bounding the Mergeset size, this term would be unbounded, making it impossible to calculate when sufficient depth has been reached to Safely Prune.

### Enables Safe Pruning

The Mergeset size limit feeds into the Anticone Finalization Depth calculation, which determines how far back the network can Safely delete old data. Without the Mergeset limit, this calculation would be unbounded, making it impossible to determine when data becomes permanently irrelevant and can be Pruned.

### Controls Storage Requirements

The mergeset size limit directly controls storage requirements in the GHOSTDAG and reachability stores. Each header must store reachability data for all blocks in its mergeset. At 10 BPS with a 248-block limit, this remains manageable. Without the limit, storage would grow rapidly with network activity, quickly becoming impractical for full nodes.

## Why This Matters

### Makes High-Throughput DAGs Practical

The Mergeset Size Limit is what makes Kaspa's 10 BPS throughput feasible in practice. By capping the total Mergeset at 248 blocks, nodes can process blocks efficiently while still supporting massive parallelism. Without this limit, the computational and storage costs would make high-BPS networks impractical.

### Guarantees Finality and Pruning

The mergeset limit is essential for calculating Anticone Finalization Depth, which determines the minimum safe depth at which a block's anticone is permanently closed.

### Prevents Resource Exhaustion Attacks

By limiting Mergeset size, the network prevents attackers from creating blocks with massive Mergesets that would overwhelm node resources. Without this protection, an attacker could force nodes to process and store arbitrarily large Mergesets, creating a denial-of-service vector.

## Key Takeaways

- **248 blocks at 10 BPS** - calculated as 2×K where K=124
- **Enables pruning** - key component of Anticone Finalization Depth Formula
- **Controls storage** - bounds storage requirements for GHOSTDAG and Reachability Data
- **Scales with K** - higher parallelism tolerance allows larger Mergesets
- **Security bound** - prevents resource exhaustion attacks through oversized Mergesets