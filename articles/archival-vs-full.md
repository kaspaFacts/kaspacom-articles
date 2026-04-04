## What are Kaspa Node Types?

Kaspa supports two distinct node configurations: archival nodes that preserve complete blockchain history, and full nodes (pruning nodes) that selectively remove historical data while maintaining full validation capabilities. Both node types participate equally in consensus and network security, but differ significantly in storage requirements and data retention patterns.

## Quick Overview

**Pruning Nodes Are Full Nodes** - Pruning nodes maintain complete validation capabilities. They can validate all new blocks, participate in consensus, and serve the network without requiring trust in external parties. The pruning point proof system ensures cryptographic verification of the entire blockchain state.

**Archival Nodes Are Optional** - The network can function entirely with pruning nodes because pruning point proofs provide mathematically verifiable guarantees. This contrasts with Bitcoin, which requires archival nodes to bootstrap new participants.

**No Additional Trust Requirements** - The pruning system maintains Bitcoin's trustless model using cryptographic proofs. New nodes bootstrap from pruning proofs and verify the entire state without downloading complete history.

**Network Sustainability** - This design enables sustainable scaling without requiring ever-increasing storage. Pruning nodes provide the same consensus security as archival nodes with reduced hardware requirements.

**Archival Node Behavior** - Archival nodes skip pruning entirely, preserving all consensus data. They serve as the network's complete record but are purely optional.

**Pruning Node Efficiency** - Regular pruning nodes achieve maximum storage efficiency while maintaining full consensus validation through the multi-level proof system.

## The Challenge: Blockchain Storage Sustainability

High-throughput blockchains face a fundamental challenge: continuous data growth threatens network decentralization as storage requirements become prohibitive for most participants. Kaspa's design processes blocks at high rates (currently 10 BPS), making complete historical storage unsustainable for widespread node operation.

In Kaspa's DAG-based architecture, this challenge is amplified because every block adds structural complexity that must be stored and processed. Without an efficient pruning mechanism, the network would gradually centralize around only those operators who could afford ever-increasing storage costs.

**Storage Growth Problem**: Without pruning, node storage requirements grow indefinitely with blockchain size, eventually exceeding practical limits for most participants.

**Network Centralization Risk**: High storage requirements would limit node operation to well-funded entities, compromising Kaspa's decentralized nature.

**Bootstrapping Inefficiency**: New nodes would need to download and process the entire historical chain before becoming functional, creating significant barriers to network participation.

Kaspa solves these challenges through its pruning system that allows nodes to delete historical data while maintaining cryptographic proofs of the deleted state, enabling trustless verification without requiring complete history storage.

## How Node Types Work

### Storage Hierarchy System

Kaspa implements a multi-level storage hierarchy represented by the mathematical relationship B ⊆ R ⊆ C ⊆ H, where each set represents progressively larger data retention scopes:

- **B (Block Bodies)**: The main DAG area actively processed, roughly `past(virtual) \ past(pp)`
- **R (Relations)**: Extends B with consensus-critical blocks including DAA windows and proof-level-0 blocks
- **C (Reachability)**: Extends R with all higher proof levels for verification
- **H (Headers)**: Extends C with headers of past pruning points for historical structure

This hierarchy enables progressive data deletion while maintaining the mathematical invariants necessary for consensus security.

### Archival Node Implementation

Archival nodes preserve all blockchain data indefinitely by bypassing the pruning process entirely. The `is_archival` configuration flag controls this behavior. When enabled, the pruning processor explicitly skips all deletion operations and maintains the retention period root stationary during pruning point advancement, ensuring no historical data is ever marked for deletion.

### Full Node Pruning Process

Full nodes implement automated data deletion through a structured process:

1. Advance the pruning point when sufficient depth is reached (configurable via network parameters)
2. Build multi-level pruning proofs before deletion to maintain verifiability
3. Delete data progressively from storage layers below the retention period root
4. Maintain proof structures and consensus-critical windows for ongoing operation

### Configuration Parameters

Node operators can fine-tune pruning behavior through several key configuration parameters:

- **pruning_depth**: Determines the minimum depth required before a pruning point can advance, ensuring sufficient network consensus before data deletion
- **retention_period_days**: Controls how much historical data to retain beyond the pruning point, allowing operators to balance storage efficiency with data availability needs
- **pruning_proof_m**: Parameter affecting proof size and security. Higher values increase proof robustness but also increase storage requirements. This parameter determines the minimum blue work required for proof validation.

The **pruning_depth** parameter is directly related to network finality - it ensures that blocks are only pruned after they have reached finality status, preventing any possibility of chain reorganizations affecting pruned data. This depth-based approach provides mathematical certainty that pruned data can never be needed for consensus validation.

These parameters work together to provide flexible storage management while maintaining network security guarantees.

## How the Network Uses This

### Consensus Participation

Both archival and full nodes participate equally in Kaspa's consensus process. They can validate all new blocks and serve network requests without requiring trust in external parties. The pruning point proof system ensures that even with pruned data, nodes maintain cryptographic verification of the entire blockchain state.

The security equivalence between node types is enforced through multiple consensus mechanisms. Both node types maintain identical validation rules, participate in the same validation processes, and contribute equally to network security. The pruning system ensures that full nodes can validate all consensus rules without access to historical data, as all necessary validation information is preserved in the proof structures and retained data windows. This design means that a network composed entirely of full nodes maintains the same security guarantees as one with archival nodes.

This design enables the Kaspa network to operate entirely with pruning nodes - archival nodes are purely optional for network operation, unlike Bitcoin where archival nodes are required for bootstrapping new participants.

The impact is that Kaspa achieves sustainable decentralization without requiring participants to store ever-increasing amounts of historical data.

### Network Bootstrapping

New nodes can sync efficiently using pruning proofs rather than downloading complete history. The mathematical relationship B ⊆ R ⊆ C ⊆ H ensures that deleted data can be cryptographically verified through the retained proof structures, maintaining Bitcoin's trustless model while enabling efficient synchronization.

Formula: B ⊆ R ⊆ C ⊆ H represents the nested subset hierarchy where each level contains progressively more blockchain data.

This formula means that every block body (B) has corresponding relations data (R), which has reachability information (C), which finally has header data (H), enabling verifiable pruning at each level.

### Storage Configuration

Node operators choose their configuration based on available resources and use case requirements. Archival nodes are configured using the `--archival` flag, while full nodes use the default configuration. The system includes safeguards that require explicit confirmation when transitioning from archival to pruning mode due to potential data loss.

For archival nodes, additional optimizations are available using the `--rocksdb-preset=hdd` flag for hard disk storage with BlobDB, compression, and rate limiting.

The system includes multiple safeguards to prevent accidental data loss. When transitioning from archival to pruning mode, nodes require explicit user confirmation. During pruning operations, the system acquires exclusive locks to prevent concurrent modifications. Additionally, the pruning processor validates depth requirements before advancing pruning points, ensuring sufficient network consensus before data deletion. These safeguards protect both node operators and network integrity.

### Choosing Your Node Type

**Choose a Full Node if:** You want to participate in network security with minimal storage overhead and don't need historical block data.

**Choose an Archival Node if:** You need complete historical access for explorers, analytics, or auditing services, have abundant storage, and can handle higher I/O requirements. Consider using HDD storage with optimization presets for cost efficiency.

## Why This Matters

### Network Sustainability

The pruning system ensures Kaspa can scale sustainably without requiring ever-increasing storage from participants. Full nodes provide the same consensus security as archival nodes while enabling broader network participation through reduced hardware requirements, maintaining bound storage usage after initial sync.

Full nodes achieve substantial storage reduction compared to archival nodes. This dramatic reduction in storage requirements enables widespread participation while maintaining identical security and validation capabilities. The exact savings vary based on network activity and configuration, but the design ensures that storage requirements remain stable and predictable for full node operators.

### Decentralization Preservation

By allowing nodes to operate with significantly reduced storage requirements, the pruning system preserves Kaspa's decentralized nature. More participants can afford to run full nodes, strengthening network security and resilience against centralization pressures.

### Operational Flexibility

The dual-node approach provides operational flexibility for different use cases. Archival nodes serve specific needs requiring complete historical access (blockchain explorers, analytical services), while full nodes provide the optimal balance of security and efficiency for typical network participation.

While archival nodes are optional for network operation, they serve critical ecosystem functions. They provide complete historical access for blockchain explorers, enable detailed analytics and auditing, and can serve pruning proofs to new nodes more efficiently during network bootstrap. The system includes specific optimizations for archival nodes, such as HDD storage presets with BlobDB and compression, making it practical for operators who choose to maintain full history. However, the network's security model does not depend on their presence, ensuring true decentralization regardless of archival node distribution.

## Network Longevity Through Mathematical Proofs

### Crash Recovery and Data Integrity

The pruning system includes robust recovery mechanisms to handle node crashes during pruning operations. On startup, nodes automatically detect incomplete pruning workflows and recover safely. The system maintains multiple checkpoints (retention checkpoint, UTXO set position) to ensure data consistency even if pruning is interrupted. Additionally, optional sanity checks can verify UTXO commitment integrity after pruning point advances, providing operators with confidence in data integrity.

### Cryptographic Proof System

The pruning proof system is the foundation that enables permanent network operation with pruned nodes. The `PruningProofManager` constructs multi-level proofs that cryptographically verify deleted data. These proofs use a hierarchical sampling approach where higher difficulty blocks achieve higher levels, creating a sparse but mathematically verifiable representation of chain history.

### Multi-Level Proof Construction

Proofs are built level by level, starting from the highest block level down to 0. Each level represents a different sampling density based on block difficulty, ensuring that the proof contains sufficient work to guarantee security while minimizing storage requirements.

### Trustless Bootstrapping

During Initial Block Download (IBD), nodes create a staging consensus environment to safely apply pruning proofs without affecting the main consensus state. The process involves expanding the proof with verified block headers, topologically sorting each proof level by blue work, and populating reachability and header stores. This isolated staging approach ensures that proof application can be validated and rolled back if necessary, maintaining the integrity of the consensus state. Once the proof is successfully applied, the node updates its virtual state based on the new pruning point and resumes normal operation with full validation capabilities.

### Mathematical Guarantees

The system enforces strict mathematical requirements through blue score validation and inter-level consistency checks. Proofs must satisfy minimum blue score requirements (typically 2M where M is the pruning proof parameter), ensuring sufficient cumulative work is represented in each proof level. Additionally, inter-level consistency ensures that blocks at depth M from level L+1 must be present in level L, creating a mathematical chain of verification that spans the entire proof structure. This hierarchical validation guarantees that even with sparse sampling, the proof maintains cryptographic integrity across all levels.

### Proof Validation and Security

When a node receives a pruning proof, it validates it by reconstructing the DAG structure and verifying that the proof satisfies mathematical requirements. This includes proof-of-work verification for each header, inter-level consistency checks, blue score threshold validation, and structural integrity verification.

### Network Resilience

The pruning proof system is designed to be forward-compatible with network upgrades. Proof sizes are small and can vary depending on network activity. Archival nodes can serve pruning proofs to pruning nodes, enabling efficient network bootstrapping. The system maintains security even if all nodes operate in pruning mode, ensuring the network can survive indefinitely without requiring archival nodes for operation.

## Key Takeaways

- **Equal Security** - Both node types maintain identical consensus security and validation capabilities through cryptographic proofs
- **Storage Efficiency** - Full nodes achieve stable storage requirements, while archival nodes grow continuously
- **Network Operation** - Kaspa can function entirely with pruning nodes; archival nodes are optional for network operation
- **Trustless Verification** - The pruning system maintains Bitcoin's trustless model using mathematical proofs rather than trusted parties
- **Configuration Choice** - Node operators choose based on resources and needs, with safeguards preventing accidental data loss