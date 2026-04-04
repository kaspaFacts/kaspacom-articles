## What are Coinbase Transactions?

Coinbase Transactions are special transactions that create new Kaspa coins as block rewards for miners who successfully mine blocks. They serve as the primary mechanism for coin issuance and miner compensation, containing the block subsidy plus transaction fees from the block's transactions.

## The Challenge: Reward Distribution and Security

Kaspa needs a secure and predictable way to distribute new coins while preventing double-spending and ensuring network stability. The challenge lies in balancing miner incentives with network security and economic policy.

In Kaspa's high-throughput [DAG](https://kaspa.com/learn-kaspa/post/dag-and-kaspa) structure, this becomes more complex because multiple blocks can be mined simultaneously, creating parallel reward chains that must be properly coordinated to prevent duplicate rewards and ensure fair distribution across the network.

**Double Spending Prevention**: Without proper maturity rules, miners could immediately spend newly created coins from Coinbase Transactions. If those blocks are later reorganized, the spending transaction and any subsequent transactions that relied on those Coinbase outputs would become invalid, as their inputs would no longer exist in the [UTXO](https://kaspa.com/learn-kaspa/post/kaspa-utxo-model) set.

**Network Security**: The reward mechanism must incentivize honest mining behavior and participation in Consensus, just as in any Proof-of-Work system. Miners are compensated for their computational work and for following the Consensus rules, which Secure the network against attacks and ensures reliable transaction processing.

**Economic Policy**: The subsidy schedule must be predictable and enforceable to maintain the network's monetary policy. The [DAA Score](https://kaspa.com/learn-kaspa/post/daa-score) based calculation ensures a controlled emission rate that transitions from pre-deflationary to deflationary phases at known milestones, preventing unexpected inflation.

Coinbase Transactions solve these problems through three tightly coordinated mechanisms: Maturity Periods (1000 blocks post-Crescendo) that prevent spending reorganized rewards, DAA Score based subsidy calculations enforcing predictable emission schedules, and Validation Rules that verify both subsidy amounts and Blue Score consistency.

## How Coinbase Transactions Work

### Subsidy Calculation

The block subsidy is calculated based on the network's DAA Score using a predictable emission schedule. This ensures controlled coin issuance over time while maintaining network Security through miner incentives.

- **Difficulty-Based**: Subsidy amount depends on the network's current DAA Score
- **Predictable Schedule**: Follows a predefined emission schedule that controls coin creation rate
- **Network Security**: Ensures miners are compensated for Securing the network

The calculation implements the emission schedule logic to determine the appropriate reward for each block based on the network's current state and phase of the monetary policy.

### Coinbase Validation

Every block's first transaction must be a valid Coinbase Transaction that passes multiple validation checks to ensure network integrity and economic consistency.

The validation process enforces two critical checks: the Coinbase payload must contain the correct [Blue Score](https://kaspa.com/learn-kaspa/post/blue-score-and-blue-work) matching the block header, and the subsidy amount must match the expected calculation for that DAA Score.

### Maturity Period

Coinbase Transactions have a maturity period of 1000 blocks before their outputs can be spent, ensuring network stability by preventing immediate use of newly created coins.

1. Block is mined and Coinbase Transaction created with new coins
2. Transaction outputs are marked as immature and unspendable
3. Network must accumulate 1000 blocks after the Coinbase Transaction
4. After maturity period, outputs become spendable in transactions

## How the Network Uses This

### Block Template Generation

When miners request block templates from the network, they provide their own payment address where they want to receive the Coinbase reward. The network validates this address and constructs a complete block template that includes a Coinbase Transaction paying the block subsidy plus transaction fees to the miner's specified address.

This process ensures miners receive their rewards directly in their wallet while maintaining the predictable monetary policy through proper subsidy calculation.

### Network Consensus Integration

Coinbase Transactions are deeply integrated with Kaspa's [Consensus Mechanism](https://kaspa.com/learn-kaspa/post/ghostdag-simplified), ensuring reward calculations align with each block's position in the network's Directed Acyclic Graph. The Blue Score recorded in the Coinbase Transaction must match the block's Consensus State, preventing reward manipulation and maintaining economic consistency across parallel chains.

This integration ensures fair reward distribution even when multiple blocks are mined simultaneously, coordinating rewards across Kaspa's high-throughput structure.

### Network Reliability

The network maintains reliability through comprehensive validation of Coinbase Transactions. Incorrect subsidy calculations are automatically rejected, protecting the monetary policy, while maturity period enforcement prevents network instability from premature spending.

This robust validation ensures that only properly formed Coinbase Transactions are accepted, maintaining network Security and economic stability across all participating nodes.

## Why This Matters

### Network Security

Coinbase maturity periods of 1000 blocks prevent miners from spending rewards before their blocks are sufficiently buried in the network, eliminating the risk of transaction invalidation from chain reorganizations. This DAA based security ensures that even with Kaspa's high-throughput structure, the economic stability of the network remains protected.

### Predictable Monetary Policy

The DAA based subsidy calculation enforces a transparent and unchangeable emission schedule, ensuring that no party can manipulate coin creation rates. This predictable monetary policy provides system reliability by guaranteeing that the economic rules established at launch are mathematically verifiable and cannot be altered through consensus attacks.

### Miner Incentives

Proper reward distribution through Coinbase Transactions creates a sustainable economic model that incentivizes honest mining behavior. Miners are reliably compensated for their computational work and for following consensus rules, which secures the network against attacks and ensures reliable transaction processing for all users.

## Key Takeaways

- **Block Rewards** - Coinbase Transactions create new coins as rewards for mining blocks, combining subsidy plus transaction fees
- **DAA-Based Calculation** - Rewards are calculated based on the network's DAA Score using a predictable emission schedule
- **Maturity Period** - Coinbase outputs require 1000 blocks of confirmation before becoming spendable
- **Validation Rules** - Strict checks ensure Blue Score and Subsidy amounts match expected values
- **Network Integration** - Coinbase Transactions are essential to Kaspa's DAG Consensus and Economic Security Model