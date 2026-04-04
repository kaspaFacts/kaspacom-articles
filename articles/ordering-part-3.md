## Intermediate Level - Blue and Red Classification

Blue and red classification determines which blocks contribute to consensus security in Kaspa's [DAG structure](https://kaspa.com/learn-kaspa/post/dag-and-kaspa).  This process ensures network security while maximizing block inclusion.

### The K-Cluster Foundation

The classification uses the k-cluster rule where k represents the maximum [Anticone](https://kaspa.com/learn-kaspa/post/dag-terminology) size allowed between blue blocks.  Two conditions must be met:

- The candidate's blue Anticone size cannot exceed k
- Adding the candidate cannot make any existing blue block's Anticone exceed k

### Selected Parent as Starting Point

Before classification begins, the algorithm finds the [Selected Parent](https://kaspa.com/learn-kaspa/post/parent-chain) - the parent with highest blue work.  This Selected Parent automatically becomes the first blue block and serves as the reference point.

![SP blue by default.webp](https://cdn.buttercms.com/vskIvxP3QdOct8jcvOKj)

### The Classification Process

For each candidate block in the [Mergeset](https://kaspa.com/learn-kaspa/post/kaspa-mergeset-creation), the algorithm applies k-cluster validation.  The process walks through the Mergeset, checking relationships with existing blue blocks.  In the above image, the blue candidates are green (and are ordered using tiebreaking).

**Blue Path**: Blocks that pass validation are added to the blue set.  In the following image, the first blue candidate is checked while k=1.  There is only 1 blue block in the anticone of this candidate block, and adding it as a blue block would not cause the existing blue (Selected Parent) have too many blues in its anticone (the blue SP would have 1 blue in its anticone if adding the blue candidate as blue), the blue candidate becomes blue.

![undefined](https://cdn.buttercms.com/eVrJIOmURkqe2ZrlVR6m)

**Red Path**: Blocks that would violate k-constraints become red but remain in the DAG.  In the following image, we check the next blue candidate (green block from above).  This blue candidate already has 2 blue blocks in its Anticone (more than 1), this blue candidate becomes red when k=1.

![undefined](https://cdn.buttercms.com/EmH3QBGSC6M053XodiUU)

### Anticone Size Calculation

The system tracks Anticone sizes efficiently by using cached data.  This avoids expensive recomputation.

### K-Cluster Validation Details

The validation examines each blue candidate to ensure adding it to the blue set won't violate constraints.  Blocks in the past of the candidate are skipped since they're not in its Anticone.

### The Result

The classification produces two sets: blue blocks that contribute to consensus security and red blocks that are preserved but don't affect the main chain.  This enables high throughput while maintaining security through the k-cluster rule.

### Simplified Explanation

Blue and red classification sorts blocks into two groups:  (blue) "good" blocks that help secure the network and (red) "questionable" blocks that are kept but don't count toward security.  The system uses a simple rule - if adding a block would create too many conflicts, it becomes red instead of blue.  This way, Kaspa can include more blocks than Bitcoin while still maintaining security by only counting the honest ones.