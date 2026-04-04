# Introductory Level - DAG Terminology

Past, Future, Anticone, Mergeset, K parameter, what does it all mean?

Past, Future, and Anticone are DAG terms, while Mergeset and K are used in GHOSTDAG.

[DAG](https://www.kaspa.com/learn-kaspa/post/dag-and-kaspa) terminology is a specialized vocabulary for describing relationships in a BlockDAG structure.  We start with Linear Chain, DAG, and then a touch of GHOSTDAG.

## Linear Chain Terminology - Traditional Blockchain

Linear chain terminology uses simple concepts where blocks form a single sequence.  Each block has one parent and potentially one child, creating straightforward ancestor-descendant relationships.  Terms like "height," "previous block," and "next block" describe the linear progression.  When conflicts occur, blocks are either "accepted" into the main chain or "orphaned" and discarded.

![undefined](https://cdn.buttercms.com/ztcWf0UT4ysiEhzt0WSE)

## DAG Terminology -

Allowing blocks to have multiple parents creates new relationships within the [DAG](https://www.kaspa.com/learn-kaspa/post/dag-and-kaspa).

![kaspa DAG.webp](https://cdn.buttercms.com/P31XzHa9RySKXmCLTazx)

## Past and Future Relationships - DAG

Past relationship defines all blocks reachable by following parent links backward from a given block.  A block is in the past of another if there exists any directed path connecting them.  Future relationship works inversely - if block A is in the past of block B, then B is in the future of A.

![undefined](https://cdn.buttercms.com/oWhSigCmRPShKmCFE1IA)

## Anticone Relationship - DAG

Anticone describes blocks that are neither ancestors nor descendants of each other - they exist concurrently in the [DAG](https://www.kaspa.com/learn-kaspa/post/dag-and-kaspa).  Two blocks are in each other's Anticone if neither can reach the other through any directed path.  This relationship is crucial for GHOSTDAG's security parameter K, which limits Anticone sizes to maintain consensus safety.  Here, Block A and Block C are in each others Anticone, Block A is not reachable from Block C, and Block C is not reachable from Block A.

![undefined](https://cdn.buttercms.com/RNhZNbRUmeGmQUUWU9aA)

## Mergeset and Blue/Red Classification - GHOSTDAG

Mergeset refers to the collection of blocks that are merged when creating a new block.  The Mergeset contains the direct parents of a block, but can additionally contain blocks that are not direct parents.  GHOSTDAG classifies Mergeset blocks as "Blue" (honest) or "Red" (potentially conflicting) based on Anticone size constraints.  This classification determines which blocks contribute to network security through Blue Work accumulation.  Here is an example of block B classifying its Mergeset into Blue and Red while the Anticone size constraint = 0.

![1 blue 1 red.webp](https://cdn.buttercms.com/gYCi4AK2QCG1HRd7UzmB)

## K Parameter - GHOSTDAG

The parameter K controls the maximum allowed Anticone size for blue blocks.  This parameter is calculated based on network delay, block production rate, and desired security guarantees.  In this example, instead of k = 0 like the example above, k = 1 so each blue block has 1 other blue block in its Anticone.

![2 blue.webp](https://cdn.buttercms.com/6yB2YGeOSPaBOD8zBJLY)

## Simplified Definitions

**Past Relationship** - All blocks reachable by following parent links backward from a given block.

**Future Relationship** - All blocks that can reach a given block by following parent links forward.

**Anticone Relationship** - Blocks that are neither ancestors nor descendants of each other.

**Mergeset** - GHOSTDAG's collection of blocks merged when creating a new block.

**Blue/Red Classification** - GHOSTDAG's categorization of blocks as honest (blue) or potentially conflicting (red).

**Security Parameter K** - GHOSTDAG's maximum allowed anticone size for maintaining consensus safety.

## Bitcoin and Kaspa

**Bitcoin** - Uses simple linear terminology: "previous block," "next block," "chain height," and "longest chain."  Relationships are straightforward ancestor-descendant connections.  Competing blocks are "orphaned" with no intermediate states.

![bitcoin all data.webp](https://cdn.buttercms.com/FErgQ6EIQgyRfxUX0dK4)

**Kaspa** - Uses additional terminology including [DAG](https://www.kaspa.com/learn-kaspa/post/dag-and-kaspa) Past/Future/Anticone relationships, GHOSTDAG Mergeset, and Mergeset Blue/Red Classification.  Kaspa maintains multiple concurrent blocks, handles their relationships and provides consistent ordering.

![kaspa all data.webp](https://cdn.buttercms.com/4tsNNkRsej8ZwaJQDywf)