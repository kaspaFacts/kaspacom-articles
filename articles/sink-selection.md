## What is Sink Selection?

Sink Selection is Kaspa's algorithm for choosing the [Virtual](https://kaspa.com/learn-kaspa/post/virtual-block) [Selected Parent](https://kaspa.com/learn-kaspa/post/parent-chain) (Sink) from multiple competing block tips in the [BlockDAG](https://kaspa.com/learn-kaspa/post/dag-and-kaspa). It identifies the highest-scoring valid block that represents the current network state, serving as the primary reference point for all consensus operations.

## The Challenge: Choosing a Single Tip in a Parallel World

In Kaspa's BlockDAG, multiple blocks can be created simultaneously, creating ambiguity about which block represents the "current" state. Without a clear selection mechanism, nodes would disagree on which transactions are confirmed, what difficulty to use for mining, and how to build the next block.

Kaspa's parallel block structure means the network needs a deterministic way to choose a single reference point from multiple valid tips, otherwise consensus would break down.

**Chain State Ambiguity**: Multiple competing tips make it unclear which UTXO Set represents the current state, preventing wallets from showing accurate balances and miners from building valid blocks.

**Mining Direction Uncertainty**: Without a clear parent selection, miners might build on different tips, fragmenting the network and wasting computational resources.

**Consensus Instability**: Different nodes could choose different tips as their reference point, leading to chain splits and network partitioning.

Sink Selection solves these problems by providing a deterministic algorithm that all nodes can follow to agree on a single Sink Block.

## How Sink Selection Works

### The Sink Search Algorithm

The core algorithm searches for the highest-scoring valid block among current tips using a max-heap data structure.

- **Blue Work Priority**: Uses a max-heap to prioritize blocks with higher [Blue Work](https://kaspa.com/learn-kaspa/post/blue-score-and-blue-work) values
- **UTXO Validation**: Ensures candidates have valid [UTXO](https://kaspa.com/learn-kaspa/post/kaspa-utxo-model) State before selection
- **Finality Compliance**: Rejects blocks that violate [Finality](https://kaspa.com/learn-kaspa/post/finality-depth) constraints to maintain network stability

The algorithm maintains an antichain invariant in its heap and iteratively searches for the highest Blue Work block that meets all validation criteria.

### Virtual Parent Selection

After selecting the Sink, additional Virtual Parents are chosen to incorporate parallel work into the [Virtual Block](https://kaspa.com/learn-kaspa/post/virtual-block).

For example, with a maximum of 10 parents, the system might select the Sink plus 9 additional high-scoring blocks from different parts of the DAG, ensuring diversity while respecting [Mergeset Size Limits](https://kaspa.com/learn-kaspa/post/mergeset-size-limit).

### Virtual State Resolution

The entire process is triggered during `resolve_virtual()` which orchestrates the Sink Selection and updates the Virtual State.

1. Filter tips by Finality Point
2. Execute Sink Search algorithm
3. Select additional Virtual Parents
4. Update Virtual State with new Sink
5. Emit notifications about the Sink change

## How the Network Uses This

### Mining Block Templates

Miners query the current Sink to determine which blocks to use as Parents when building new blocks. The Sink provides the Authoritative State that all miners reference, ensuring coordinated mining efforts and preventing orphan blocks.

This matters because it prevents mining waste and ensures network Security through coordinated mining efforts.

### Transaction Validation

The Sink's UTXO State serves as the reference for validating new transactions. When a transaction is submitted, it's checked against the Virtual UTXO Set to ensure it spends valid outputs.

Validation formula: Check if transaction inputs exist in Virtual UTXO set AND verify signatures

This formula means every transaction must spend existing, unspent outputs and be properly signed to be valid.

### Network Information Services

RPC interfaces expose Sink information to external applications like Block Explorers and Wallets.

For example, a Wallet can query the Sink hash to show a user their current balance with confidence that it reflects the network's agreed-upon State.

## Why This Matters

### Consistent Network State

Sink selection ensures all nodes agree on the current State, even when they have slightly different views of the underlying DAG. This consistency is crucial for maintaining Consensus and preventing chain splits.

### Efficient Mining Operations

By providing clear guidance on which blocks to use as parents, Sink Selection enables efficient mining operations. Miners don't need to analyze the entire DAG - they can simply query the Sink and start building valid blocks.

### Simplified Application Development

External applications can interact with Kaspa as if it were a traditional blockchain, querying the Sink for balances, transaction confirmations, and network metrics. This abstraction dramatically simplifies wallet and exchange development.

## Key Takeaways

- **Sink is the Selected Parent** - The Sink represents the highest-scoring valid block in the current DAG
- **Blue Work Determines Priority** - Blocks with higher Blue Work are preferred during selection
- **UTXO Validation is Required** - Only blocks with valid UTXO State can become the Sink
- **Finality Constraints Must Be Respected** - The Sink must maintain network stability by following Finality rules
- **Enables Consensus in Parallel World** - Sink selection allows Kaspa to benefit from parallel block creation while maintaining agreement