# Kaspa's DAA Window

## What is the DAA Window?

The DAA (Difficulty Adjustment Algorithm) Window is Kaspa's system for keeping mining difficulty balanced as network hashrate changes. It looks at recent blocks to calculate how hard the next block should be to mine, ensuring blocks arrive at a steady pace even when miners join or leave the network.

## The Challenge: Maintaining Efficiency at High Block Rates

When you mine cryptocurrency, the network needs to adjust difficulty to maintain consistent block times.

Bitcoin checks 2,016 blocks (~2 weeks) to adjust difficulty. This works fine at 1 block per 10 minutes, but Kaspa produces blocks much faster - 10 blocks every second after the Crescendo upgrade.

At 10 BPS, checking every single block in a 44-minute window would mean processing 26,410 blocks for each difficulty calculation. This would be computationally expensive and slow down the network.

The DAA Window solves this through Sampling - it only looks at every 40th block, reducing the calculation from 26,410 blocks down to just 661 sampled blocks while maintaining the same 44-minute time coverage.

## How the DAA Window Works

### The Sampling System

Post-Crescendo, Kaspa uses a sampled window instead of checking every block:

- **Window size**: 661 sampled blocks
- **Sample rate**: Every 4 seconds (40 blocks at 10 BPS)
- **Time coverage**: ~44 minutes (2,641 seconds)

The network selects blocks using a simple rule: only blocks whose DAA score is divisible by 40 get included in the window.

### Difficulty Calculation Process

Every new block gets a difficulty calculation, but the calculation uses the sampled window.

When you mine a new block:

1. The network inherits the 661-sample window from your selected parent block
2. If your block qualifies for sampling (DAA score divisible by 40), it gets added to the window
3. The difficulty calculation uses this window to determine the next block's target

### The Difficulty Formula

The network calculates difficulty by comparing actual time vs. expected time: new_difficulty = average_difficulty * (measured_time / expected_time)

- **Measured time**: Time span between oldest and newest blocks in the window
- **Expected time**: 100ms × 40 blocks/sample × 661 samples = 2,644 seconds (~44 minutes)

If blocks arrive faster than expected, difficulty increases. If blocks arrive slower, difficulty decreases.

## How the Network Uses This

### During Block Validation

When validating a new block, the network checks that the block's difficulty matches the calculated value:

1. Build the DAA window for the block's GHOSTDAG data
2. Calculate expected difficulty bits from the window
3. Verify the block's difficulty field matches
4. Reject blocks with incorrect difficulty

### For Reward Distribution

The DAA window determines which blocks receive full mining rewards:

- Blocks **inside the DAA window** (within ~44 minutes): Receive full rewards (coinbase subsidy + transaction fees)
- Blocks **outside the DAA window** but within merge depth: Receive only transaction fees (no coinbase subsidy)

This ensures miners are rewarded proportionally to their contribution to network security through difficulty calculation.

## Why This Matters

### Enables 10 Blocks Per Second

Without sampling, processing 26,410 blocks for each difficulty calculation would slow down the network significantly. Sampling reduces this to 661 blocks - a 40x improvement - while maintaining accuracy.

### Maintains Fair Mining

The 44-minute window responds quickly to hashrate changes while being long enough to prevent manipulation. An attacker trying to lower difficulty by mining slowly would need to sustain the attack for the entire window duration, which the merge depth bound (1 hour) prevents.

### Smooth Difficulty Adjustments

Every block gets a fresh difficulty calculation based on recent network performance. This means difficulty adjusts dynamically and smoothly rather than in sudden jumps, keeping block times stable even as miners join or leave.

### Protection Against Attacks

The combination of the DAA window (~44 minutes) and merge depth bound (1 hour) prevents attackers from manipulating the network. Blocks can only be merged if they're within the 1-hour window, which isn't enough time to meaningfully manipulate the 661-sample difficulty window.

## Key Takeaways

- **661 sampled blocks** - covers 44 minutes of network activity with efficient computation
- **Every 40th block sampled** - reduces processing from 26,410 blocks to 661 samples at 10 BPS
- **Dynamic adjustment** - difficulty recalculates with every new block for smooth transitions
- **Time-based security** - 44-minute window prevents manipulation while responding to real hashrate changes
- **Enables high throughput** - makes 10 BPS computationally feasible without sacrificing accuracy
- **Fair reward distribution** - only blocks that contributed to difficulty calculation receive full rewards