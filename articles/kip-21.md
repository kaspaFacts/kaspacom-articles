## What is Partitioned Sequencing Commitment?

Partitioned Sequencing Commitment is a consensus rule change that replaces Kaspa's linear transaction ordering with a lane-based approach, enabling more efficient zero-knowledge proof generation. Instead of committing to all transactions in one global stream, it maintains separate commitments for active application lanes while preserving global order reconstructibility. This allows ZK provers to generate proofs proportional to lane activity rather than global throughput.

## Core Design Principles

KIP-21 is built around three fundamental design principles that guide its architecture:

1. **Single Header Commitment**: Maintains compatibility with existing consensus by keeping a single 32-byte commitment in the block header that can be consumed by existing script opcodes. This ensures backward compatibility while enabling the new lane-based functionality.
2. **O(Activity) Proving**: Makes per-lane proving scale with that lane's own activity rather than global network throughput. This is the primary efficiency gain - applications only need to process their own lane activity, not the entire network's transaction volume.
3. **Bounded Memory**: Enables automatic cleanup of inactive lanes through a purge mechanism keyed by blue score. This ensures node memory scales with recent activity rather than historical lane diversity, preventing unbounded memory growth.

These principles work together to enable efficient zero-knowledge proof generation while maintaining Kaspa's consensus security and operational practicality.

## The Challenge: Scaling ZK Proofs in High-Throughput Systems

Kaspa's current linear sequencing commitment creates computational efficiency challenges for zero-knowledge proving systems that need to verify application-specific state transitions. As network throughput increases, these systems face proof costs that scale with global throughput making proving computationally expensive at scale.

In Kaspa's current implementation, the <code>accepted_id_merkle_root</code> field commits to a linear list of all accepted transactions per block. This design forces ZK provers to process global activity even when they only care about specific application lanes.

**Global Throughput Dependency**: ZK provers must process or commit to all network activity in every block, creating proof costs proportional to global throughput regardless of application scope.

**Memory Scaling Issues**: Maintaining complete transaction history for proof generation requires unbounded memory growth as the network scales.

**Limited Application Isolation**: Applications cannot efficiently prove statements about their own state without processing unrelated network activity, reducing practical utility for L2 systems.

Partitioned Sequencing Commitment solves these problems by organizing transactions into lanes based on their subnetwork ID, allowing applications to prove statements using only their lane's activity while maintaining global consensus security.

## How Partitioned Sequencing Commitment Works

### Lane-Based Transaction Organization

The mechanism groups transactions by their subnetwork ID into separate lanes, each maintaining its own recursive tip hash and activity tracking. This replaces Kaspa's current linear commitment approach, where all transactions in a block are committed to as one global append-only stream.

- **Lane ID Extraction**: Uses <code>tx.subnetwork_id</code> as the lane identifier, treating native, coinbase, and registry subnets as distinct lanes
- **Activity Digests**: Computes per-lane Merkle roots of transaction activity with merge indices for order reconstruction
- **Tip Updates**: Maintains recursive hash chains for each lane's activity history

Each lane operates independently while contributing to the global commitment structure, enabling efficient lane-local proofs without sacrificing consensus integrity.

### Sparse Merkle Tree Commitment

Active lane tips are stored in a sparse Merkle tree (SMT) that provides efficient proofs of inclusion and non-inclusion. The SMT root commits to all currently active lanes while allowing inactive lanes to be purged after a timeout period.

For example, a DeFi application operating on subnet ID 0x1234... would have its transactions grouped into lane 0x1234..., with its activity committed separately from other applications like payment channels or NFT marketplaces using different subnet IDs.

### Commitment Structure Construction

The system builds the final commitment through a layered approach that maintains backward compatibility:

1. Compute per-lane activity digests from grouped transactions
2. Update lane tips recursively using previous tips or global anchors
3. Store updated lane records in the active-lanes SMT to get ActiveLanesRoot
4. Compute block context hash (timestamp, DAA score, blue score) and miner payload root
5. Combine these into SeqStateRoot through selected-parent chaining
6. Publish the final SeqCommit in the existing accepted_id_merkle_root field

## How the Network Uses This

### Zero-Knowledge Rollup Proving

ZK rollups can now generate proofs proportional to their application activity rather than global network throughput. A rollup processing 100 transactions per second can create compact proofs even when the network processes thousands of transactions total.

The proving model uses two global anchors plus a compressed lane-local transition: SeqCommit(B_start) and SeqCommit(B_end) provide global security, while the lane tip transition proves application-specific state changes between these anchors.

This reduces proof generation costs from O(global_throughput × time_window) to O(application_activity × time_window), making ZK rollups practical at scale.

### Application Lane Isolation

Applications can prove statements about their state without processing unrelated network activity. The lane-based commitment provides natural isolation while maintaining the security guarantees of Kaspa's consensus.

Proof complexity formula: O(lane_activity) instead of O(global_activity)

This means a payment channel processing 10 transactions can generate efficient proofs even when the network processes 10,000 transactions, as the proof only needs to consider the payment channel's lane activity (only 10 txs in this example).

### Bounded Memory Management

Inactive lanes are automatically purged after a fixed blue score threshold F, ensuring node memory scales with recent activity rather than historical lane diversity. The purge mechanism tracks last-touch blue scores and removes lanes that haven't been updated within the activity window.

For example, with F = 1000 blue score units, lanes inactive for more than 1000 units are removed from the active set, though they can be re-anchored later if activity resumes.

## Why This Matters

### ZK Proof Efficiency

Reduces ZK proof generation costs proportional to application activity for application-specific proving, achieving efficiency gains proportional to the ratio of global to application activity. For example, if global throughput is 10,000 transactions and an application processes 100 transactions, the proof scope is reduced by a factor of 100.

### Scalable L2 Ecosystem

Enables practical deployment of diverse L2 applications on Kaspa without forcing them to compete for proving resources. Each application scales with its own usage rather than network-wide activity, supporting many independent specialized rollups.

### Future Compatibility

Designed to work seamlessly with future vProg-based lanes and richer lane-local update rules without requiring consensus changes. The outer commitment machinery remains the same while lane-specific logic can evolve independently.

## Key Takeaways

- **Lane-Based Organization** - Transactions are grouped by subnetwork ID into independent lanes with separate activity tracking
- **O(Activity) Proving** - ZK proof costs scale with application activity rather than global network throughput
- **Backward Compatibility** - Uses existing <code>accepted_id_merkle_root</code> field with activation-based logic switching
- **Bounded Memory** - Inactive lanes are purged after timeout F, ensuring memory scales with recent activity
- **Global Security** - Maintains Kaspa's consensus security while enabling efficient application-specific proofs

## Implementation Status

KIP-21 is currently a proposal and has not yet been implemented in the Kaspa codebase. The specification represents a future upgrade path that will require a hard fork activation when implemented. The current Kaspa network continues to operate with the linear sequencing commitment mechanism described in the challenges section.