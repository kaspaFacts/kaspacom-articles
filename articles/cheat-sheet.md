# Kaspa Consensus Concepts Cheat Sheet

*A comprehensive guide to Kaspa's blockDAG consensus system*

## Quick Reference

| Parameter | Value (Post-Crescendo) | Purpose |
| --- | --- | --- |
| Blocks Per Second | 10 BPS | Network throughput |
| GHOSTDAG K | 124 | Anticone size limit |
| Block Time | 100ms | Target time between blocks |
| Finality | 12 hours | Time until transactions are irreversible |
| Pruning Depth | 30 hours | How far back nodes store data |
| Merge Depth | 1 hour | Maximum age of mergeable blocks |
| Block Mass Limit | 500,000 | Computational cost limit per block |
| Max TX Inputs/Outputs | 1,000 each | Transaction size limits |

---

## Part I: Core Concepts

*Why Kaspa exists and what problems it solves*

### 1. Consensus Protocols

**PHANTOM**

- **What**: An NP-hard consensus protocol that produces a linear ordering over blocks in a DAG by finding the maximum k-cluster
- **Why**: Need to take a DAG as input and output a linear order, but the optimal solution is computationally infeasible for real-time consensus

**GHOSTDAG**

- **What**: The "greedy" approximation of PHANTOM where the selected parent chain and k-cluster are inherited from the parent instead of recalculated from scratch each round
- **Why**: Achieves linear ordering over a DAG in polynomial time (computationally feasible) while maintaining security properties close to optimal PHANTOM

**DAGKNIGHT**

- **What**: A consensus protocol that outputs linear ordering over a DAG while removing maximum network delay as an input parameter and remaining secure for ANY network latency
- **Why**: GHOSTDAG security breaks down once the assumed max network delay is violated; DAGKNIGHT security does not break down at any latency, making it more robust to network conditions

### 2. GHOSTDAG Implementation Details

**Selected Parent**

- **What**: The parent block with the highest blue work, chosen as the chain anchor for the new block
- **Why**: Ensures the heaviest (most work) chain is always extended, providing Nakamoto-style security guarantees in a DAG structure

**Blue Score**

- **What**: The number of blue (honest) blocks in a block's past, used as a measure of chain depth
- **Why**: Provides a monotonically increasing metric for block ordering that accounts for parallel blocks in the DAG, replacing traditional block height

**Blue Work**

- **What**: The accumulated proof-of-work from all blue blocks in a block's past
- **Why**: Determines the heaviest chain for selected parent selection and provides security against 51% attacks by measuring cumulative honest work

**Mergeset Blues/Reds**

- **What**: Blue blocks are those in the mergeset that don't violate the k-cluster constraint; red blocks are those that do violate it
- **Why**: Blue blocks contribute to consensus ordering and rewards; red blocks are acknowledged but treated as potentially adversarial, preventing attackers from gaining rewards

**K-Cluster Violation Check**

- **What**: Algorithm that checks if adding a candidate block would create an anticone larger than K with any existing blue block
- **Why**: Enforces the security assumption that honest blocks arrive within network delay bounds; blocks violating this are colored red to prevent attacks

### 3. Security Foundations

**2Dλ Parameter**

- **What**: The product of 2 × maximum network delay (D) × block rate (λ), used to calculate GHOSTDAG K
- **Why**: Represents the expected anticone size under honest conditions; K is chosen such that larger anticones occur with probability < δ (1%), ensuring security

**Anticone**

- **What**: The set of blocks that are neither ancestors nor descendants of a given block (parallel blocks in the DAG)
- **Why**: Measures network concurrency; blocks with large anticones may indicate network delays or attacks, hence the K parameter limits acceptable anticone sizes

**GHOSTDAG K**

- **What**: The anticone size parameter that determines how many blocks can be in a block's anticone while still being considered "blue" (honest)
- **Why**: Calculated to ensure anticones larger than K occur with probability less than δ (0.01), maintaining security under network delay assumptions

**Network Delay Bound**

- **What**: Estimated upper bound on network propagation delay, set to 5 seconds
- **Why**: Used in GHOSTDAG K calculations to ensure security under realistic network conditions; blocks arriving within this bound are expected to be properly ordered

**Reachability Service**

- **What**: Efficient data structure for answering ancestor/descendant queries in the DAG
- **Why**: GHOSTDAG requires frequent reachability checks during k-cluster validation; optimized reachability queries make the greedy algorithm computationally feasible

### 4. Economic Model

**Blue vs Red Blocks**

- **What**: Blue blocks are those in the mergeset that don't violate the k-cluster constraint; red blocks are those that do violate it
- **Why**: Blue blocks contribute to consensus ordering and rewards; red blocks are acknowledged but treated as potentially adversarial, preventing attackers from gaining rewards

**Blue Block Rewards**

- **What**: Each blue block in the mergeset (that's also in the DAA window) receives its subsidy + fees as a separate coinbase output
- **Why**: Incentivizes honest mining and proper parent selection; rewards blocks that contribute to consensus ordering

**Red Block Rewards**

- **What**: Post-Crescendo, red blocks in the mergeset contribute their fees (and subsidy if in DAA window) to a single output paid to the merging block's miner
- **Why**: Prevents attackers from profiting from red blocks while still acknowledging their existence; changed in Crescendo to include fees from non-DAA red blocks

**Mass System Overview**

- **What**: Three-component system measuring block computational cost: compute mass (CPU), storage mass (UTXO growth), transient mass (bandwidth/memory)
- **Why**: Prevents blocks from being too expensive to validate while incentivizing UTXO consolidation; ensures decentralization

**Emission Schedule**

- **What**: Block rewards start at 500 KAS per second for ~6 months, then switch to deflationary schedule
- **Why**: Provides initial distribution and mining incentives while transitioning to long-term sustainable emission

---

## Part II: How Kaspa Works

*The concrete rules and mechanisms that make Kaspa function*

### 5. Block Production & DAG Structure

**Blocks Per Second (BPS)**

- **What**: The expected number of blocks produced per second (10 BPS post-Crescendo)
- **Why**: Higher BPS increases throughput; 10 BPS provides ~10x more transactions per second than traditional blockchains

**Target Time Per Block**

- **What**: The target interval between blocks in milliseconds (100ms for 10 BPS)
- **Why**: Determines the block production rate, balancing throughput with network propagation time and security

**Max Block Parents**

- **What**: Maximum number of direct parent blocks a new block can reference (typically K/2, capped at 10-16)
- **Why**: Limits processing complexity while ensuring sufficient DAG connectivity; prevents quadratic growth of parent references with BPS

**Mergeset Size Limit**

- **What**: The maximum number of blocks that can be in a block's mergeset (typically 2×K, bounded 180-512)
- **Why**: Keeps computation manageable for UTXO validation and reachability calculations; exceeding this limit results in block rejection

**Merge Depth**

- **What**: The maximum depth in the past that blocks can be merged without violating bounded merge depth rules (1 hour = 3,600 seconds)
- **Why**: Ensures blocks eventually converge and prevents attacks where old blocks are merged to manipulate the DAG structure

### 6. Difficulty Adjustment System

**DAA Score**

- **What**: Difficulty Adjustment Algorithm score; a monotonically increasing counter representing the cumulative "work" in the selected chain
- **Why**: Provides a canonical ordering for blocks and triggers fork activations; used instead of block height in DAG systems

**Difficulty Window Size**

- **What**: The number of blocks sampled to calculate the next difficulty target (660 blocks sampled at 4-second intervals)
- **Why**: Smooths difficulty adjustments to respond to hashrate changes while avoiding manipulation; maintains ~44 minutes of history

**Difficulty Sample Rate**

- **What**: How frequently blocks are sampled from the DAG for difficulty calculation (every 2×BPS blocks = every 20 blocks at 10 BPS)
- **Why**: Reduces computation at high BPS while maintaining accurate difficulty tracking

**Difficulty Scaling at Activation**

- **What**: At Crescendo activation, difficulty target is scaled by the ratio of old/new target time per block
- **Why**: Maintains consistent mining difficulty across the BPS transition; prevents sudden hashrate-driven difficulty spikes or drops

**Minimum Window Enforcement**

- **What**: DAA requires at least 150 blocks in the difficulty window before activating dynamic adjustment
- **Why**: Stabilizes difficulty during network startup and fork transitions; with 4s sampling equals 10 minutes of fixed difficulty

**Max Difficulty Target**

- **What**: Highest allowed PoW difficulty target, set to 2<sup>255</sup> - 1
- **Why**: Defines the easiest possible difficulty; blocks with higher targets are invalid

### 7. Timestamp Validation System

**Past Median Time (PMT)**

- **What**: The median timestamp of a sampled window of past blocks
- **Why**: Prevents timestamp manipulation attacks; blocks must have timestamps greater than PMT

**PMT Sample Rate**

- **What**: How frequently blocks are sampled for median time calculation (every 10×BPS blocks = every 100 blocks at 10 BPS)
- **Why**: Reduces computation at high BPS while maintaining timestamp security

**PMT Averaging Frame**

- **What**: The 11 values closest to the center of sorted timestamps used to calculate past median time (post-Crescendo)
- **Why**: Provides more stable timestamp validation than a single median value; averages reduce impact of outliers while maintaining security

**Timestamp Deviation Tolerance**

- **What**: The parameter controlling the PMT window size (132 seconds)
- **Why**: Balances protection against timestamp attacks with tolerance for clock skew

### 8. Transaction Rules & Validation

**Max Transaction Inputs**

- **What**: Maximum number of inputs a transaction can have (1,000 post-Crescendo)
- **Why**: Limits mass calculation cost and prevents DoS attacks; soft fork reduction for Crescendo

**Max Transaction Outputs**

- **What**: Maximum number of outputs a transaction can have (1,000 post-Crescendo)
- **Why**: Limits mass calculation cost and UTXO set growth

**Max Signature Script Length**

- **What**: Maximum size of input signature scripts in bytes (10KB post-Crescendo)
- **Why**: Prevents script engine abuse; aligned with script engine's 10KB limit

**Max Script Public Key Length**

- **What**: Maximum size of output script public keys in bytes (10KB post-Crescendo)
- **Why**: Limits compute mass and storage mass calculations; aligned with script engine limits

**Block Mass**

- **What**: A measure of block computational cost, limited to 500,000 per block
- **Why**: Prevents blocks from being too expensive to validate, ensuring decentralization

**Mass Per Transaction Byte**

- **What**: Mass cost per byte of transaction data (1 mass/byte)
- **Why**: Accounts for bandwidth and storage costs

**Mass Per Signature Operation**

- **What**: Mass cost per signature verification (1000 mass/sigop)
- **Why**: Reflects the high computational cost of signature verification

**Storage Mass Parameter**

- **What**: Parameter for KIP-0009 storage mass calculation, penalizing UTXO set growth
- **Why**: Incentivizes UTXO consolidation and prevents state bloat

**Transaction Mass Field**

- **What**: A field in transactions (post-Crescendo) that must be preserved from GetBlockTemplate and passed back in SubmitBlock
- **Why**: Enables proper mass calculation and validation; blocks with transactions having mass=0 post-Crescendo are invalid

**Coinbase Maturity**

- **What**: Number of blocks a coinbase output must wait before being spendable (1,000 blocks post-Crescendo = ~100 seconds at 10 BPS)
- **Why**: Prevents miners from spending rewards from blocks that might be reorganized out of the chain; provides time for consensus to stabilize

### 9. Finality & Pruning System

**Finality Depth**

- **What**: The depth at which blocks are considered finalized and their ordering cannot change (12 hours = 43,200 seconds)
- **Why**: Provides economic finality guarantees; transactions in finalized blocks are irreversible

**Anticone Finalization Depth**

- **What**: The depth at which a block's anticone becomes permanently closed, calculated as finality_depth + merge_depth + 4×mergeset_limit×K + 2×K + 2
- **Why**: Ensures the anticone set stabilizes for pruning safety; derived from the prunality proof

**Pruning Depth**

- **What**: The depth beyond which old block data can be safely deleted (30 hours = 108,000 seconds)
- **Why**: Allows nodes to discard old data while maintaining security, enabling lightweight nodes and reducing storage requirements

**Finality Score**

- **What**: A score calculated from blue score and finality depth used to determine if a block is a pruning sample
- **Why**: Enables efficient identification of pruning samples without examining entire chain history; blocks become samples when their finality score increases

**Pruning Point Candidate**

- **What**: A moving block that maintains pruning depth from sink but has the same finality score as current pruning point
- **Why**: Optimizes pruning point search by providing a better starting point than always searching from current pruning point; reduces chain traversal work

### 10. Virtual State & Mining

**Virtual Block**

- **What**: A conceptual block representing the current DAG tip that merges all valid tips
- **Why**: Provides a single canonical view of the current UTXO state for transaction validation and mining

**Sink Block**

- **What**: The selected parent of the virtual block, chosen via the sink search algorithm
- **Why**: Ensures the virtual block has valid UTXO state and respects finality constraints

**Virtual Parent Selection**

- **What**: Algorithm to choose which tips become virtual parents, respecting mergeset size limits and bounded merge depth
- **Why**: Maximizes DAG connectivity while maintaining validation efficiency and security properties

---

## Part III: Implementation Details

*How Kaspa's systems are built and optimized*

### 11. Post-Crescendo Optimizations

**Sampled Windows**

- **What**: Instead of examining every block, sample blocks at fixed time intervals (4s for difficulty, 10s for median time)
- **Why**: At 10 BPS, examining every block becomes computationally expensive; sampling maintains accuracy while keeping validation costs constant

**Time-Based Depths**

- **What**: Finality (12h), pruning (30h), and merge depth (1h) expressed in time units rather than block counts
- **Why**: Ensures consistent security guarantees across different BPS rates; 12 hours provides same economic security whether at 1 BPS or 10 BPS

### 12. Validation Pipeline

**Header Validation Stages**

- **What**: Multi-stage pipeline including pre-GHOSTDAG, GHOSTDAG calculation, pre-POW, POW, and post-POW validation
- **Why**: Enables early rejection of invalid blocks before expensive operations; supports parallel processing of independent blocks in the DAG

**Body Validation Stages**

- **What**: Two-phase validation with isolation checks (merkle root, mass limits, double spends) followed by context checks (parent bodies, coinbase subsidy, lock time)
- **Why**: Separates context-free validation from UTXO-dependent validation for efficiency; isolation checks can be performed without accessing chain state

### 13. UTXO Processing

**UTXO Processing Context**

- **What**: A structure containing mergeset diff, multiset hash, accepted tx IDs, acceptance data, and mergeset rewards used during virtual state calculation
- **Why**: Encapsulates all state changes from processing a block's mergeset, enabling efficient parallel validation and state accumulation

**Multiset Hash (MuHash)**

- **What**: A cryptographic accumulator tracking the UTXO set state that supports efficient addition/removal operations
- **Why**: Enables compact UTXO set commitments and efficient validation without recomputing entire set; updated incrementally as transactions are processed

**Accepted Transaction Ordering**

- **What**: Post-Crescendo, accepted transaction IDs preserve canonical order (pre-Crescendo they were sorted)
- **Why**: Maintains deterministic merkle root calculation while enabling more efficient processing post-fork

### 14. Window Management

**Window Origin Tracking**

- **What**: Windows are tagged as either <code>Full</code> or <code>Sampled</code> origin to prevent cache mixing between pre and post-Crescendo
- **Why**: Ensures correct window type is used for validation; prevents using pre-Crescendo full windows for post-Crescendo sampled calculations

**Dual Window Manager**

- **What**: System that switches between <code>FullWindowManager</code> (pre-Crescendo) and <code>SampledWindowManager</code> (post-Crescendo) based on selected parent's DAA score
- **Why**: Enables seamless transition between window calculation methods at fork activation; maintains both managers for backward compatibility

**Mergeset Non-DAA Tracking**

- **What**: Set of blocks in the mergeset that are not included in the DAA window (either not sampled or outside window bounds)
- **Why**: Required for correct coinbase reward calculation; red blocks outside DAA window have different reward treatment

### 15. Pruning Point Advancement

**Pruning Point Candidate**

- **What**: A moving block that maintains pruning depth from sink but has the same finality score as current pruning point
- **Why**: Optimizes pruning point search by providing a better starting point than always searching from current pruning point; reduces chain traversal work

**Finality Score Buckets**

- **What**: Blocks are grouped into finality score buckets based on <code>blue_score / finality_depth</code>
- **Why**: Enables efficient pruning point advancement; new pruning point is found when a block's finality score exceeds previous pruning point's score

**V1 vs V2 Algorithms**

- **What**: Pre-Crescendo uses v1 algorithm; post-Crescendo uses v2 with time-based depths; both are validated to be equivalent pre-activation
- **Why**: V2 algorithm handles longer post-Crescendo pruning depths correctly; sanity checks ensure consistency during transition

**Pruning Point Monotonicity**

- **What**: Post-Crescendo pruning point advancement maintains monotonicity property despite finality depth increase
- **Why**: Ensures pruning point never moves backward; property holds because new finality depth is multiple of old finality depth

### 16. Performance Optimizations

**Chain Iterator**

- **What**: Forward chain iterator that traverses from one block to another along the selected parent chain
- **Why**: Enables efficient pruning point searches and finality calculations without examining entire DAG; critical for pruning point advancement algorithm

**Reachability Reindexing**

- **What**: Periodic reindexing of the reachability data structure when slack accumulates beyond threshold
- **Why**: Maintains query performance as DAG grows; reindex depth and slack parameters balance reindexing cost with query efficiency

**Fork-Aware Cache Sizing**

- **What**: Cache sizes use <code>upper_bound()</code> of forked parameters to allocate for worst-case across fork boundaries
- **Why**: Ensures sufficient cache capacity regardless of whether fork has activated; prevents cache thrashing during transition

**Window Cache Byte Budgets**

- **What**: Difficulty and median time window caches use <code>after()</code> values for byte calculations to prefer long-term permanent values
- **Why**: Optimizes memory allocation for post-fork state since that's the permanent configuration

**Natural DAG Parallelism**

- **What**: Blocks in each other's anticone can be processed concurrently since they have no dependency relationship
- **Why**: Higher BPS creates more parallel blocks, enabling better CPU utilization; dependency tracking delays processing until parent blocks complete

### 17. Network & Sync

**Orphan Resolution Range**

- **What**: P2P layer heuristic that checks if orphan blocks are within retrievable distance using block locator protocol
- **Why**: Prevents unbounded orphan pool growth while enabling recovery of missing ancestors

**IBD Relay Coordination**

- **What**: During Initial Block Download, relay flow keeps ~M/2 blocks in orphan pool around IBD trigger point
- **Why**: Ensures smooth transition from IBD to normal sync using DAA score ranges to determine which orphans to keep

**Finality Duration Heuristics**

- **What**: IBD uses finality duration to determine if node's consensus has matured enough to reject headers proofs
- **Why**: Prevents spam attacks while allowing legitimate syncing; adjusts to post-Crescendo shorter finality duration only after sufficient time has passed

**Pruning Depth for IBD Type**

- **What**: IBD determination uses post-Crescendo pruning depth based on headers selected tip's DAA score
- **Why**: Ensures IBD type selection adapts to the longer post-Crescendo pruning depth for proper sync behavior

---

## Appendices

### A. Glossary

**Anticone**: The set of blocks that are neither ancestors nor descendants of a given block (parallel blocks in the DAG)

**Blue/Red Blocks**: Blue blocks don't violate the k-cluster constraint and contribute to consensus ordering; red blocks violate it and are treated as potentially adversarial

**DAA Score**: Difficulty Adjustment Algorithm score; a monotonically increasing counter representing cumulative work in the selected chain

**Mergeset**: The set of blocks merged by a new block, including its parents and anticone blocks (up to limits)

**Virtual State**: The conceptual current tip of the DAG that merges all valid tips, providing canonical UTXO state

**Reachability**: Efficient data structure for answering ancestor/descendant queries in the DAG

**Sink Block**: The selected parent of the virtual block, chosen as the heaviest valid chain tip

### B. Post-Crescendo Changes Summary

- **BPS**: 1 → 10 blocks per second
- **Time-based depths**: Finality (12h), Pruning (30h), Merge depth (1h) now in time units vs block counts
- **Sampled windows**: Difficulty (4s intervals), PMT (10s intervals) instead of examining every block
- **Transaction limits**: Max inputs/outputs reduced from 1 billion to 1,000
- **Script lengths**: Max signature script and script public key reduced to 10KB
- **Mass field**: Required in all transactions, must be preserved from GetBlockTemplate
- **Transaction payloads**: Now supported for non-coinbase transactions
- **Coinbase mass**: Must be set to zero
- **Accepted TX ordering**: Preserves canonical order instead of sorting

### C. Network Types Comparison

| Network | BPS | Crescendo | PoW | Purpose |
| --- | --- | --- | --- | --- |
| Mainnet | 10 (post-Crescendo) | Active at DAA 110,165,000 | Enabled | Production network with real economic value |
| Testnet | 10 (post-Crescendo) | Active at DAA 88,657,000 | Enabled | Public testing without risking real funds |
| Simnet | 10 | Always active | Disabled | Fast testing and benchmarking |
| Devnet | 1 | Disabled by default | Enabled | Stable development environment |

### D. Key Configuration Parameters

- **RAM Scale**: 0.1-10.0 multiplier for cache sizes (default 1.0)
- **Archival Mode**: Keep all historical data (no pruning)
- **UTXO Index**: Enable address-based UTXO queries
- **Unsafe RPC**: Allow state-modifying RPC commands
- **Enable Unsynced Mining**: Allow mining while not synced (for network bootstrap)
- **Retention Period Days**: Custom pruning policy (days of data to retain)
- **Block Template Cache Lifetime**: Duration to cache mining templates (milliseconds)