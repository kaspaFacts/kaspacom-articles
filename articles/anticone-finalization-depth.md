# Anticone Finalization Depth

## What is Anticone Finalization Depth?

Anticone Finalization Depth is the security parameter that determines when a block's anticone—the set of blocks that are neither ancestors nor descendants of it—becomes effectively permanently closed with exponentially strong probabilistic guarantees. This concept is unique to DAG-based blockchains; linear chains don't have anticones because every block is strictly before or after every other block. Anticone finalization extends the same exponential proof-of-work security that protects against chain reorganizations to also cover the DAG-specific requirement of proving that parallel block sets (anticones) are permanently closed. The probability of the anticone growing beyond this depth decays exponentially, making it negligibly small in practice under honest majority assumptions—similar to how we treat cryptographic hash collisions as impossible despite being theoretically possible.

## The Challenge: Proving Parallel Paths Are Closed

Both traditional linear blockchains and Kaspa's DAG face pruning attacks where an attacker could create a secret fork, wait for the fork point to be pruned, then release their chain—forcing nodes to require previously deleted data. In linear chains, this attack requires making a pruned block from the pruning point's past reappear. In DAGs, the attack can additionally target blocks in the pruning point's anticone, potentially causing the pruning point's past to change. Both systems rely on the same exponential probabilistic security: the attacker must win a block race from the pruned depth to current tips, which becomes exponentially unlikely with depth under honest majority assumptions.

However, Kaspa's DAG structure introduces an additional challenge that doesn't exist in linear chains. When blocks exist in parallel, they form an "anticone" relationship—neither block is in the other's past or future. The network must prove not only that deep reorganizations are exponentially unlikely (like linear chains), but also that the entire set of parallel blocks (the anticone) is permanently closed and will never grow. This requires additional security mechanisms beyond simple depth-based finality.

**Parallel Uncertainty**: In a DAG, how do you know when all parallel branches have converged? A block might have been mined in parallel but delayed in transmission. While no approach can deterministically prevent late arrivals, the network needs a mechanism where the probability of problematic late blocks decays exponentially with depth—and must prove the parallel set itself is closed.

**Temporal Ambiguity**: Blocks can arrive late due to network delays or adversarial withholding. When can you be confident that no more "late" blocks will arrive that belong to a block's anticone? You need exponentially strong probabilistic guarantees that the anticone set is permanently closed, not just that deep reorgs are unlikely.

**Trustless Verification**: New nodes joining the network need to verify the blockchain state without trusting other nodes. They must independently confirm that a pruning point's anticone is closed with sufficient security. Without provable finalization rules for anticone closure, nodes would have to trust others' claims about which data is safe to skip.

Anticone Finalization Depth solves these problems through a sophisticated invalidation rule system that extends proof-of-work security to cover the DAG-specific anticone closure requirement. This forces any attacker attempting a pruning attack into a block race against the honest network, where the probability of success decays exponentially with the pruning depth parameter—providing equivalent probabilistic security to linear chains at comparable depths, but for a broader attack surface.

## How Anticone Finalization Depth Works

### The Conceptual Building Blocks

Anticone finalization isn't a single parameter—it's a composite guarantee built from multiple security concepts. Each component addresses a different way the anticone could potentially grow:

- **Chain Stability Depth (Finality Depth)**: Ensures the main selected chain is stable enough that deep reorganizations are exponentially unlikely. This prevents the anticone from reopening due to chain reorgs under honest majority assumptions.
- **Merge Window (Merge Depth)**: Accounts for the maximum "lateness" of blocks—how far back in time a parallel block can still merge into the DAG. This bounds the temporal uncertainty of block arrivals.
- **Anticone Size Bounds (K Parameter and Mergeset Limit)**: Uses the K security parameter and mergeset size limit to constrain how large an anticone can grow. This prevents adversarial miners from creating unbounded parallel structures.
- **Mathematical Safety Margins**: Additional buffer terms derived from formal security proofs that account for worst-case scenarios and ensure the exponential decay property holds.

These components combine into a single formula (detailed in the Pruning Depth article): finality depth + merge depth + anticone size bounds + safety margins. Together, they ensure that chain stability prevents the past from changing, the merge window prevents late arrivals, anticone size bounds prevent parallel expansion, and safety margins guarantee the proof holds even under adversarial conditions with honest majority.

### The Invalidation Rule System

The security of anticone finalization relies on three consensus invalidation rules that force attackers into losing block races. A block is considered invalid if:

1. **Invalid Parent Propagation**: The block has an invalid parent. Since invalid blocks are discarded, any block pointing to them cannot be trusted as the data is unavailable.
2. **Bounded Mergeset**: The block's mergeset (blocks in its past that are neither its selected parent nor in its selected parent's past) exceeds the size limit. This prevents attackers from performing a "climbing attack"—creating a kosherizing block pointing to both a problematic block and the finality point, then creating a succession of blocks that gradually climb up the selected chain while keeping the kosherizing block in their blueset. The mergeset limit bounds this attack to a fixed number of blocks, forcing the attacker into a block race they will exponentially likely lose.
3. **Kosherizing Block Requirement**: There exists a block in both the block's past and the pruning point's anticone that has no "kosherizing block." A kosherizing block must be in the block's blueset, have the finality point in its selected chain, and contain the problematic block in its past. This ensures blocks in the pruning point's anticone were known to the network before the finality point was established.

Under these rules and the assumption that the virtual block must always be valid, any attacker trying to make the network require data from a pruned block's anticone must win a block race against the honest network. The formal security proof shows that under honest majority, the probability of such an attack succeeding is exponentially small in the pruning depth parameter.

### Probabilistic vs. Effectively Deterministic

It's important to understand the security model:

1. **Theoretical Possibility**: A 51% attack could theoretically force the network to require data from a pruned block's anticone. No proof-of-work system can deterministically prevent this.
2. **Exponential Decay**: Under honest majority (less than 50% adversarial hash power), the probability of a successful pruning attack decays exponentially with the pruning depth parameter. This makes it negligibly small in practice.
3. **Implementation Treatment**: Because the probability is so small (similar to cryptographic hash collisions), the implementation treats anticone finalization as effectively deterministic once the depth threshold is reached. The network operates as if the anticone is permanently closed.
4. **Security Assumptions**: This guarantee holds under two key assumptions: (1) there is never a reorg deeper than the finality depth (itself probabilistic under PoW), and (2) there is an honest majority of hash power.

## How the Network Uses This

### Trustless Pruning Point Verification

When nodes validate a pruning point, they must verify that its anticone is closed with sufficient security before accepting it. The anticone finalization depth provides the mathematical criterion: if the current chain tip is at least anticone finalization depth blocks ahead of the pruning point, the anticone is permanently closed (with exponentially small probability of reopening) under honest majority assumptions—similar to how we treat cryptographic hash collisions as impossible despite being theoretically possible.

This verification is trustless—nodes don't trust other nodes' claims about the pruning point. They independently verify the depth requirement using only block headers and the finalization formula. The term "trusted data" in the protocol refers to data that requires no external trust because it can be independently verified against the finalization proof and invalidation rules.

This is fundamentally different from systems that rely on checkpoints or trusted authorities. Every node can independently verify the anticone is closed using only the mathematical properties of the DAG structure and the consensus invalidation rules.

### Enabling Fast Synchronization

New nodes joining the network use anticone finalization to sync quickly without downloading all historical data. Instead of downloading every block, they download a compact "pruning proof" that demonstrates the anticone is closed with sufficient security.

The proof works because anticone finalization provides a hard boundary: everything beyond the anticone finalization depth is permanently closed with exponentially strong guarantees and can be summarized compactly. Everything within the anticone finalization depth must be downloaded in full. This creates a natural division between "historical data that can be compressed" and "recent data that must be verified in detail."

The synchronization is trustless because the finalization depth formula and invalidation rules are public and deterministic. New nodes can verify that the proof respects the finalization boundary and that blocks satisfy the invalidation rules without trusting the node providing the proof.

### Security Boundary for Data Deletion

Anticone finalization creates a clear security boundary: data beyond the anticone finalization depth can be safely deleted because the anticone is permanently closed with exponentially strong probabilistic guarantees. Data within the anticone finalization depth must be retained because the anticone might still grow.

This boundary is effectively absolute under honest majority assumptions. Unlike systems where pruning is a "best guess" based on empirical observation, Kaspa's pruning is backed by formal security proofs showing exponential decay of attack probability. If a node prunes data beyond the anticone finalization depth, it's exponentially unlikely that future blocks will reference that data, because the anticone closure proof and invalidation rules make such scenarios require winning block races against the honest network.

## Why This Matters

### DAG-Specific Innovation

Anticone finalization is a concept unique to DAG-based blockchains. Linear chains don't need it because they don't have anticones—every block is either before or after every other block. Both linear and DAG blockchains face pruning attacks requiring exponential security, but DAGs have the additional complexity of proving that parallel structures (anticones) have closed. The invalidation rules extend the same exponential proof-of-work security model to cover this additional attack vector, providing equivalent probabilistic security to traditional blockchains at comparable depths, but for a broader set of attack scenarios.

### Exponentially Strong Probabilistic Security

Both traditional blockchains and Kaspa rely on probabilistic finality based on proof-of-work security, with attack probability decaying exponentially as depth increases. Kaspa's DAG structure introduces a unique attack vector—pruning attacks targeting the anticone—that doesn't exist in linear chains. Anticone finalization extends the same exponential security model to cover this attack vector through sophisticated invalidation rules. The protocol forces attackers attempting pruning attacks into block races they will exponentially likely lose, with the probability of success decaying exponentially with the pruning depth parameter—providing equivalent probabilistic security to traditional blockchains at comparable depths. This makes the security guarantee so strong that the implementation treats it as effectively deterministic—but it's important to understand it remains probabilistic under the hood, relying on honest majority assumptions. This is not a weakness but rather the fundamental nature of proof-of-work security, similar to how we treat cryptographic hash collisions as "impossible" despite being theoretically possible.

### Foundation for Scalability

Without anticone finalization, DAG-based blockchains would face an impossible choice: either store all history forever (unsustainable), or prune data without strong security guarantees (insecure). Anticone finalization resolves this dilemma by providing a provable boundary for safe data deletion backed by formal security proofs. This makes Kaspa's high-throughput design sustainable long-term—nodes can prune aggressively while maintaining exponentially strong security guarantees under honest majority, enabling the network to scale indefinitely without requiring ever-increasing storage.

## Key Takeaways

- **Exponentially Strong Probabilistic Guarantee** - Anticone finalization provides formal proof that attack probability decays exponentially with depth, making the anticone permanently closed in practice under honest majority assumptions (similar to treating hash collisions as impossible)
- **DAG-Specific Concept** - Linear blockchains don't need anticone finalization; it's unique to systems with parallel block production and the associated anticone relationships
- **Trustless Verification** - Nodes independently verify anticone closure using the finalization formula and invalidation rules; no external trust required (despite the term "trusted data")
- **Invalidation Rule System** - Three consensus rules (invalid parent propagation, bounded mergeset, kosherizing block requirement) force attackers into block races they exponentially likely lose
- **Scalability Foundation** - Enables unbounded network growth with bounded storage requirements, making high-throughput DAGs sustainable long-term under honest majority assumptions