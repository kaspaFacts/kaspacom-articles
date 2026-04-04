# Kaspa Consensus System: Complete Learning Guide

This guide provides comprehensive coverage of Kaspa's GHOSTDAG Consensus System, organized by topic area. Each section groups related concepts together, allowing you to explore specific aspects of the system in depth.

## How to Use This Guide

Browse the sections below to find topics that interest you. Articles are organized by conceptual category rather than difficulty level, so you can jump directly to any area you want to understand. Some concepts appear in multiple sections because they're relevant to different aspects of the system.

## Foundational Concepts

Understanding the basic building blocks of Kaspa's architecture.

1. [**DAG - Directed Acyclic Graph Conceptual Overview**](https://kaspa.com/learn-kaspa/post/dag-and-kaspa) - What DAGs are and why Kaspa uses them instead of linear chains
2. [**DAG Terminology**](https://kaspa.com/learn-kaspa/post/dag-terminology) - Essential terms: Anticone, Past, Future, Tips, and their precise meanings in Kaspa's context
3. [**GHOSTDAG Overview**](https://kaspa.com/learn-kaspa/post/ghostdag-simplified) - High-level explanation of how Kaspa orders blocks and achieves consensus
4. [**K-Cluster**](https://kaspa.com/learn-kaspa/post/ghostdag-k-cluster) - What constitutes a K-cluster violation and why it's the Security foundation
5. [**BPS Configuration**](https://kaspa.com/learn-kaspa/post/blocks-per-second) - Blocks per Second: throughput targets and parameter derivation
6. [**Anatomy of a Block**](https://kaspa.com/learn-kaspa/post/anatomy-of-a-block) - What exactly is a Block

## Core Data Structures

The fundamental data types that represent blockchain state.

7. [**UTXO - What is that?**](https://kaspa.com/learn-kaspa/post/kaspa-utxo-model) - Unspent Transaction Outputs and how Kaspa tracks spendable coins
8. [**MuHash - What is that?**](https://kaspa.com/learn-kaspa/post/kaspa-uses-muhash) - Cryptographic commitment for UTXO Set Verification and Pruning
9. [**Merkle Root and Accepted Merkle Root**](https://kaspa.com/learn-kaspa/post/kaspa-linking-the-body-to-the-header) - Transaction Commitments, linking the Body to the Header
10. [**Parents and Mergeset**](https://kaspa.com/learn-kaspa/post/parents-vs-mergeset) - The relationship between a block's direct Parents and its full Mergeset

## GHOSTDAG Mechanics

Deep dive into the consensus algorithm implementation.

11. [**K Parameter**](https://kaspa.com/learn-kaspa/post/k-parameter) - The Security parameter controlling K-cluster violations
12. [**Parent Selection and Ordering**](https://kaspa.com/learn-kaspa/post/parent-chain) - How the Selected Parent is chosen, the first step****
13. [**Mergeset Creation**](https://kaspa.com/learn-kaspa/post/kaspa-mergeset-creation) - How blocks build their Mergeset from the DAG, the second step
14. [**Blue and Red Classification**](https://kaspa.com/learn-kaspa/post/kaspa-blocks-blue-vs-red) - The coloring Algorithm, the third step
15. [**Blue Work Calculation**](https://kaspa.com/learn-kaspa/post/blue-work-ordering-pt4) - Cumulative metrics for chain selection, the last step

## Consensus Parameters

Configuration values that govern network behavior.

16. [**BPS Configuration**](https://kaspa.com/learn-kaspa/post/blocks-per-second) - Mathematical relationships between BPS and derived parameters
17. [**K Parameter**](https://kaspa.com/learn-kaspa/post/k-parameter) - Security parameter and its calculation from BPS
18. [**Mergeset Size Limit**](https://kaspa.com/learn-kaspa/post/mergeset-size-limit) - Maximum Mergeset size constraints
19. [**Merge Depth Bound**](https://kaspa.com/learn-kaspa/post/merge-depth-bound) - Preventing deep reorgs through Depth Constraints
20. [**Finality Depth**](https://kaspa.com/learn-kaspa/post/finality-depth) - When blocks become irreversible
21. [**DAA Window**](https://kaspa.com/learn-kaspa/post/daa-window) - Sliding window for difficulty adjustment
22. [**Pruning Depth**](https://kaspa.com/learn-kaspa/post/pruning-depth) - How far back full block data is required
23. **Crescendo Fork Activation** *(Coming soon)* - How parameters change dynamically at fork activation

## Block Processing

How blocks flow through the validation pipeline.

24. [**Parents and Mergeset**](https://kaspa.com/learn-kaspa/post/parents-vs-mergeset) - Block Parent Selection and Mergeset construction
25. [**Mergeset Creation**](https://kaspa.com/learn-kaspa/post/kaspa-mergeset-creation) - Detailed Mergeset algorithm
26. [**Blue and Red Classification**](https://kaspa.com/learn-kaspa/post/kaspa-blocks-blue-vs-red) - Classification during Block processing
27. [**Parent Selection and Ordering**](https://kaspa.com/learn-kaspa/post/parent-chain) - Selected Parent determination
28. [**Blue Score and Blue Work**](https://kaspa.com/learn-kaspa/post/blue-score-and-blue-work) - Score calculation during processing
29. **[Block Processing Pipeline](https://kaspa.com/learn-kaspa/post/block-processing) **- Four-stage architecture: Header → Body → Virtual → Pruning

## Difficulty Adjustment (DAA)

How Kaspa maintains consistent block times.

30. [**DAA Window**](https://kaspa.com/learn-kaspa/post/daa-window) - Window construction for difficulty calculation
31. [**DAA Score**](https://kaspa.com/learn-kaspa/post/daa-score) - Block-based timing metric for difficulty adjustment
32. **Non-DAA Blocks** *(Coming soon)* - Blocks excluded from difficulty windows

## Anticone Finalization & Safe Pruning

Mathematical foundations for safe data removal.

33. [**DAG Terminology**](https://kaspa.com/learn-kaspa/post/dag-terminology) - Anticone definition and related concepts
34. [**Finality Depth**](https://kaspa.com/learn-kaspa/post/finality-depth) - Finality Guarantees and Depth Calculations
35. [**Merge Depth Bound**](https://kaspa.com/learn-kaspa/post/merge-depth-bound) - Depth constraints for Pruning Safety
36. [**K Parameter**](https://kaspa.com/learn-kaspa/post/k-parameter) - K's role in Finalization
37. [**Mergeset Size Limit**](https://kaspa.com/learn-kaspa/post/mergeset-size-limit) - Size constraints for Finalization
38. [**Pruning Depth**](https://kaspa.com/learn-kaspa/post/pruning-depth) - Safe Pruning Depth
39. [**Anticone Finalization Depth**](https://kaspa.com/learn-kaspa/post/anticone-finalization-depth) - Mathematical foundation for safe pruning depth

## Pruning System

Managing blockchain data growth.

40. [**MuHash - What is that?**](https://kaspa.com/learn-kaspa/post/kaspa-uses-muhash) - MuHash's role in Pruning
41. [**Anticone Finalization & Safe Pruning**](https://kaspa.com/learn-kaspa/post/anticone-finalization-depth) - Complete finalization theory
42. [**First Order Pruning**](https://kaspa.com/learn-kaspa/post/first-order-pruning-in-kaspa) - Removing old Block Bodies and Transactions
43. [**Second Order Pruning**](https://kaspa.com/learn-kaspa/post/second-order-pruning) - Removing old Headers while preserving Proof of Work
44. [**Archival vs Pruning Nodes**](https://kaspa.com/learn-kaspa/post/archival-nodes-vs-full-nodes) - Different node types and Storage Requirements

## Transaction Processing

How transactions are validated and included in blocks.

45. [**UTXO - What is that?**](https://kaspa.com/learn-kaspa/post/kaspa-utxo-model) - UTXO model in Transaction context
46. [**Transaction Selection (Intro)**](https://kaspa.com/learn-kaspa/post/intro-transaction-selection) - Basic Transaction Selection for mining
47. [**Transaction Selection Strategies (Intermediate)**](https://kaspa.com/learn-kaspa/post/transaction-selection) - Advanced Transaction Selection
48. [**DAA Window**](https://kaspa.com/learn-kaspa/post/daa-window) - DAA window's role in Transaction Validation
49. [**Transaction Validation**](https://kaspa.com/learn-kaspa/post/tx-validation) - Three-Phase Validation: Isolation, Header Context, and State Checks
50. [**Mass Calculation**](https://kaspa.com/learn-kaspa/post/transaction-mass) - Compute mass, storage mass, and transient mass
51. [**Coinbase Transactions**](https://kaspa.com/learn-kaspa/post/coinbase-transactions) - Special transaction that creates new coins and rewards miners

## Virtual State

Kaspa's unique approach to representing the current DAG tip.

52. [**Virtual Block**](https://kaspa.com/learn-kaspa/post/virtual-block) - The conceptual "current state" block
53. [**Sink Selection**](https://kaspa.com/learn-kaspa/post/sink-selection) - Finding the highest UTXO-valid block
54. [**Virtual Parents**](https://kaspa.com/learn-kaspa/post/virtual-parents) - Virtual Block parent selection
55. **Virtual DAA Score** *(Coming soon)* - DAA score of the virtual block

## Timestamps & Median Time

Timestamp consensus and validation.

56. [**Past Median Time**](https://kaspa.com/learn-kaspa/post/past-median-time) - Timestamp Validation Via Sampled Windows
57. [**Timestamp Validation**](https://kaspa.com/learn-kaspa/post/timestamp-validation) - Keeping time in a Decentralized environment

## Network & Scaling

Why Kaspa's architecture enables high throughput.

58. [**Bitcoin Scaling Problem**](https://kaspa.com/learn-kaspa/post/kaspa-and-the-bitcoin-scaling-problem) - Traditional blockchain limitations
59. [**BPS Configuration**](https://kaspa.com/learn-kaspa/post/blocks-per-second) - High throughput configuration
60. [**K Parameter**](https://kaspa.com/learn-kaspa/post/k-parameter) - K's role in network Security at Scale
61. [**Merge Depth Bound**](https://kaspa.com/learn-kaspa/post/merge-depth-bound) - Preventing reorgs in high-throughput networks

## Finality & Security

Understanding Kaspa's Security Guarantees.

62. [**K Parameter**](https://kaspa.com/learn-kaspa/post/k-parameter) - K's Security Properties
63. [**Merge Depth Bound**](https://kaspa.com/learn-kaspa/post/merge-depth-bound) - Depth-based Security
64. [**Mergeset Size Limit**](https://kaspa.com/learn-kaspa/post/mergeset-size-limit) - Size-based Security constraints
65. [**DAA Window**](https://kaspa.com/learn-kaspa/post/daa-window) - DAA's role in Security
66. [**Finality Depth**](https://kaspa.com/learn-kaspa/post/finality-depth) - Finality guarantees
67. [**Pruning Depth (Anticone Finalization Depth)**](https://kaspa.com/learn-kaspa/post/pruning-depth) - Pruning Safety guarantees
68. [**Probabilistic and Deterministic Finality**](https://kaspa.com/learn-kaspa/post/finality) - Two types of Finality in Kaspa

## KIPs

Kaspa Improvement Proposals.

69. [**KIP-0009**](https://kaspa.com/learn-kaspa/post/storage-mass-kip-0009) - KIP-9 Mitigating State Bloat
70. [**KIP-0020**](https://kaspa.com/learn-kaspa/post/kip-20) - KIP-20 Covenant ID for native covenant tracking
71. [**KIP-0021**](https://kaspa.com/learn-kaspa/post/kip-21) - KIP-21 Partitioned Sequencing Commitment

## Smart Contracts

Smart Contracts.

72. [**OpCodes**](https://kaspa.com/learn-kaspa/post/opcodes) - Enabling Covenants
73. [**Smart Contracts**](https://kaspa.com/learn-kaspa/post/smart-contracts) - Current, Proposed, and Future Smart Contracts

## DAGKNIGHT

DAGKNIGHT

74. [**DAGKNIGHT**](https://kaspa.com/learn-kaspa/post/dagknight) - What is the big deal?

## Notes

This categorical structure groups related concepts together, making it easier to find specific topics. Some articles appear in multiple categories (like K Parameter, BPS Configuration) because they're relevant to multiple aspects of the system - this is intentional and helps readers understand connections.