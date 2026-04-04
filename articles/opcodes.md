## What are OpCodes?

OpCodes are single-byte numerical codes that represent specific operations in Kaspa's transaction script engine. Each OpCode corresponds to a fundamental instruction that nodes execute during transaction validation. Every transaction on the [BlockDAG](https://kaspa.com/learn-kaspa/post/dag-and-kaspa) - from simple signature checks to complex conditional logic - uses OpCodes to define spending rules and enable transaction validation.

## Simple Example: Basic Signature Verification

The simplest spending condition is a signature check. This script requires anyone spending the funds to provide a valid signature from a specific public key:

<code>// Script:  OP_CHECKSIG
// Requires a valid signature from the specified public key
//
let script = ScriptBuilder::new()
.add_data(&public_key)?       // Push 32-byte public key
.add_op(OpCheckSig)?          // Verify signature
.drain();</code></pre>

This demonstrates the core concept: OpCodes define the rules, and transactions provide the data to satisfy those rules.

### How It Works

During validation, the spending transaction provides a signature that gets verified against the public key in the script:

<code>// Human readable translation:
//
verify(signature, public_key)
// Result: true = allow spending, false = reject transaction
</code></pre>
### Stack Visualization

The script operates on a stack where data is pushed and popped:

- **Step 1**: Transaction provides signature → Stack: [signature]
- **Step 2**: Script pushes public_key → Stack: [signature, public_key]
- **Step 3**: OP_CHECKSIG verifies → Stack: [true] if valid, [false] if invalid
- **Step 4**: Final stack state determines if spending is allowed

This simple Pay-to-Public-Key (P2PK) pattern is how ALL spending works on Kaspa. Complex spending conditions are built by combining these basic OpCodes in sophisticated ways, but every transaction follows this fundamental pattern of defining rules with OpCodes and satisfying them with transaction data.

## The Challenge: Enforcing Transaction Rules

Early blockchain systems had limited mechanisms for controlling how funds could be spent. Kaspa needed a mechanism to enforce complex rules while maintaining determinism across all network nodes.

In a decentralized system, every node must validate transactions identically. Without a standardized way to define spending conditions, the network cannot achieve consensus on which transactions are valid.

**Static Spending Rules**: Without OpCodes, transactions were limited to fixed validation patterns. Most systems could only enforce basic "anyone can spend" or single signature requirements, with no way to express more complex spending conditions.

**No Conditional Logic**: Early systems lacked branching capabilities. Scripts couldn't make decisions based on time, multiple signatures, or other conditions, preventing time-locked payments and multi-signature schemes that require "M-of-N" validation.

**Limited Expressiveness**: Developers couldn't create custom validation logic. Without access to transaction data during validation, advanced patterns like covenants (scripts that control how funds can be spent in the future) were impossible to implement.

**No Transaction Introspection**: Scripts couldn't access information about the transaction itself, such as input amounts, output values, or other transaction structure, limiting the ability to create sophisticated spending rules.

OpCodes solve these problems by providing a deterministic, bounded scripting language that all nodes execute identically, enabling conditional transaction rules while maintaining deterministic consensus.

## How Opcodes Work

### Opcode Structure and Implementation

Each OpCode is defined with a unique byte value, length specification, and execution logic. The OpCode system uses a macro-based approach to generate implementations.

- **Byte Value**: Unique identifier (0x00 to 0xFF) that represents the operation
- **Length Specification**: Defines the size of data associated with the OpCode
- **Execution Logic**: The actual operation performed when the OpCode is executed

The OpCode system implements three core traits: OpCodeMetadata for properties, OpCodeExecution for running operations, and OpcodeSerialization for data handling.

### Stack-Based Execution

OpCodes operate in a stack-based virtual machine where data is pushed onto and popped from a stack during execution. This approach ensures deterministic execution across all nodes.

For example, data push OpCodes like OpData1 through OpData75 place specific amounts of data onto the stack for later operations.

### Conditional Execution Flow

OpCodes support conditional logic through control flow operations:

1. OpIf evaluates a condition from the stack and executes code based on the result
2. OpElse provides alternative execution paths
3. OpEndIf terminates conditional blocks
4. OpVerify fails the script if a condition is not met

## How the Network Uses This

### Transaction Validation

Nodes execute OpCode scripts when validating every transaction. Even the simplest signature verification uses OpCode scripts. The TxScriptEngine processes each input's signature script against the corresponding UTXO's script public key, ensuring all spending conditions - basic or complex - are properly validated.

Each transaction input must satisfy the spending conditions defined by the OpCode script, ensuring only authorized parties can spend the funds.

This validation happens in consensus, meaning all nodes agree on whether a transaction's script execution was successful.

### Multi-Signature Scripts

OpCodes enable M-of-N signature schemes where multiple parties must approve a transaction. The OpCheckMultiSig OpCode verifies multiple signatures against public keys.

Formula: M signatures required out of N total public keys

This allows for shared control of funds, requiring cooperation among multiple parties while providing flexibility in the number of required signatures.

### Transaction Introspection (KIP-10)

Kaspa extends OpCode functionality with introspection OpCodes that allow scripts to examine transaction data, enabling covenant-like behavior.

Examples include OpTxInputAmount (0xbe) to check input values and OpTxInputSpk (0xbf) to examine script public keys. These OpCodes enable covenant-like transaction restrictions that can enforce rules based on transaction structure.

## Why This Matters

### Programmable Money

OpCodes provide Kaspa with programmable transaction validation capabilities. Developers can create multi-signature wallets, time-locked payments, and transaction introspection, and conditional spending patterns without modifying the core protocol.

### Extensibility

The OpCode system is designed for extension. New OpCodes can be added through protocol upgrades (like KIP-10) to add new functionality while maintaining backward compatibility.

### Security and Determinism

OpCodes include security measures such as disabled operations and execution limits. Disabled OpCodes prevent dangerous operations, and execution limits prevent resource exhaustion attacks.

## Conditional Script: Time-Locked Transaction

Here's a simple script that requires a signature or a time lock:

<code>// Script: OP_IF locktime OP_CHECKLOCKTIMEVERIFY OP_DROP OP_ELSE signature OP_CHECKSIG OP_ENDIF
// This script requires either a valid signature OR the specified locktime to have passed
//
let script = ScriptBuilder::new()
.add_op(OpIf)?                    // Start conditional
.add_lock_time(1640995200)?       // Push locktime (Jan 1, 2022)
.add_op(OpCheckLockTimeVerify)?   // Verify locktime has passed
.add_op(OpDrop)?                  // Remove locktime from stack
.add_op(OpElse)?                  // Else branch
.add_data(&signature)?            // Push signature
.add_data(&public_key)?           // Push public key
.add_op(OpCheckSig)?              // Verify signature
.add_op(OpEndIf)?                 // End conditional
.drain();
</code></pre>

This script demonstrates how OpCodes combine to create complex spending conditions - in this case, a transaction that can be spent either with a signature or after a specific time.

<code>// Human readable translation:
//
IF transaction.lock_time >= 1640995200 (Jan 1, 2022) THEN
allow spending
ELSE
verify(signature, public_key)
ENDIF
</code></pre>
## Security Warning: Conditional Script Above Allows Anyone to Spend After Timelock

**Important Security Note**: The sample time-locked script shown above contains a critical security vulnerability. Once the timelock expires (after January 1, 2022), **anyone can spend the UTXO** without any authorization.

### How the Vulnerability Works

The current script structure:

<code>.add_op(OpIf)?                    // Start conditional
.add_lock_time(1640995200)?        // Push timestamp
.add_op(OpCheckLockTimeVerify)?    // Verify time condition
.add_op(OpDrop)?                   // Clean up stack
// IF branch ends here - no spending conditions!
.add_op(OpElse)?                   // Else branch
.add_data(&signature)?             // Push signature
.add_data(&public_key)?            // Push public key
.add_op(OpCheckSig)?               // Verify signature
.add_op(OpEndIf)?                  // End conditional
</code></pre>

The <code>OpCheckLockTimeVerify</code> OpCode only verifies that enough time has passed. When it succeeds, the script continues execution. Since there are no additional OpCodes in the IF branch to verify authorization, the script succeeds and allows anyone to spend the funds.

### The Security Implication

This creates a dangerous scenario where:

- **Before January 1, 2022**: Only someone with the valid signature can spend
- **After January 1, 2022**: Anyone can spend the [UTXO](https://kaspa.com/learn-kaspa/post/kaspa-utxo-model) without any signature or authorization

## Key Takeaways

- **Fundamental Instructions** - OpCodes are the basic building blocks that enable Programmable Transactions on Kaspa
- **Stack-Based Execution** - All OpCode operations work through a deterministic stack-based virtual machine
- **Conditional Logic** - Control flow OpCodes enable complex decision-making within transaction scripts
- **Extensible Design** - New OpCodes can be added through protocol upgrades to enhance functionality
- **Smart Contract Foundation** - OpCodes provide the foundation for advanced financial applications and covenant-like behavior
