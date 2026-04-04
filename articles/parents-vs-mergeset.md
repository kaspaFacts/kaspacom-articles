# Introductory Level - Kaspa Uses Parents and Mergeset - What's the difference?

What are Parents and Mergeset and how does Kaspa use them?

Two different ways of describing block relationships in Kaspa's BlockDAG structure.

What does that mean? This article assumes no prior knowledge, so we start with traditional blockchain parent relationships, then explain how Kaspa's [DAG](https://www.kaspa.com/learn-kaspa/post/dag-and-kaspa) structure creates multiple types of block relationships, what Parents and Mergeset represent, how they're calculated, and how they work together to enable consensus in a parallel block environment.

## Traditional Blockchain Parents vs DAG Relationships

**Traditional Blockchain Parents** - In a linear blockchain like Bitcoin, each block has exactly one parent (except Genesis), creating a simple chain structure.  The parent relationship is straightforward: each new block references the hash of the previous block, forming an unbroken sequence from Genesis to the current tip.

![bitcoin all data.webp](https://cdn.buttercms.com/FErgQ6EIQgyRfxUX0dK4)

**DAG Parent Complexity** - Kaspa's BlockDAG allows blocks to have multiple parents, creating a more complex web of relationships.  When a block is created, it can reference multiple existing blocks as parents, enabling parallel block creation and higher throughput.

![undefined](https://cdn.buttercms.com/4tsNNkRsej8ZwaJQDywf)

## Parents - Direct Block References

**Parents Definition** - Parents are the blocks that a new block directly references in its header.  These are explicit relationships declared by the block creator - they're the blocks this new block is directly building upon.  When viewing a Kaspa [DAG](https://www.kaspa.com/learn-kaspa/post/dag-and-kaspa) Visualizer, these arrows represent the parent relationship.

![kaspa all data.webp](https://cdn.buttercms.com/4tsNNkRsej8ZwaJQDywf)

**How Parents Work** - When creating a block, miners select which existing blocks to reference as parents based on what they see as the current tips of the [DAG](https://www.kaspa.com/learn-kaspa/post/dag-and-kaspa).  The system validates these parent relationships and uses them to determine the block's position in the DAG structure.  Here you can see a new block being created, referencing DAG tips, the blocks found without another block pointing to them.

![undefined](https://cdn.buttercms.com/7PzbnWMcSTSDUKDokZjA)

## Mergeset - Consensus-Relevant Blocks

**Mergeset Definition** - The mergeset is the set of blocks that are in the anticone of a block's selected parent but are still considered part of the block's consensus context.  Here, block C is in the anticone of B, and block B is in the anticone of C.

![b c anticone.webp](https://cdn.buttercms.com/gAO7lt5DQzOi7cRMCgP0)

**How Mergeset is Calculated** - The mergeset is computed by finding all blocks that are not ancestors of the selected parent but are still reachable through the block's parent set.  This creates a broader context of blocks that need to be considered for consensus decisions.  In this example, if block B is the selected parent, the mergeset of the block being created would include both block C and block D.

![undefined](https://cdn.buttercms.com/e6wrFls1R1Kcx1i2gs3z)

**Mergeset in GHOSTDAG** - The GHOSTDAG protocol processes the mergeset to determine which blocks should be colored "blue" (contributing to consensus) or "red" (valid but not contributing).  This coloring process is essential for maintaining consensus in the parallel block environment.

## How Parents and Mergeset Work Together

**Selected Parent Selection** - From all the parents, the system selects one as the "selected parent" - the one with the highest blue work.  This creates a main chain backbone through the [DAG](https://www.kaspa.com/learn-kaspa/post/dag-and-kaspa) while still acknowledging the other parent relationships.  Here the parent chain is highlighted.

![undefined](https://cdn.buttercms.com/uJPJ6AStQo6StkloUnfG)

**Mergeset Processing** - Once the selected parent is chosen, the mergeset is calculated and processed to determine the final GHOSTDAG data.  The mergeset excludes the selected parent since it's already accounted for in the main chain.  Here the mergeset includes block C, because it is in the anticone of block B (the selected parent) even though it is not a parent of the new block (parents include only block B and block D).

![undefined](https://cdn.buttercms.com/wp6VvEUvSt2dTadrmFED)

**Virtual Parent Selection** - When creating the virtual state, the system uses both concepts: it picks virtual parents from candidate blocks while ensuring the resulting mergeset doesn't exceed size limits.  This balances including many parallel blocks while maintaining manageable consensus complexity.

## Practical Differences

**Storage and Iteration** - Parents are stored directly in block headers, while mergeset data is computed and stored separately in GHOSTDAG data structures.  The system provides different iterators for accessing mergeset blocks in various orders (consensus order, blue work order, etc.).

**Consensus Impact** - Parents determine the basic [DAG](https://www.kaspa.com/learn-kaspa/post/dag-and-kaspa) structure, but the mergeset determines which blocks actually contribute to consensus calculations like blue score and blue work.  A block might be a parent but end up colored red in the mergeset, meaning it doesn't contribute to the main consensus chain.

## Simplified Definitions

**Parents** - The blocks that a new block directly references in its header, establishing explicit relationships in the [DAG](https://www.kaspa.com/learn-kaspa/post/dag-and-kaspa).

**Mergeset** - The set of blocks in a block's anticone that are considered for consensus processing, excluding the selected parent.

**Selected Parent** - The parent with the highest blue work, forming the main chain backbone.

**Mergeset Blues/Reds** - Blocks in the mergeset that either contribute to consensus (blue) or don't (red).

Parents define the DAG structure, while mergeset determines consensus participation.

## Bitcoin vs Kaspa

**Bitcoin** - Has only one parent per block (except Genesis), so there's no distinction between parents and mergeset. The single parent is both the structural relationship and the consensus relationship.

![bitcoin all data.webp](https://cdn.buttercms.com/FErgQ6EIQgyRfxUX0dK4)

**Kaspa** - Separates structural relationships (parents) from consensus relationships (mergeset).  Multiple parents create the [DAG](https://www.kaspa.com/learn-kaspa/post/dag-and-kaspa) structure, but the mergeset processing determines which blocks actually contribute to the consensus state.

![kaspa mergeset.webp](https://cdn.buttercms.com/wp6VvEUvSt2dTadrmFED)