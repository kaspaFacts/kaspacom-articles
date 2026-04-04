# Introductory Level - Kaspa Uses GHOSTDAG - What is that?

What is GHOSTDAG and how does Kaspa use it?

A consensus protocol that orders blocks in a [DAG](https://www.kaspa.com/learn-kaspa/post/dag-and-kaspa) structure while maintaining security properties.

What does that mean? This article explains GHOSTDAG by starting with traditional consensus, then GHOSTDAG's approach, how it classifies blocks, and how it differs between Bitcoin and Kaspa.

## Traditional Consensus - Linear Chain Ordering

Traditional blockchain consensus operates on a linear chain where blocks form a single sequence.  Each block has exactly one parent, creating a straightforward ordering mechanism.  When conflicts arise (multiple blocks at the same height), the network selects one block and discards others as orphans.  This approach ensures clear ordering but limits throughput since only one block can be accepted at each level.

![bitcoin orphan.webp](https://cdn.buttercms.com/qiUhW8JRnGjEV02RcEp0)

## GHOSTDAG Protocol - DAG Consensus

GHOSTDAG extends consensus to work with [Directed Acyclic Graph (DAG)](https://www.kaspa.com/learn-kaspa/post/dag-and-kaspa) structures where blocks can have multiple parents.  The protocol processes blocks by first selecting a parent with the highest Blue Work, then examining all blocks in the Mergeset to classify them as either "blue" (honest) or "red" (potentially conflicting).  This classification is based on mathematical constraints involving the security parameter K, which limits the size of Anticones to maintain security properties.

![kaspa DAG.webp](https://cdn.buttercms.com/P31XzHa9RySKXmCLTazx)

## Block Classification Rules

GHOSTDAG classifies blocks using two key constraints related to the security parameter K.  First, the number of blue blocks in the Anticone of a candidate block must not exceed K blocks.  Second, for every existing blue block, adding the candidate must not cause any blue block's Anticone to exceed K blocks.  The algorithm maintains Anticone size tracking to efficiently validate these constraints during block processing.  The gray block here is currently being validated by the network, block C is its Selected Parent.  If k=0, then the chain block C is already 1 blue block, resulting in block B being classified as Red.  If k = 1 (or greater), block B is classified as blue, since it only has 1 blue block (block C) in its Anticone.

![b c anticone.webp](https://cdn.buttercms.com/gAO7lt5DQzOi7cRMCgP0)

## Blue Work Accumulation

The protocol accumulates proof-of-work only from blue blocks, creating the Blue Work metric.  Blue blocks contribute their computational work to the cumulative security score, while red blocks are excluded from this calculation.  This selective accumulation ensures that only consensus-valid blocks contribute to network security, preventing malicious or conflicting blocks from undermining the system.  In this example, lets assume block B is red (k=0), the blue work of our gray block would be calculated as block C inherited blue work, plus block C blue work.  If block B is blue, our new block blue work would inherit the blue work of its Selected Parent (block C), then add the blue work of its Selected Parent (C) and the blue work of blue blocks in its Mergeset (block B).

![b c anticone.webp](https://cdn.buttercms.com/gAO7lt5DQzOi7cRMCgP0)

## Parent Selection and Ordering

GHOSTDAG determines block ordering through parent selection based on Blue Work values.  The protocol selects the parent with the highest accumulated Blue Work as the "Selected Parent," creating a backbone chain within the [DAG](https://www.kaspa.com/learn-kaspa/post/dag-and-kaspa) structure.  Block ordering uses Blue Work as the primary criterion, with the blocks header hash providing deterministic tiebreaking <span role="link" tabindex="0">ordering</span>.  In our example, we assume block C is the Selected Parent and block B is blue.  The ordering for transaction processing is 1. Selected Parent (C) 2. Ordered Mergeset (B)

![b c anticone.webp](https://cdn.buttercms.com/gAO7lt5DQzOi7cRMCgP0)

## Data Storage and Management

The protocol stores classification results in structured data containing blue and red block lists.  Blue blocks are added with Anticone size tracking for future classification decisions, while red blocks are simply appended to the red list.  This organization maintains complete DAG information while clearly distinguishing consensus roles.

## Simplified Definitions

**Traditional Consensus** - A linear chain ordering system where blocks form a single sequence with one parent per block.

**GHOSTDAG Protocol** - A DAG consensus mechanism that classifies blocks as blue or red based on Anticone size constraints.

**Block Classification** - The process of determining whether blocks are blue (consensus-valid) or red (potentially conflicting).

**Blue Work Accumulation** - Selective proof-of-work counting that only includes work from blue blocks.

GHOSTDAG is a consensus protocol that enables [DAG](https://www.kaspa.com/learn-kaspa/post/dag-and-kaspa) structures while maintaining blockchain security properties.

## Bitcoin and Kaspa

**Bitcoin** - Uses traditional linear chain consensus where blocks form a single sequence.  Conflicting blocks are orphaned and contribute no security.  The longest chain (most accumulated work) determines consensus through a simple comparison mechanism.

![bitcoin orphan.webp](https://cdn.buttercms.com/qiUhW8JRnGjEV02RcEp0)

**Kaspa** - Uses GHOSTDAG protocol to handle [DAG](https://www.kaspa.com/learn-kaspa/post/dag-and-kaspa) structures with multiple concurrent blocks.  Blue blocks contribute to security through Blue Work accumulation, while red blocks remain in the DAG but are excluded from consensus decisions.  The protocol maintains both block types for complete network state tracking.

![kaspa all data.webp](https://cdn.buttercms.com/4tsNNkRsej8ZwaJQDywf)