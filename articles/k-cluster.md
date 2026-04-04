# Kaspa's k-Cluster

## What is a k-Cluster?

A k-cluster is the best connected [DAG](https://kaspa.com/learn-kaspa/post/dag-and-kaspa) within the full DAG.

## The Challenge: Maintain Security

When multiple blocks can exist in parallel, the network needs a way to decide which blocks are best connected to measure and maintain Security.  The k-cluster solves this by identifying blocks that are well-connected.

## How K-Cluster Works

### Finding Well-Connected Blocks

The k-cluster identifies how connected a block is by examining its [Anticone](https://kaspa.com/learn-kaspa/post/dag-terminology).  The Anticone represents blocks that have no clear ordering relationship with the candidate block.

- **Well-connected blocks** have small Anticones (they closely reference other blocks and are closely referenced by other blocks)
- **Poorly-connected blocks** have large Anticones (they are isolated from other blocks)

### Blue vs Red Blocks

[GHOSTDAG](https://kaspa.com/learn-kaspa/post/ghostdag-simplified) uses the k-cluster to classify blocks into two categories:

**Blue Blocks**: Well-connected blocks that contribute to Security.

**Red Blocks**: Valid blocks that are poorly-connected and don't contribute to Security.

Under normal conditions Anticones are smaller than k with > 99% chance (all blocks are Blue under normal conditions).

### Building the K-Cluster

GHOSTDAG creates the k-cluster through a step by step process:

**Step 1: Find the Foundation**

The algorithm starts by finding the Selected Parent.  This Selected Parent is built on the best existing k-cluster choice and this k-cluster becomes the foundation of the new k-cluster.

**Step 2: Identify Candidate Blocks**

Next, it finds all blocks in the [Mergeset](https://kaspa.com/learn-kaspa/post/dag-terminology) and orders them in topological order.

**Step 3: Test Each Candidate**

For each candidate block, GHOSTDAG checks if adding it would violate the k-cluster rules:

- **Rule 1**: The candidate's Blue Anticone must not exceed k
- **Rule 2**: Adding the candidate must not cause any existing Blue block's Anticone to exceed k

**Step 4: Add or Reject**

If both rules pass, the block becomes Blue (part of the k-cluster).  If either rule fails, the block becomes Red (valid but not part of the k-cluster).

This process ensures the k-cluster grows incrementally while maintaining its Security properties, with each new Blue block strengthening the Security of the cluster.

## The K Parameter: Setting Connection Standards

The 'k' in k-cluster is a number that sets the connection standard.  Blocks that violate either of the two k-cluster rules (described above) get marked as Red (poorly-connected).

### Setting k

The k parameter uses the mathematical formula x = 2Dλ where D is Max Block Propagation Time and λ is Block Creation Rate, then finds the minimum k where blocks are well connected with more than 99% chance:

- **Glacial Slow (1 block/10 minutes): **k = 1****
- **Slow (1 block/second)**: k = 18
- **Fast (10 blocks/second)**: k = 124
- **Ultra Fast (32 blocks/second)**: k = 362
- **Faster than Fast (100 blocks/second)**: k = 1013
- **Probability: **k is set so Anticones > k occur with less than 1% chance

## Why This Matters

**Enables High-Speed Consensus**

Identifying the k-cluster allows Kaspa to process blocks much faster than traditional blockchains.  While Bitcoin creates one block every 10 minutes, Kaspa can handle multiple blocks per second by using the k-cluster to identify which parallel blocks can contribute to Security.

**Mathematical Security Guarantees**

The k parameter ensures that blocks with Anticones larger than k occur with less than 1% probability.  This mathematical guarantee allows the k-cluster to reliably identify well-connected blocks even at high Block Creation Rates, since poorly-connected blocks (those violating the k constraint) remain statistically rare.

**Scales with Network Speed**

As Block Creation Rates increase, k is scaled up (from k=18 at 1 BPS to k=1013 at 100 BPS) to maintain the same Security level.  This allows GHOSTDAG to use ANY Block Creation Rate while remining Secure.

## Key Takeaways

<ul>
<li style="font-weight: bold;">**k-cluster identifies the best-connected blocks in a DAG structure**</li>
<li style="font-weight: bold;">**Blue blocks (k-cluster) identify the best connected DAG, red blocks are still valid**</li>
<li style="font-weight: bold;">**The k parameter sets how strict the connection requirements are**</li>
<li style="font-weight: bold;">**Higher block creation rates use higher k values to maintain Security**</li>
<li style="font-weight: bold;">**GHOSTDAG maintains Security while enabling fast, parallel block creation**</li>
</ul>