## Intermediate Level - Blue Work Calculation

Blue work represents the accumulated proof-of-work of Blue blocks in Kaspa's [DAG](https://kaspa.com/learn-kaspa/post/dag-and-kaspa) structure.  Note, for simplicity in the illustrations we use Blue Count, Blue Count can represent the Blue Work when work of each block is equal.

### The Blue Work Foundation

Blue Work Calculation builds upon the [Blue/Red Classification](https://kaspa.com/learn-kaspa/post/kaspa-blocks-blue-vs-red).  Blue Work is calculated by summing the individual work contributions of all Blue blocks in a block's [Mergeset](https://kaspa.com/learn-kaspa/post/kaspa-mergeset-creation), plus the accumulated Blue Work from its [Selected Parent](https://kaspa.com/learn-kaspa/post/parent-chain).

### Selected Parent as Starting Point

Just like Blue/Red Classification, Blue Work Calculation begins with finding the Selected Parent - the parent with the highest Blue Work.  This Selected Parent provides the base Blue Work value that gets extended by the current block's Blue Mergeset.  For simplicity, we will illustrate all work as being equal (Blue Count).  Here we see the Selected Parent has a Blue Score of 25.  This is the total work in the past of B.SP, note this does not include B.SP itself.

![undefined](https://cdn.buttercms.com/wQKsVWcxRHSEmTDTtrll)

### The Calculation Process

The Blue Work Calculation happens during GHOSTDAG processing in three key steps:

**Step 1: Individual Block Work Calculation**<br>Each Blue block in the Mergeset contributes its individual proof-of-work, calculated from the block's difficulty bits.  The system uses a minimum threshold to ensure blocks meet a baseline work requirement.  For simplicity we illustrate calculating the blue count of each block in the Mergeset.  Here you can see the each block in the Mergeset has its individual contribution calculated.

![undefined](https://cdn.buttercms.com/eQUj3nUcQemh6u7sWSHh)

**Step 2: Accumulation from Selected Parent**<br>The algorithm retrieves the accumulated Blue Work from the Selected Parent, which represents all the work from Blue blocks in the Past of the Selected Parent.

![blue count step 1.webp](https://cdn.buttercms.com/wQKsVWcxRHSEmTDTtrll)

**Step 3: Final Blue Work Calculation**<br>The final Blue Work equals the Selected Parent's Blue Work plus the sum of all Blue blocks' individual work in the current Mergeset.  Taking the Selected Parents blue count of 25, we add the work of each individual block, totaling 2, and get 27.  The Blue Score of block B is 27, this represents the total work this block is built on top of.

![undefined](https://cdn.buttercms.com/q2E46MLgRoaXP1saMiaf)

### Storage and Access

The calculated Blue Work gets stored in the GHOSTDAG Data structure alongside other consensus data.  This enables efficient retrieval during block ordering operations, where blocks are sorted by Blue Work for consensus decisions.

### Block Ordering and Tie-Breaking

Blue Work serves as the primary ordering mechanism for blocks in the DAG.  When blocks have equal Blue Work, Kaspa uses Block Hash as a tie-breaker to ensure deterministic ordering.  This block B now has a Blue Count value for comparing it to other blocks when the next round of blocks is created with B as one of the tips.  We can now restart the [ordering process back at step 1](https://kaspa.com/learn-kaspa/post/parent-chain).

![undefined](https://cdn.buttercms.com/drIOPRqZTGeooOv8sBeE)

### The Result

Blue Work Calculation produces a cumulative security metric that represents the total proof-of-work backing a particular path.  This enables Kaspa to maintain Bitcoin-level Security while achieving high throughput through parallel block processing.  Only Blue blocks contribute to this security metric, ensuring that conflicting (Red) blocks don't dilute the chain's security guarantees.

### Simplified Explanation

Blue work is like a "security score" that adds up all the computational effort (proof-of-work) from honest blocks in the chain.  Each Blue block contributes its mining difficulty to this running total, creating a measure of how much work attackers would need to overcome the honest chain.  Red blocks don't count toward this score, so the network maintains security even when including many parallel blocks.