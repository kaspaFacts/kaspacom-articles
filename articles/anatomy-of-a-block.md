## What is a Kaspa Block?

A Kaspa block is a data container consisting of a header and a list of transactions that commits to the [BlockDAG](https://kaspa.com/learn-kaspa/post/dag-and-kaspa) state and enables parallel block creation. It serves as the fundamental unit of consensus, encoding proof-of-work, DAG topology, and transaction commitments in a compact, immutable structure.

## The Challenge: Scaling a Blockchain into a BlockDAG

Kaspa must achieve high throughput and fast [finality](https://kaspa.com/learn-kaspa/post/finality) without sacrificing security or decentralization in a directed acyclic graph (DAG) of blocks. Traditional single-parent chains cannot sustain the required block rates, creating performance bottlenecks and economic pressures that drive mining centralization.

**Single-parent throughput limit**: A linear chain restricts the network to one block per reference interval, creating bottlenecks and encouraging centralization among miners.

**High orphan rates**: Rapid block generation in a chain structure causes many valid blocks to be orphaned, wasting network resources and weakening security.

**Inefficient synchronization**: Nodes replaying entire histories to validate state creates prohibitive bandwidth and storage requirements, limiting participation.

Kaspa solves these problems with multi-parent headers, mass-based resource limits, and cryptographic commitments ([UTXO](https://kaspa.com/learn-kaspa/post/kaspa-utxo-model) and accepted transaction IDs) that enable fast sync and parallel block processing.

## How a Block Works

## ![undefined](https://cdn.buttercms.com/2YOvl8nTQkK12KSqH96r)

## The Block Container

A block consists of two main parts: a header and a transaction list. The header contains metadata and commitments, while transactions encode state transitions. Blocks exist in mutable form during construction (MutableBlock) and immutable form (Block) when shared across threads.

- **MutableBlock**: Used during [mining template](https://kaspa.com/learn-kaspa/post/virtual-block) creation with mutable header and transactions.
- **Block**: Immutable version with Arc-wrapped header and transactions for cheap cloning.
- **BlockTemplate**: Mining-specific wrapper containing the mutable block plus miner data and fee information.

## The Header

**What is the header?** — The header is a 13-field structure that gets hashed for proof-of-work and commits to all other block data. It encodes DAG topology, difficulty, timing, and cryptographic commitments.

**Why is the header needed?** — Without a compact header, miners would need to hash entire blocks, making proof-of-work computationally prohibitive. The header-only design enables efficient PoW by committing to transactions via hash_merkle_root while keeping the hashed data minimal. The header enables efficient validation, DAG traversal, and verification of cryptographic commitments to transactions and UTXO state without requiring full block data or historical replay .

**How is the header built?** — Each field is serialized in a defined order and hashed. The hash is cached for fast access. The header is finalized by recomputing its hash after all fields are set.

**---HASH---**

**What is the hash?** — The block's unique 32-byte identifier computed from all other header fields.

**Why is the hash cached?** — Looking up blocks by hash is the most common operation; caching avoids recomputing the cryptographic hash repeatedly.

**How is the hash computed?** — Serialized fields are hashed in order: version → parents_by_level → hash_merkle_root → accepted_id_merkle_root → utxo_commitment → timestamp → bits → nonce → daa_score → blue_score → blue_work → pruning_point.

**---VERSION---**

**What is version?** — A 16-bit integer indicating the block format version (currently 1).

**Why does version exist?** — It allows future protocol upgrades to change block structure while maintaining backward compatibility.

**How is version used?** — Nodes reject blocks with unknown versions to prevent misinterpretation of new fields.

**---PARENTS_BY_LEVEL---**

**What is parents_by_level?** — A run-length encoded structure encoding [multiple parent blocks](https://kaspa.com/learn-kaspa/post/parents-vs-mergeset) across DAG levels, enabling BlockDAG topology.

**Why is it compressed?** — Many consecutive levels can share the same parent set; Run Length Encoding reduces header size significantly.

**How does it work?** — Each tuple (cumulative_level_count, parent_hashes) encodes a run of levels with identical parents. Level 0 contains direct parents.

**---HASH_MERKLE_ROOT---**

**What is hash_merkle_root?** — A 32-byte [Merkle root](https://kaspa.com/learn-kaspa/post/kaspa-linking-the-body-to-the-header) committing to all transaction hashes in this block.

**Why is it needed?** — It enables compact Merkle proofs for transaction inclusion and prevents transaction tampering after mining.

**How is it computed?** — Transaction hashes (including the [storage mass](https://kaspa.com/learn-kaspa/post/storage-mass-kip-0009) commitment) are hashed into a binary Merkle tree.

**---ACCEPTED_ID_MERKLE_ROOT---**

**What is accepted_id_merkle_root?** — A commitment to transaction IDs accepted by this block's [mergeset](https://kaspa.com/learn-kaspa/post/parents-vs-mergeset), unique to Kaspa's DAG.

**Why does it exist?** — In a DAG, blocks merge transactions from other blocks; this field provides cryptographic proof of acceptance.

**How is it computed?** — It creates a chain of commitments by combining the [selected parent](https://kaspa.com/learn-kaspa/post/parent-chain)'s accepted ID root with the Merkle root of transaction IDs newly accepted in this block's mergeset. This builds a running hash chain across the entire DAG history, allowing anyone to prove which transactions from the wider DAG have been definitively accepted without storing all transaction data.

**---UTXO_COMMITMENT---**

**What is utxo_commitment?** — A cryptographic commitment to the entire UTXO set state after applying this block's transactions.

**Why is it needed?** — It enables fast sync by allowing nodes to verify a received UTXO set without replaying history.

**How is it computed?** — It uses a [MuHash](https://kaspa.com/learn-kaspa/post/kaspa-uses-muhash) accumulator that maintains a running commitment as UTXOs are added and removed during transaction processing. This special type of hash allows elements to be inserted or deleted in any order while producing the same final result. When all UTXO changes for the block are applied, finalize() converts the accumulator into a 32-byte hash stored in the header.

**---TIMESTAMP---**

**What is timestamp?** — Unix timestamp in milliseconds when the block was created.

**Why milliseconds?** — Kaspa's high block rate (1-10 BPS) requires millisecond precision for meaningful timing.

**How is it used?** — This ensures blocks can't travel back in time and helps the network [adjust mining difficulty](https://kaspa.com/learn-kaspa/post/daa-window).

**---BITS---**

**What is bits?** — A 32-bit compact encoding of the proof-of-work difficulty target.

**Why is it in the header?** — Being part of the hash pre-image prevents retroactive difficulty reduction.

**How is it calculated?** — The [difficulty adjustment algorithm](https://kaspa.com/learn-kaspa/post/daa-window) measures how quickly blocks were actually created compared to the expected rate, then adjusts the target proportionally. This encoded target determines how difficult it is to find a valid proof-of-work solution.

**---NONCE---**

**What is nonce?** — A 64-bit value miners iterate to find a valid proof-of-work solution.

**Why 64-bit?** — High block rates require a larger search space than Bitcoin's 32-bit nonce.

**How is it used?** — Miners adjust this number until they find a value that makes the block hash small enough to satisfy the difficulty target.

**---DAA_SCORE---**

**What is daa_score?** — A monotonically increasing counter of DAA-included blocks in the block's past.

**Why does it exist?** — In a DAG, it provides an unambiguous measure of progress for subsidy and lock-time calculations.

**How is it computed?** — It starts from the selected parent's [DAA score](https://kaspa.com/learn-kaspa/post/daa-score) and adds the number of blocks in this block's mergeset that are recent enough to be included in difficulty calculations.

**---BLUE_WORK---**

**What is blue_work?** — Cumulative proof-of-work contributed by all blue blocks in the block's past.

**Why is it needed?** — It's the primary criterion for selecting the selected parent, ensuring canonical ordering.

**How is it computed?** — It starts from the selected parent's [blue work](https://kaspa.com/learn-kaspa/post/blue-work-ordering-pt4) and adds the proof-of-work value (derived from difficulty) of each blue block in this block's mergeset.

**---BLUE_SCORE---**

**What is blue_score?** — A counter of blue blocks in the block's entire past.

**Why differ from daa_score?** — Blue score counts all blue blocks; daa_score excludes very old blocks in the mergeset.

**How is it computed?** —  It starts from the selected parent's blue score and adds the count of blue blocks in this block's mergeset.

**---PRUNING_POINT---**

**What is pruning_point?** — The hash of the global pruning boundary as seen by the block's miner.

**Why is it in the header?** — It provides a consensus anchor during IBD, preventing eclipse attacks with fraudulent pruning points.

**How is it determined?** — Each block declares the pruning point from its perspective; nodes validate consistency.

## The Transaction List

**What are transactions?** — Transactions are state transitions that spend UTXOs (inputs) and create new UTXOs (outputs). The first transaction (index 0) is always the [coinbase](https://kaspa.com/learn-kaspa/post/coinbase-transactions).

**Why are transactions structured this way?** — This structure enables efficient validation, mass calculation, and support for sub-networks while maintaining compatibility with existing tooling.

**How are transactions ordered?** — Coinbase at index 0, followed by regular transactions. Their hashes are committed in hash_merkle_root.

**---VERSION---**

**What is version?** — A 16-bit integer indicating transaction format version (currently 0).

**Why does it exist?** — Allows future transaction upgrades while maintaining backward compatibility.

**How is it used?** — Nodes reject transactions with unknown versions to prevent misinterpretation.

**---INPUTS---**

**What are inputs?** — A list of TransactionInput objects referencing UTXOs to be spent.

**Why are inputs needed?** — They provide the cryptographic proof of ownership and authorization to spend referenced UTXOs.

**How do inputs work?** — Each input contains a previous outpoint (txid + index), signature script, sequence, and sig-op count.

**---OUTPUTS---**

**What are outputs?** — A list of TransactionOutput objects creating new UTXOs.

**Why are outputs needed?** — They define the amount and conditions (script) under which future transactions can spend these UTXOs.

**How are outputs structured?** — Each output contains a value (in sompi) and a script public key (locking script).

**---LOCK_TIME---**

**What is lock_time?** — A field specifying the earliest time a transaction can be included in a block.

**Why does it exist?** — It enables time-locked transactions for payment channels and vesting schedules.

**How is it interpreted?** — If the value is small, it refers to a block count; if large, it refers to a specific calendar time.

**---SUBNETWORK_ID---**

**What is subnetwork_id?** — A 20-byte identifier categorizing the transaction (native, coinbase, registry).

**Why does it exist?** — It enables Layer-2 sub-networks and partial node validation by routing transactions.

**How is it used?** — Built-in IDs: native (0x00), coinbase (0x01), registry (0x02). Others can be registered.

**---GAS---**

**What is gas?** — A field reserved for sub-network smart contract execution metering.

**Why is it reserved?** — Future sub-networks may require gas for computation; native/coinbase transactions use 0.

**How is it calculated?** — Currently unused; set to 0 for native and coinbase transactions.

**---PAYLOAD---**

**What is payload?** — An arbitrary byte array carrying additional data.

**Why does it exist?** — Coinbase uses it for metadata; sub-networks use it for smart contract data.

**How is it structured?** — Empty for native transactions; coinbase has a specific binary layout.

**---MASS---**

**What is mass?** — A commitment to storage mass consumed by the transaction ([KIP-0009](https://kaspa.com/learn-kaspa/post/storage-mass-kip-0009)).

**Why is it needed?** — Storage mass accounts for long-term UTXO set bloat; transactions creating many small UTXOs pay more.

**How is it computed?** — This commitment is included when calculating the transaction's hash but not its ID, keeping wallet software compatible.

**---ID---**

**What is id?** — A cached 32-byte transaction identifier.

**Why is it cached?** — Transaction IDs are frequently referenced; caching avoids recomputation.

**How is it computed?** — For non-coinbase: hash all fields except signature scripts and mass; for coinbase: include full payload.

### The Coinbase Transaction

**What is the coinbase?** — A special transaction at index 0 with no inputs that creates new KAS (subsidy) and distributes fees.

**Why is it mandatory?** — It provides the block reward mechanism and fee distribution while carrying essential metadata.

**How is it structured?** — Its payload follows a precise binary layout: blue_score (8 bytes) + subsidy (8 bytes) + SPK version (2 bytes) + SPK length (1 byte) + script + extra data.

## How the Network Uses Block Structure

### Mining

Miners request BlockTemplate objects containing a mutable block, miner data, and fee information. They construct the coinbase payload, select transactions respecting mass limits, and search for a valid nonce/timestamp combination satisfying the difficulty target.

The template includes selected parent metadata (timestamp, DAA score, hash) to validate the miner's chosen timestamp. Calculated fees ensure proper reward distribution.

This enables decentralized mining while enforcing consensus rules and preventing reward manipulation.

### Validation

Nodes validate blocks by checking header hash against difficulty, verifying Merkle roots, ensuring mass limits, and validating transactions against the UTXO set. The BlockStatus enum tracks validation stages.

Header validation includes checking parents_by_level, timestamps, and difficulty. Transaction validation checks signatures, scripts, and mass commitments.

This layered validation ensures security and consistency across the DAG while enabling efficient processing.

### Synchronization

During Initial Block Download (IBD), nodes use the pruning_point and utxo_commitment to verify received UTXO sets without replaying history. The accepted_id_merkle_root provides efficient inclusion proofs for transaction acceptance.

Nodes download headers first, then block bodies as needed. The pruning_point defines the finality boundary for safe pruning.

This enables fast sync and reduced storage requirements while maintaining cryptographic guarantees.

## Why This Matters

### High Throughput

Kaspa's block structure supports 10 blocks per second with security via blue_work selection and multi-parent headers. The mass model (500,000 limit) bounds computational and storage requirements for individual blocks while enabling rapid block propagation.

### Fast Sync

UTXO commitments and pruning points enable nodes to sync in minutes by verifying a snapshot rather than replaying years of history. This reduces barriers to running a node.

### Future-Proofing

Version fields and subnetwork routing allow protocol upgrades and Layer-2(or Layer-1.5) development without breaking existing infrastructure. The separation of transaction ID and hash maintains ecosystem compatibility.

## Key Takeaways

- **Multi-parent headers** - Enable BlockDAG topology and parallel block creation, fundamentally different from single-parent chains.
- **Cryptographic commitments** - Merkle roots, UTXO commitments, and accepted ID roots provide compact proofs and enable fast sync.
- **Mass-based limits** - Replace byte-size limits with a nuanced model accounting for computation, bandwidth, and storage costs.
- **Coinbase structure** - Encodes essential metadata (blue score, subsidy, miner payout) in a standardized binary format.
- **DAG-specific fields** - Blue work, blue score, and DAA score provide consensus metrics unique to Kaspa's parallel block processing.
