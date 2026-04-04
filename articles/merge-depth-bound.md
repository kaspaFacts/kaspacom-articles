# Kaspa's Merge Depth Bound

## What is Merge Depth?

The Merge Depth Bound is Kaspa's 1-hour time limit on how far back blocks can be merged into the DAG. It serves as a fundamental security mechanism that prevents difficulty manipulation attacks, enables finality calculations, and ensures network stability.

## The Challenge: Balancing Parallel Mining with Network Security

When you mine blocks in a DAG, the network faces a fundamental challenge: how to allow parallel block production while preventing attacks.

Kaspa's DAG structure enables high throughput by letting miners produce blocks simultaneously. But this creates opportunities that don't exist in single-chain systems. Without time-based limits on block merging, several critical problems emerge:

**Difficulty Manipulation**: Attackers could mine side-chains with manipulated timestamps to create blocks at artificially low difficulty, then merge them later for disproportionate rewards. By using future timestamps, they make the DAA think blocks are arriving slowly, dropping difficulty on their side-chain. This lets them produce many blocks with minimal hashrate.

**Unstable History**: Without merge limits, the network's history would never stabilize. Old blocks could be merged at any time, making deep reorganizations possible and preventing the network from ever reaching finality.

**Impossible Pruning**: The network couldn't safely delete old data because there would be no guarantee that ancient blocks wouldn't suddenly be merged, requiring that historical data.

The Merge Depth Bound solves all these problems with a single mechanism: a 1-hour time window. Blocks can only be merged if they're within this window, which simultaneously forces real-time mining at current difficulty, creates a moving finality boundary, and enables safe data pruning.

## How Merge Depth Works

### The Merge Depth Root

Every block has a merge depth root - an ancestor block marking the boundary of what can be merged.

Post-Crescendo, the merge depth is 1 hour (36,000 blocks at 10 BPS). The network calculates this by going back merge_depth blocks in blue score from the selected parent.

### Kosherizing Blues

Red blocks (merged blocks not in the main chain) outside the merge depth root can still be merged if a blue block (main chain block) within the merge depth root validates them. This allows legitimate parallel mining while preventing attacks.

For example, if a red block was mined 90 minutes ago (outside the 1-hour merge depth), but a blue block from 30 minutes ago (within merge depth) has that red block in its past, the red block can still be merged. This prevents legitimate parallel mining from being rejected while still blocking malicious pre-mining attacks.

### Validation Process

During block validation, the network calculates the merge depth root, checks all red blocks in the mergeset, verifies each red is either within the merge depth root or kosherized by a blue, and rejects the block if it attempts to merge a red that violates the merge bound.

## How the Network Uses This

### Prevents Low-Difficulty Side-Chain Attacks

The primary purpose is preventing DAA manipulation. Without the Merge Depth Bound, an attacker could mine their own side-chain using future timestamps. This makes the DAA think blocks are arriving slowly, dropping the side-chain's difficulty. The attacker mines many blocks quickly with minimal hashrate, then merges them into the honest network for disproportionate rewards. Beyond giving attackers unfair rewards, merging low-difficulty blocks would also disrupt the honest network's difficulty calculation, potentially causing instability in block times for all miners.

The 1-hour bound (matching the ~44-minute DAA Window) prevents this by requiring blocks to be recent enough that they must have been mined at the honest network's current difficulty.

### For Reward Distribution

The Merge Depth Bound works with the DAA Window to determine mining rewards: blocks inside the DAA window (~44 minutes) receive full rewards (coinbase + fees), blocks outside DAA but within Merge Depth (44 min to 1 hour) receive fees only, and blocks outside Merge Depth cannot be merged.

### For Safe Pruning

Merge Depth Bound is a key component of the pruning depth formula:

pruning_depth = finality_depth + merge_depth_bound + (4 × mergeset_size_limit × ghostdag_k) + (2 × ghostdag_k) + 2

This formula ensures data is only pruned after blocks are truly final and their Anticones are permanently closed. The Merge Depth component accounts for the maximum time blocks can be merged, ensuring no future merges can affect pruned data.

### Prevents Deep Reorganizations

The merge depth bound ensures network history becomes increasingly stable over time. Any block older than 1 hour (36,000 blocks at 10 BPS) cannot be merged, creating a permanent barrier against deep reorganizations. The virtual processor actively filters out tips that violate this bound during virtual state resolution.

## Why This Matters

### Protects DAA Integrity

The 1-hour merge depth is explicitly designed to match the ~44-minute DAA window duration to prevent merging low-difficulty side-chains. Attackers cannot pre-mine blocks to manipulate difficulty because they'd need sustained hashrate for the full merge depth period.

### Enables Finality and Pruning

Merge depth is essential for calculating when blocks become final. Without it, the network couldn't safely determine pruning depth, preventing efficient data management.

### Allows Parallel Mining

Kosherizing blues let honest miners merge temporarily delayed blocks while preventing malicious side-chains. This flexibility is crucial for 10 BPS throughput.

## Key Takeaways

- **1 hour merge depth** - blocks must be within 36,000 blocks (at 10 BPS) to merge
- **Protects DAA calculations** - explicitly prevents difficulty manipulation through pre-mined side-chains
- **Enables finality** - key component of Pruning Depth formula with finality_depth, k, and mergeset_size_limit
- **Kosherizing blues** - allow legitimate parallel mining while blocking attacks
- **Time-based security** - requires real-time hashrate, not pre-mined blocks