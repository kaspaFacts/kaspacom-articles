# Kaspa's Finality Depth

## What is Finality Depth?

Finality Depth is the point where your transactions become permanent and unchangeable in Kaspa. Once a block containing your transaction reaches this depth, it's locked in forever and cannot be reversed or reorganized.

## The Challenge: Enabling Safe Pruning While Maintaining Security

When you send or receive cryptocurrency, you want to know: "When is this transaction truly final?"

Like Bitcoin, Kaspa uses Proof-of-Work and provides Probabilistic Finality.  The more confirmations, the more computationally expensive it becomes for an attacker to reverse the transaction.  While this probabilistic approach works for Bitcoin's single-chain structure, Kaspa's parallel block DAG generates far more data that must be managed.

This is where the Finality Point becomes critical.  The Finality Point marks a specific depth where the network can guarantee that blocks will never be reorganized.  This guarantee allows Kaspa to safely delete historical block data beyond this point, knowing that no future blocks can ever change what happened before the Finality Point.

Without this Finality Point, Pruning would be unsafe - the network might delete data that could later be needed to validate.  The Finality Point solves this by creating a permanent barrier that enables both Security and Scalability.

## How Finality Works

### Understanding Finality Depth and Finality Point

Finality Depth is the network parameter (432,000 blocks at 10 BPS, or 12 hours). Finality Point is calculated for each block - it's the specific ancestor block that is exactly finality_depth blocks behind in the selected chain. As new blocks arrive, each calculates its own finality point, creating a moving boundary that advances with the chain.

**The Key Rule**: No new blocks can reference blocks that would reorganize past any finality point. The network enforces this by rejecting such blocks during validation.

### Time-Based Security

Finality depth is measured in time, not just block count:

Currently (post-Crescendo activation on 5-5-25), transactions become permanent after 12 hours (432,000 blocks at 10 BPS) .  Before Crescendo, the finality period was 24 hours (86,400 blocks at 1 BPS).

### How the Network Enforces This

The network enforces finality through multiple validation layers that work together:

**Filtering Out Violating Blocks**

When the network processes new blocks, it automatically filters out any block that would violate Finality.  These blocks are rejected before they can affect the DAG.

**Validating Chain Candidates**

During chain selection, the algorithm verifies each candidate block to ensure it respects the Finality Point.  If a block fails this check, it gets rejected with a finality violation warning.

**Preventing Deep Reorganizations**

Additional validation rules ensure that even poorly-connected blocks cannot violate finality.  This prevents any attempt to reorganize the chain beyond Finality Depth.

## Why This Matters

### Your Transactions Become Irreversible

Once your transaction reaches Finality Depth, it cannot be undone.  An attacker would need to control >50% of network hashpower for 12+ hours to reorganize finalized blocks - which is economically infeasible and easily detectable.  This gives you absolute certainty that payments you receive are final.

### The Network Can Safely Prune Old Data

Because nothing beyond Finality Depth can change, the network can safely delete old block data without risk.  This keeps the blockchain manageable in size while maintaining Security.

The Pruning system uses finality depth as the foundation of a larger safety formula (finality_depth + merge_depth + (4 × mergeset_size_limit × k) + (2 × k) + 2) that ensures the anticone of chain blocks is permanently closed before pruning. This ensures that only truly finalized data is removed.

### Protection Against Attacks

The Finality Depth creates a moving barrier that protects the entire history of the blockchain.  The combination of Proof-of-Work and time-based Finality makes deep reorganizations economically impossible.

## Key Takeaways

- **Probabilistic Finality like Bitcoin** - uses Proof-of-Work with mathematical Security guarantees
- **12 hours for guaranteed transaction Finality** - after this period, transactions cannot be changed or reversed no matter what
- **Finality enables safe Pruning** - allows the network to delete old data without compromising Security
- **The network actively prevents Finality violations** - blocks that would violate Finality are rejected during processing
- **Enables Scalability** - old data can be deleted without compromising Security