## **Introductory Level - What is First Order Pruning and how does Kaspa use it?**

First Order Pruning is the **first stage** of Kaspa's multi-phase storage optimization.  It removes old block transaction data while maintaining a UTXO Set at the pruning point for state validation - but crucially, it preserves all block headers to maintain blockchain integrity.

**Why "First Order"?** - This terminology emphasizes that removing block bodies is just the beginning.  While First Order Pruning dramatically reduces storage requirements and lowers barriers to running a node (increasing decentralization), it's followed by additional pruning stages that can remove even more data ([Second Order Pruning](https://kaspa.com/learn-kaspa/post/second-order-pruning)).  This article focuses specifically on block body removal - the foundation that makes all subsequent optimizations possible.

**What does that mean?** - This article assumes no prior knowledge, so we start with traditional blockchain storage challenges, then explain how First Order Pruning works by maintaining UTXO Sets, what gets removed versus what gets preserved, how the pruning point UTXO Set enables validation, and how this creates the foundation for Kaspa's scalable storage model that enables broader network participation.

## Traditional Storage vs First Order Pruning

**Traditional Full Storage** - In traditional blockchain implementations, nodes store complete block data including all transaction details from Genesis to the current tip.  This means every transaction input, output, signature, and script is preserved forever, leading to ever-growing storage requirements that can become prohibitive for many users.

**First Order Pruning Challenge** - The challenge is removing old transaction data while still being able to validate new transactions.  New transactions need to reference previous outputs (UTXOs), so the system must maintain enough information to validate these references even after pruning old block bodies.

## UTXO Set as the Foundation

**UTXO Set Definition** - The UTXO Set represents all unspent transaction outputs at a specific point in the blockchain.  A snapshot of all the "coins" that exist and can be spent at that moment, similar to taking inventory of all the money in circulation.

**Pruning Point UTXO Set** - Kaspa maintains a special UTXO Set at the Pruning Point, which serves as the baseline state for validation.  This UTXO Set gets updated as the Pruning Point advances, ensuring it always reflects the correct spendable state at that checkpoint.

**UTXO Set Advancement** - When the Pruning Point moves forward, the system applies <span style="color: #000000;"><span title="A UTXO diff records the changes a block makes to spendable coins: which coins were spent and which new coins were created.">UTXO diffs</span></span> from chain blocks to update the pruning point UTXO Set.  This process ensures the UTXO Set remains accurate as old data is pruned.

## What Gets Pruned in First Order Pruning

**Block Body Data Removal** - First order pruning removes the actual transaction data from old blocks, including transaction inputs, outputs, signatures, and scripts.  This includes UTXO multisets, <span title="A UTXO diff records the changes a block makes to spendable coins: which coins were spent and which new coins were created." style="color: #000000;">UTXO diffs</span>, acceptance data, and the complete block transactions store.

**Header Preservation** - While transaction data is removed, block headers are preserved to maintain the structural integrity of the blockchain.  Blocks transition to "header-only" status, indicating the header exists but the body has been pruned.

**Essential Data Retention** - The system preserves critical data needed for consensus validation, including the pruning point anticone, DAA window blocks, and GHOSTDAG blocks.  This ensures that consensus operations can continue even after pruning.

## How UTXO Set Enables Validation

**Transaction Validation Process** - New transactions can be validated against the pruning point UTXO Set plus any subsequent UTXO changes.  The system validates that referenced UTXOs exist and haven't been double-spent, even without the original transaction data.

**State Reconstruction** - The UTXO Set at the pruning point, combined with <span title="A UTXO diff records the changes a block makes to spendable coins: which coins were spent and which new coins were created." style="color: #000000;">UTXO diffs</span> from subsequent blocks, allows reconstruction of the current spendable state.  This enables full validation capabilities without requiring complete historical transaction data.

**Commitment Verification** - The system can verify UTXO Set integrity using cryptographic commitments in block headers.  This ensures the pruned UTXO Set matches what the blockchain headers claim it should be.

## [Archival vs Pruning Nodes](https://kaspa.com/learn-kaspa/post/archival-nodes-vs-full-nodes)

**Archival Node Behavior** - Nodes configured as archival skip first order pruning entirely, preserving all transaction data.  These nodes serve as the network's complete historical record but require significantly more storage.

**Pruning Node Efficiency** - Regular pruning nodes use first order pruning to maintain manageable storage while still participating fully in consensus validation.  The UTXO Set provides sufficient information for validating new transactions without requiring complete historical data.

## Simplified Definitions

**First Order Pruning** - Removing old block transaction data while maintaining a UTXO Set for validation.

**Pruning Point UTXO Set** - A snapshot of all spendable outputs at the pruning point, used as the baseline for validation.

**Header-Only Status** - Blocks that have had their transaction data pruned but retain their headers.

**UTXO Advancement** - The process of updating the Pruning Point UTXO Set as the Pruning Point moves forward.

First Order Pruning enables storage efficiency while preserving validation capabilities through UTXO Sets.

**Bitcoin vs Kaspa: Full Node Bootstrapping**

**Bitcoin** - Full nodes must download and validate all block data from genesis to bootstrap, requiring complete historical transaction data.  While Bitcoin supports simple pruning after initial sync, new nodes still need the full blockchain history to establish initial state. The linear chain structure makes this process straightforward but storage-intensive.

**Kaspa** - Full nodes can bootstrap using pruning proofs without downloading complete historical data, thanks to First Order Pruning integration with the consensus protocol.  The system validates pruning proofs and applies cryptographically verifiable data (called "trusted data" in the codebase) to establish initial state.  This "trusted data" requires no trust in any party - it's mathematically verified through cryptographic proofs that ensure the data matches consensus rules.  The validation process cryptographically proves that the pruning point proof represents valid consensus state,  while the trusted data undergoes rigorous verification to ensure it matches the expected blockchain state.  This allows new nodes to sync efficiently while maintaining full validation capabilities without trusting any external party.
