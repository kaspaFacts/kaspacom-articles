# Intermediate Level - K Parameter

The k parameter is a magic number, but what exactly is k, why is it required, and how is it determined?

## The K Parameter Foundation

The k parameter is a fundamental security parameter in [GHOSTDAG](https://kaspa.com/learn-kaspa/post/ghostdag-simplified) that defines the maximum allowed Blue [Anticone](https://kaspa.com/learn-kaspa/post/dag-terminology)size.  It serves as a critical security constraint that prevents k-cluster violations by limiting the maximum number of conflicting [Blue Blocks](https://kaspa.com/learn-kaspa/post/blue-work-ordering-pt4) that can exist in any block's Anticone, ensuring network integrity while enabling parallel block processing and maintaining optimal network security with high throughput.

## Anticone Size Probability Constraint

The k parameter is calculated using a probabilistic formula that ensures Anticones larger than k occur with probability less than a specified threshold δ (delta).  With Kaspa, it is set so that the probability of Anticones larger than k occur less than 1% of the time.

### The Core Equation

The calculation follows equation 1 from section 4.2 of the PHANTOM GHOSTDAG paper, where k is the minimal value such that:

P(anticone size > k) < δ

Or... The probability of Anticones greater than k is less than delta (1%)

This is computed iteratively using the Poisson distribution, meaning plug in each possible k value (starting at 0) until we get an output of less than 1%.

### Input Parameters and Their Reasoning

**Network Delay Parameter (x = 2Dλ):**

- D represents the maximal network delay (max time to propagate a block to every node)
- λ represents the block mining rate (blocks per second)
- The factor of 2 accounts for the round-trip (or more accurately "-t")
- This parameter captures the fundamental relationship between network latency and block production rate

**Security Threshold (δ):**

- Represents the acceptable probability of k-cluster violations
- Typically set to a small value (Kaspa uses 0.01 or 1%)
- Lower values provide higher security but require larger k values
- This threshold directly impacts the security guarantees of the consensus protocol

## The Calculation Process

The k parameter determination happens through iterative probability calculation using the Poisson distribution:

### Step 1: Set Starting Point

The algorithm begins with k=0 and calculates a starting probability value based on the network conditions.  (Using x as an input, where x = 2Dλ)

### Step 2: Test Each k Value

For each possible k value (0, 1, 2, 3...), the algorithm calculates the probability of having that many conflicting blocks.  It builds up a running total of these probabilities, checking how likely different Anticone sizes are to occur.

### Step 3: Find the Security Sweet Spot

The algorithm looks for the point where the chance of having more than k conflicting blocks drops below δ (delta), in Kaspa delta is 1%.  Once it finds this threshold, that becomes the chosen k value.

**Why This Approach Works:**

- Uses mathematical shortcuts to avoid complex calculations
- Incrementally builds up the probability picture
- Stops when it reaches the desired security level (99% confidence that Anticones won't exceed k)

## Network-Specific K Values

Different network configurations require different k values based on their block production rates.

Higher throughput networks (more blocks per second) require proportionally larger k values to maintain the same security level, as faster block production increases the likelihood of simultaneous block creation and larger Anticones.

## Security Implications

The k parameter directly affects network security by controlling the maximum size of conflicting block sets.  A properly calculated k ensures that:

- Honest blocks maintain consensus despite network delays
- Attackers cannot exploit parallelism to break consensus
- The network can process high transaction volumes safely

## The Result

Proper k parameter calculation produces a security constraint that maintains protocol integrity while enabling high-throughput parallel processing.  The mathematical foundation ensures that even under adverse network conditions, the probability of consensus violations remains below acceptable thresholds.

## Simplified Explanation

The k parameter is like a "safety buffer" calculated using network speed and desired security level.  By considering how long messages take to travel across the network and how fast blocks are created, the formula determines the maximum number of conflicting blocks that can safely coexist while keeping the chance of security problems below an acceptable threshold.