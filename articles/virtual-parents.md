## What are Virtual Parents?

Virtual Parents are the direct Parent Blocks of Kaspa's [Virtual Block](https://kaspa.com/learn-kaspa/post/virtual-block). They are dynamically selected to maintain Consensus Rules and Network Health, enabling the network to process blocks with bounded processing requirements while maintaining Security Properties in Kaspa's [blockDAG](https://kaspa.com/learn-kaspa/post/dag-and-kaspa) structure.

## The Challenge: Managing Consensus in a BlockDAG

In a traditional blockchain, there's a single chain tip, but Kaspa's blockDAG allows multiple blocks to coexist. This creates core challenges for determining which blocks represent the current State and which should be built upon next.

**Multiple Competing Tips**: Without a mechanism to select parents, miners wouldn't know which blocks to build on, leading to fragmented Security and reduced collective Security Benefit.

**Performance Degradation**: Including too many Parent Blocks would cause quadratic growth in processing requirements and storage needs, making the network unsustainable at high throughput. The maximum number of parents represents a deliberate balance between performance and accessibility - while higher limits could increase throughput, they would also raise minimum hardware requirements. Kaspa's design prioritizes keeping node requirements low to ensure Broad Participation and Decentralization.

**Finality and Security**: The system must ensure that [Selected Parents](https://kaspa.com/learn-kaspa/post/parents-vs-mergeset) respect [Finality](https://kaspa.com/learn-kaspa/post/finality-depth) constraints and don't allow attacks that could rewrite deeply buried transaction history.

Virtual Parents solve these problems by selecting a limited set of Parent Blocks that maintains Security constraints while optimizing Performance and Network Convergence through a constraint-based selection algorithm.

## How Virtual Parents Work

### Sink Search Algorithm

The first step finds the [Sink](https://kaspa.com/learn-kaspa/post/sink-selection). This algorithm searches for the highest [Blue Work](https://kaspa.com/learn-kaspa/post/blue-score-and-blue-work) Block with valid [UTXO](https://kaspa.com/learn-kaspa/post/kaspa-utxo-model) State that respects Finality constraints.

- **Antichain Maintenance**: Candidates are blocks where none are [ancestors](https://kaspa.com/learn-kaspa/post/dag-terminology) of another
- **Blue Work Ordering**: Candidates are processed in descending Blue Work order
- **UTXO Validation**: Each candidate is validated by ensuring it has valid UTXO State
- **Finality Checks**: Ensures candidates don't violate Finality constraints

The Sink becomes the first Virtual Parent and the foundation for additional parent selection.

### Virtual Parent Selection

Once the Sink is found, additional Parents are selected from remaining candidates through a constraint-based process that enforces network diversity while maintaining performance requirements.

The algorithm considers a multiple of the Parent Limit for candidates and uses randomization for lower-ranked candidates to ensure network diversity. The selection prioritizes candidates by Blue Work while applying [Mergeset Size](https://kaspa.com/learn-kaspa/post/mergeset-size-limit) and Parent Count constraints to maintain network health.

### Constraint Validation

The selection process enforces multiple constraints to ensure network health:

1. **Mergeset Size Limit**: Calculates if adding a candidate would exceed the limit for the set of blocks that will be merged into the virtual chain, ensuring manageable memory requirements for nodes
2. **Parent Count Limit**: Enforces the maximum of parents to keep processing requirements low and maintain broad node accessibility
3. **Bounded Merge Depth**: Removes parents that would violate the [Merge Depth Bound](https://kaspa.com/learn-kaspa/post/merge-depth-bound)
4. **Finality depth**: Ensures all parents respect the Finality window

## How the Network Uses This

### Block Template Construction

Miners use Virtual Parents as the foundation for building new blocks. The Virtual Parents become the parent references in new Block Headers, ensuring all miners build on the same consensus foundation.

This creates a unified mining target while still allowing the blockDAG structure to accommodate multiple parallel blocks, providing both Security and High Throughput.

### UTXO State Management

The Virtual Block's UTXO State is calculated by inheriting the Sink's State and applying the [Mergeset](https://kaspa.com/learn-kaspa/post/parents-vs-mergeset) of blocks, derived from all Virtual Parents. This ensures the network maintains a consistent view of spendable outputs.

Formula: Virtual UTXO State = Sink UTXO State + Changes from Merged Blocks

This allows State Updates with minimal computational overhead without requiring full chain reorganizations when the Virtual Parent Set changes.

### Network Synchronization

Virtual Parents serve as the synchronization point for network nodes. When nodes communicate their Virtual Parent Sets, they can determine with minimal latency if they're on the same Consensus View or need to request missing blocks.

## Why This Matters

### High Throughput Performance

Virtual Parents enable Kaspa to achieve high throughput while maintaining Security and keeping node requirements low. The Parent Limit prevents quadratic growth in processing requirements, ensuring broad participation across the network.

### Network Convergence

By selecting Parents based on Blue Work and using randomization for diversity, Virtual Parents ensure the network reaches agreement on a single Consensus View without central coordination, maintaining Decentralization while enabling rapid Finality.

### Security Guarantees

The multi-layered constraint system (Mergeset Limits, Merge Depth Bounds, Finality Constraints) ensures that Virtual Parent selection cannot be manipulated to attack the network or rewrite transaction history, providing robust Security Properties even at high throughput.

## Key Takeaways

- **Dynamic Selection** - Virtual Parents are not fixed but dynamically selected based on network conditions and block topology
- **Performance Optimized** - Limited to a maximum number of Parents to prevent quadratic growth in processing requirements and maintain low node hardware requirements
- **Multi-Constraint System** - Selection enforces Mergeset Size, Merge Depth, and Finality Constraints for Security
- **Network Foundation** - Serves as the Consensus Foundation for mining, UTXO Management, and Node Synchronization
- **Configurable Design** - Parameterized by Network Throughput Rate, allowing adaptation to different Network Configurations