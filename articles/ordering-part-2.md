## Intermediate Level - What is Mergeset Creation?

Mergeset creation is the process by which Kaspa's GHOSTDAG protocol determines which blocks to include alongside a [Selected Parent](https://kaspa.com/learn-kaspa/post/parent-chain) when forming consensus.  Unlike traditional blockchains that only reference a single parent, Kaspa creates a Mergeset containing multiple blocks that can coexist without violating consensus rules.

![parent chain.webp](https://cdn.buttercms.com/mMzceOBWSpGN2n2wtzgr)

## The Discovery Algorithm

The Mergeset creation process begins with a breadth-first traversal algorithm that discovers all candidate blocks.  Starting from the block's direct parents (excluding the Selected Parent), the algorithm:

1. **Initializes the search**: Creates a queue with all non-selected parents
2. **Traverses backwards**: For each block in the queue, examines its parents
3. **Applies exclusion rules**: Skips blocks that are already in the Mergeset or in the past of the Selected Parent
4. **Expands the set**: Adds qualifying blocks to both the Mergeset and processing queue

The key insight is that blocks "in the past" of the Selected Parent are excluded.

Here block B is having its Mergeset created.  The Selected Parent of B in blue, and the Past of the Selected Parent labeled SP.P (everything in gray is also in the past of the Selected Parent, so we can simplify this example), and the Mergeset of block B is in green.  note: The Parent Inclusive Mergeset = Blue + Green, the Parent Exclusive Mergeset = Green

![mergeset selection.webp](https://cdn.buttercms.com/Fcosu9vtTUmuwmts7EWR)

## Selected Parent Foundation

Before Mergeset creation begins, the protocol identifies the [Selected Parent](https://kaspa.com/learn-kaspa/post/parent-chain) using blue work comparison.  The Selected Parent becomes the anchor point - it's automatically included as the first blue block in the Mergeset and serves as the reference for determining which other blocks can be safely included.

![selecting a parent.webp](https://cdn.buttercms.com/EKkr4BpNRvSPmQRvoNqn)

## Blue and Red Classification

Once candidate blocks are discovered, each undergoes classification through the k-cluster validation process.  This determines whether blocks become:

- **Blue blocks**: Blocks that don't violate the k-parameter security constraint and contribute to consensus
- **Red blocks**: Blocks that would violate k-constraints but are still preserved in the DAG structure

## Deterministic Ordering

The final Mergeset is arranged in a consensus-agreed order that all nodes can reproduce.  Blue and red blocks are merged in ascending blue work order, ensuring deterministic transaction processing across the network.

## Size Constraints

The protocol enforces Mergeset size limits to maintain performance and prevent abuse.  These constraints ensure that block validation remains computationally feasible while still allowing significant parallelism compared to traditional blockchains.

## The Result

The Mergeset creation process produces a structured set of blocks that maximizes inclusion while maintaining consensus safety.  This enables Kaspa to achieve high throughput by processing multiple blocks simultaneously, rather than discarding competing work as traditional blockchains do.

## Simplified Explanation

This blocks Mergeset is made up of all the blocks in the past of this block, that are not in the past of this blocks Selected Parent.