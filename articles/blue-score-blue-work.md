# Introductory Level - Kaspa Uses Blue Score and Blue Work - What are they?

What is Blue Score and Blue Work and how does Kaspa use them?

A system for measuring the cumulative proof-of-work in Kaspa's [BlockDAG](https://kaspa.com/learn-kaspa/post/dag-and-kaspa) structure.

What does that mean? This article assumes no prior knowledge, so we start with traditional blockchain work measurement, then explain how Kaspa's DAG structure requires new metrics, what Blue Score and Blue Work represent, how they're calculated, and how they enable consensus in a parallel block environment.

## Traditional Blockchain Work vs DAG Work

**Traditional Blockchain Work** - In a linear blockchain like Bitcoin, measuring cumulative work is straightforward because there's only one chain.  Each block builds on the previous one, so you can simply add up all the proof-of-work from Genesis to any given block to determine which chain has the most work.  This works because blocks form a single sequence where each block has exactly one parent.

![undefined](https://cdn.buttercms.com/LO0FuyLLQBuQEUGMbLL2)

**DAG Work Challenge** - Kaspa's BlockDAG allows multiple blocks to be created in parallel, forming a [Directed Acyclic Graph](https://kaspa.com/learn-kaspa/post/dag-and-kaspa) rather than a linear chain.  In this structure, blocks can have multiple parents and there are multiple valid paths through the DAG, making traditional work calculation impossible.  The system needs a way to measure cumulative work that accounts for parallel blocks while maintaining consensus on which path represents the "main chain."

## Blue Score - Counting Consensus Blocks

**Blue Score Definition** - Blue Score represents the count of "blue" blocks in a block's past within the DAG structure.  Blue blocks are those that are part of the selected subgraph chosen by the [GHOSTDAG](https://kaspa.com/learn-kaspa/post/ghostdag-simplified) protocol, essentially the blocks that contribute to the main consensus chain.  Think of Blue Score like counting the number of "valid" blocks that led to the current state, similar to block height in a linear blockchain but accounting for parallel blocks.

![undefined](https://cdn.buttercms.com/LmCePYd7Q8CnHVE2ayVE)

**How Blue Score Works** - When a new block is processed, the GHOSTDAG algorithm determines which blocks in its [Mergeset](https://kaspa.com/learn-kaspa/post/parents-vs-mergeset) should be colored "blue" (part of the consensus) or "red" (valid but not part of the main path).  The Blue Score is calculated by taking the selected parent's Blue Score and adding the number of new blue blocks in the current block's Mergeset.  In this example, k=0, so only the selected parent of B is blue.  The Blue Score of B is 2, the number of blue blocks in its past, by taking the selected parents Blue Score (1), and adding the number of blue blocks in the Mergeset of B (+1).

![undefined](https://cdn.buttercms.com/eQgqyvlOSPqb6s26tjfr)

**Blue Score in Block Headers** - Every block header contains its Blue Score as a field, which must match the value calculated by the GHOSTDAG protocol.  During validation, the consensus system verifies that the header's Blue Score matches the computed value.

## Blue Work - Measuring Cumulative Proof-of-Work

**Blue Work Definition** - Blue Work represents the accumulated proof-of-work of all blue blocks in a block's past, providing a measure of the total computational effort behind the current consensus state.  Unlike Blue Score which simply counts blocks, Blue Work accounts for the actual difficulty and computational work of each blue block.

**Blue Work Calculation** - Blue Work is calculated by summing the individual work contributions of all blue blocks in the Mergeset, where each block's work is derived from its difficulty bits.

**Blue Work for Chain Selection** - Blue Work serves as the primary metric for determining which blocks and chains have the most proof-of-work behind them.  When the virtual processor searches for the new sink (the block with highest accumulated work), it uses Blue Work to order and compare candidates.

## How Blue Score and Blue Work Enable Consensus

**Selected Parent Selection** - When a block has multiple parents, the system chooses the "selected parent" as the one with the highest Blue Work.  This creates a main chain through the DAG that represents the path with the most accumulated proof-of-work.

**Virtual State Resolution** - The consensus system maintains a "virtual" state representing the current tip of the blockchain.  This virtual state is determined by finding the block with the highest Blue Work that also has valid UTXO state, ensuring both security (most work) and validity (correct state).

**Fork Resolution** - When competing chains exist in the DAG, Blue Work provides an objective way to determine which chain should be considered canonical.  The chain with higher accumulated Blue Work is preferred, similar to Bitcoin's longest chain rule but adapted for the DAG structure.

## Simplified Definitions

**Blue Score** - The count of blue (consensus-contributing) blocks in a block's past within the DAG structure.

**Blue Work** - The accumulated proof-of-work of all blue blocks in a block's past.

**Selected Parent** - The parent block with the highest Blue Work, forming the main chain through the DAG.

**Virtual State** - The current consensus state determined by the block with highest Blue Work and valid UTXO state.

Blue Score and Blue Work are Kaspa's way of measuring consensus strength in a parallel block environment.

## Bitcoin vs Kaspa

**Bitcoin** - Uses cumulative work on a single linear chain.  Each block has exactly one parent, making work calculation straightforward by summing difficulty from Genesis to Tip.

**Kaspa** - Uses Blue Work to measure cumulative work across a DAG structure.  Multiple parallel blocks can contribute to the total work, but only "blue" blocks (chosen by GHOSTDAG) count toward the cumulative Blue Work measurement.