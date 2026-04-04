## What is KIP-20?

KIP-20 introduces covenant identifiers (unique tags for covenant groups), which are 32-byte network-verified identifiers carried by UTXOs to establish a stable covenant chain (sequence of related covenant transactions) across coin movements. The mechanism provides native covenant chain tracking and tamper-proof protection at the consensus level (network-wide agreement rules), eliminating the need for previous transaction proofs in covenant validation.

**Note:** KIP-20 is currently in proposal stage with implementation pending network activation.

## The Challenge: Covenant Implementation Limitations

KIP-17 enables covenants via transaction introspection (scripts that can read transaction data), but establishing continuous application identity requires recursive proofs and parent transaction witnesses. This creates significant overhead in transaction size and forces covenant logic to depend on standard transaction format within the script processing system.

**Witness Size Overhead**: Covenant validation requires carrying parent and often grandparent transactions as witness data (proof information included with transactions), increasing transaction size and network bandwidth requirements.

**Encoding Dependencies**: Covenant logic must depend on standard transaction format and hashing within the script processing system, creating implementation complexity.

**State Forgery Risk**: Parent/grandparent-witness schemes can only ensure forged covenant UTXOs are unspendable, not that they are uncreatable in the first place.

KIP-20 solves these problems by introducing a minimal consensus-enforced mechanism that provides native covenant chain tracking, prevents state forgery at the consensus level, and eliminates the need for previous transaction proofs.

## How KIP-20 Works

### Data Model Extensions

KIP-20 extends core transaction and UTXO structures with covenant-related fields while maintaining backward compatibility for non-covenant transactions.

- **TransactionOutput.covenant**: Optional CovenantBinding containing authorizing_input (input number) and covenant_id (32-byte hash)
- **UtxoEntry.covenant_id**: Optional 32-byte hash identifier carried by the UTXO
- **Transaction versioning**: Covenant bindings only supported in transactions with version >= 1

The covenant binding declares which input authorizes the output and which covenant instance it belongs to, while the UTXO entry carries the actual covenant identifier for consensus tracking.

### Consensus Validation Rules

KIP-20 introduces two fundamental rules for covenant creation at the consensus level:

**Continuation Rule**: An output can carry a covenant_id if its authorizing input spends a UTXO with the same covenant_id. This enables covenant chains to continue across transactions.

**Genesis Rule**: New covenant instances can be created through genesis initialization, where the covenant_id is computed as a secure hash of the authorizing outpoint and approved outputs.

### Genesis Hashing Algorithm

Genesis covenant IDs are computed using a secure hash function of the authorizing outpoint and approved outputs. The approved outputs are listed in order by output index, ensuring deterministic covenant creation.

### Script Introspection Opcodes

KIP-20 introduces new opcodes for covenant-related reasoning within transactions:

- **OpInputCovenantId**: Get covenant identifier of a UTXO input
- **OpAuthOutputCount**: Count approved outputs for an input
- **OpAuthOutputIdx**: Get index of k-th approved output
- **OpCovInputCount**: Count inputs with specific covenant ID
- **OpCovInputIdx**: Get index of k-th input with covenant ID
- **OpCovOutCount**: Count outputs with specific covenant ID
- **OpCovOutputIdx**: Get index of k-th output with covenant ID

These opcodes enable efficient covenant validation without requiring parent/grandparent transaction data as witnesses.

## How the Network Uses This

### Singleton Covenant Pattern

Supports simple state machines where each covenant has exactly one active UTXO. The covenant script validates that the spending transaction creates exactly one continuation output with the same covenant_id, ensuring linear state progression.

For example, a payment channel could use this pattern where each state update spends the current channel UTXO and creates a new one with updated parameters.

### Covenant Transition Pattern

Enables safe exit/enter patterns between covenants within single transactions. A transaction can simultaneously exit one covenant (spending its UTXO without continuation) while entering another (creating genesis outputs for the new covenant).

This atomic behavior ensures that either both operations succeed or neither does, preventing partial state transitions that could leave funds in inconsistent states.

### Canonical Bridge Infrastructure

KIP-20 provides foundational infrastructure for L1/L2 canonical bridges, though complete implementation requires additional components:

**Entry Operations**: Users can initiate L2 entry by sending KAS to covenant addresses that use covenant scripts to verify proper authorization and route funds to the L2 state commitment.

**Exit Operations**: L2 state transitions can include exit operations that transfer funds back to L1 addresses, with validity enforced through zero-knowledge proofs using KIP-16's verification capabilities and covenant validation.

**Additional Requirements**: Complete bridge implementation requires KIP-16's ZK proof verification capabilities, origin verification patterns, and layer connection linking beyond KIP-20's scope.

### Many-to-Many Leader-Follower Pattern

Supports complex state transitions where multiple covenant inputs and outputs participate, using a designated leader input for validation coordination.

For example, native asset protocols can use this pattern where multiple UTXOs representing different token holders participate in a single all-or-nothing transfer, with one input designated as the leader responsible for validating the complete transition.

## Why This Matters

### Reduced Transaction Size

Eliminates the need to carry parent and grandparent transactions as witness data, reducing transaction size and lowering network bandwidth and storage requirements.

### Enhanced Security Model

Shifts covenant security from script-level validation (program-based checking) to consensus-level enforcement (network rule enforcement), making forged covenant UTXOs un-creatable rather than just un-spendable, providing stronger security guarantees.

### Simplified Covenant Development

Removes the burden of standard format and hashing logic from covenant scripts, allowing developers to focus on application logic rather than low-level transaction structure validation.

## Key Takeaways

- **Native Chain Tracking** - KIP-20 provides consensus-level covenant chain tracking through 32-byte identifiers carried by UTXOs
- **Tamper-Proof Guarantee** - Covenant UTXOs cannot be created unless they follow strict continuation or genesis rules
- **Witness Elimination** - Removes the need for parent/grandparent transaction proofs in covenant validation
- **All-or-Nothing Covenant Transitions** - Enables safe exit/enter patterns between covenants within single transactions
- **Bridge Infrastructure** - Provides foundational primitives for L1/L2 canonical bridges and advanced applications