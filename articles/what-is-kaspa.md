# Conceptual Overview of Kaspa

Kaspa is a cryptocurrency that uses [BlockDAG](https://kaspa.com/learn-kaspa/post/dag-and-kaspa) technology to achieve high-speed transactions while maintaining Security and Decentralization.

## Trilemma Solved

• **Scalability**: Parallel block creation theoretically removes throughput limitations (Limited only by target hardware and network latency).

• **Decentralization**: Lower barriers to mining participation through consistent rewards (864,000 blocks per day).

• **Security**: Maintains Bitcoin-level Security through Proof-of-Work while enabling higher throughput (2500+/sec) and faster transactions (sub-second).

## Foundation

• **BlockDAG Architecture**: Kaspa uses a [Directed Acyclic Graph](https://kaspa.com/learn-kaspa/post/dag-terminology) (DAG) structure instead of a traditional blockchain, allowing multiple blocks to be created in parallel and reference multiple parents.

• **GHOSTDAG Consensus Protocol**: Orders blocks in the DAG and classifies them as [Blue (contributing to Security) or Red (valid but not contributing to Security)](https://kaspa.com/learn-kaspa/post/kaspa-blocks-blue-vs-red).

• **Unlimited Block Rate Security**: [GHOSTDAG](https://kaspa.com/learn-kaspa/post/ghostdag-simplified) remains secure for ANY block creation rate, enabling transaction throughput that is only limited by hardware.

## How Blocks Are Ordered

• **Block Ordering**: GHOSTDAG creates a linear ordering of blocks from the DAG structure for transaction processing.

• **Security Model**: Blue blocks contribute to Security while Red blocks are still Valid.

• **Fast Finality**: Transactions achieve [Probabilistic Finality](https://kaspa.com/learn-kaspa/post/finality) within seconds as confirmations accumulate, with [Deterministic Finality](https://kaspa.com/learn-kaspa/post/finality) at deeper confirmation levels

## Real World Performance

• **10 Blocks Per Second:** Recent Crescendo upgrade increased from 1 to 10 BPS while maintaining Security.

• **Instant Confirmations:** Sub-second transactions accumulate Confirmations almost instantly.

• **Throughput:** Multiple simultaneous blocks eliminate network bottlenecks that plague single-chain systems.

## Transaction Architecture

• **UTXO-based**: Uses [Unspent Transaction Output](https://kaspa.com/learn-kaspa/post/kaspa-utxo-model) model for efficient validation.

• **Programmable Transactions**: Supports scripting for complex transaction logic.

• **Fast Confirmation**: Parallel processing enables near-instant transaction confirmation.

## Mining and Fees

• **Proof of Work**: Uses a unique ASIC-friendly hash algorithm modified to be distinct from other cryptocurrencies.

• **Mass-Based Fees**: Transaction fees calculated using a multi-dimensional Mass system considering compute, storage, and network costs for fair pricing.

• **Consistent Mining Rewards**: 864,000 blocks daily provide predictable income for all miners, reducing dependence on large mining pools.

## Sustainable Growth

• **Automatic [Pruning](https://kaspa.com/learn-kaspa/post/first-order-pruning-in-kaspa)**: Removes old blockchain data while preserving cryptographic security proofs, keeping storage requirements manageable.

• **Trustless Bootstrap**: New nodes sync quickly using cryptographic proofs instead of downloading complete blockchain history, enabling fast bootstrap [without requiring trust](https://kaspa.com/learn-kaspa/post/kaspa-uses-muhash).

• **Configurable Retention**: Nodes can configure historical data retention based on storage capacity, with archival nodes maintaining complete history.