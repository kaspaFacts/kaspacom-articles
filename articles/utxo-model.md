## **Introductory Level - Kaspa Uses UTXO - What is that?**

What is a UTXO and how does Kaspa use it?

A structure for keeping track of who can spend what Kaspa.

What does that mean?  This article is put together in a way that assumes no prior knowledge, so we start with the Account Model and the UTXO model, then what a UTXO is, what it contains, and how it is spent.  Then how it applies to both Bitcoin and Kaspa.

**Account Model -** The Account Model behaves like a traditional bank account by maintaining a balance and providing familiar operations.  Just like checking your bank balance, you can query the account's current holdings, and similar to how banks track your transaction history, the Account manages your financial state.  The system provides standard account operations such as receiving deposits and making transfers, with each account having its own unique identifier and name for easy management.  Multiple account types are available to suit different needs, much like how banks offer various account types for different purposes.

![account.webp](https://cdn.buttercms.com/J0PKCuTGGwKVGPNBRAAa)
**UTXO Model -** The UTXO Model behaves like physical cash or coins in your wallet, where each coin has a specific value and can only be spent once.  Just as you might have several bills and coins of different denominations in your physical wallet, a digital wallet contains multiple UTXOs of varying amounts that represent your spendable balance.  When you make a transaction, specific UTXOs are consumed as inputs (like spending exact bills), and new UTXOs are created as outputs for the recipient and any change back to you, similar to how a cashier gives you change when you pay with a larger bill.  The system tracks these individual "coins" across all transactions, maintaining a complete record of which UTXOs exist and can be spent, much like how physical money moves from person to person while maintaining its individual identity.

![undefined](https://cdn.buttercms.com/eXfeIzXmR9iC2L6hdB1S)

**UTXO Structure -** A UTXO (Unspent Transaction Output) is structured like a digital receipt that contains all the essential information needed to spend it, similar to how a check contains the amount, recipient details, and authorization information.  Each UTXO contains the amount of value it holds and defines the spending conditions.  Just as a physical coin has its denomination stamped on it and can be verified as authentic, each UTXO carries its value and cryptographic proof of ownership, making it a self-contained unit of value that can be independently verified and spent.  The system treats each UTXO as a discrete object with its own unique identifier, allowing precise tracking of individual units of value as they move through the network.

![undefined](https://cdn.buttercms.com/pDnIfnjvS02Q2wfukUbS)

**Spending a UTXO -** Spending a UTXO behaves like using physical cash, where you must present the exact bill or coin to make a purchase, and once spent, it cannot be used again.  The process begins by locating the specific UTXO you want to spend and verifying it exists in the UTXO set, similar to checking that a bill in your wallet is genuine and unspent.  When creating a transaction, you reference the UTXO by its unique identifier and provide a signature script that proves you have the right to spend it.  The system validates that the UTXO hasn't already been spent (preventing double-spending), checks that you meet the spending conditions, and then removes the UTXO from the spendable set while creating new UTXOs as outputs, completing the transfer of value from one party to another.

![undefined](https://cdn.buttercms.com/Nwzvhfn5SdWUK2coOfS5)

## **Simplified Definitions**

**Account Model -** A system that behaves like a bank account, maintaining a balance and providing familiar operations for managing funds.

****
**UTXO Model -** A system that behaves like physical coins in your wallet, where each coin (UTXO) has a specific value and can only be spent once.


**UTXO Structure -** A data structure containing the amount of value and spending conditions.

**Spending a UTXO -** The process of referencing a specific UTXO, providing cryptographic proof of ownership(and any other spending conditions), and consuming it as input to create new UTXOs as outputs.

A UTXO is just a structure for tracking who can spend what.

## **Bitcoin and Kaspa**

**Bitcoin -** Uses the UTXO model.  Transactions are collections of UTXOs being consumed and created.  Transactions are stored in the body of each block within the Chain.

![undefined](https://cdn.buttercms.com/cqL2o0MSQkm4A01vRnej)

**Kaspa -** Uses the UTXO Model.  Transactions are collections of UTXOs being consumed and created.  Transactions are stored in the body of each block within the [DAG](https://www.kaspa.com/learn-kaspa/post/dag-and-kaspa).

![utxo in block.webp](https://cdn.buttercms.com/cqL2o0MSQkm4A01vRnej)