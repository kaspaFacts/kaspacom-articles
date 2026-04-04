## **Introductory Level - Kaspa Uses MuHash - What is that?**

What is MuHash and how does Kaspa use it?

A structure for keeping track of UTXOs and pruning old block body data.

What does that mean?  This article is put together in a way that assumes no prior knowledge, so we start with MuHash.  What a MuHash structure is, how it is calculated, and how it retains the properties of <span style="text-decoration: underline;">MU</span>ltiplication.  Then how it applies to both Bitcoin and Kaspa.

**MuHash -**  In cryptographic systems, MuHash (Multiplicative Hash) is a specialized hashing algorithm designed to efficiently compute a single hash value from a set of elements, such as transactions in a blockchain.  It allows for incremental updates, meaning elements can be added or removed without recomputing the entire hash, which enhances performance in dynamic datasets.  MuHash ensures collision resistance, making it highly secure for verifying data integrity.  This makes it particularly useful in applications like Kaspa, where it helps validate large sets of transactions quickly.  Unlike traditional hashing, MuHash supports efficient set operations, reducing computational overhead.

![undefined](https://cdn.buttercms.com/pgVe1IneSBGr1xra0w7R)
**Numerator -** A running tally that keeps track of all elements that have been added to a mathematical set.  Every time an element is added, it gets converted into a special mathematical representation and then multiplied into this running tally, with all operations performed modulo a prime number.  The numerator starts at the number 1 and grows larger as more elements are added to the system.  What makes this special is that it doesn't matter what order the elements are added - you'll always get the same final number due to the commutative property of multiplication in the finite field.  This allows multiple processors to work on different parts of the calculation simultaneously and still arrive at the correct answer.

![add_element.webp](https://cdn.buttercms.com/z9U0vhIStClzVOzqgkjp)

**Denominator -** A separate running tally that keeps track of all elements that have been removed from the mathematical set.  When an element is removed, it gets converted into the same special mathematical form and multiplied into this denominator tally, again with all arithmetic performed modulo the same prime.  Like the numerator, it starts at 1 and grows as elements are removed.  The clever part is that when you want to know the current state of the set, you simply divide the numerator (elements added) by the denominator (elements removed), with division implemented as multiplication by the modular inverse in the finite field.  This mathematical approach allows the system to efficiently track millions of element operations without storing every individual element, while ensuring the final result is always accurate regardless of operation order.

![remove_element.webp](https://cdn.buttercms.com/1r1RnlfDQCq8TsOj80X0)

**Modular Prime Constraint** - The modular prime acts as a mathematical boundary that keeps both the numerator and denominator within a manageable range during all arithmetic operations.  Every multiplication is performed modulo prime number, which means that no matter how many elements are added or removed, the results always "wrap around" to stay within the finite field.  This constraint prevents the numbers from growing infinitely large and ensures that all computations remain mathematically well-defined and computationally feasible.  Without this modular arithmetic constraint, the multiplicative operations would eventually produce numbers too large to handle efficiently.  A clock is an example of a finite field, if it is now 1, 14 hours from now is 3.

### ![undefined](https://cdn.buttercms.com/Uv4WpPFMTQqYKD912Zn4)

**Modular Inverse -** The modular inverse is the mathematical operation that makes division possible in the finite field used by MuHash .  When you need to "divide" the numerator by the denominator to get the final hash result, you're actually multiplying the numerator by the modular inverse of the denominator.

## **Simplified Definitions**

**MuHash -** A structure for hashing elements in a set, quickly, where the order of adding or removing elements does not matter, and does not have to be recalculated from scratch each time there is a change.

****
**Numerator -** The first field, where elements are multiplied when added.


**Denominator -** The second field, where elements are multiplied when removed.

**Modular Prime Constraint -** A prime number that defines the mathematical field where all operations occur, ensuring the numerator and denominator remain computationally manageable while preserving mathematical correctness.

**Modular Inverse - **The result of a calculation that allows "division" in a finite field.

MuHash is just a structure, consisting of a numerator and a denominator, that allows fast hashing of elements of a set in any order without recalculating from scratch.

## **Bitcoin and Kaspa**

**Bitcoin -** Full nodes keep every transaction, including old spent transactions.  The ability to prune old spent transactions without breaking the header(and by extension the PoW chain) was described in section 7 of the Bitcoin paper by Satoshi Nakamoto.  No pruning has been implemented.  Bitcoin Full nodes store every block in full (all transactions, unspent or spent) back to Genesis.

![undefined](https://cdn.buttercms.com/FErgQ6EIQgyRfxUX0dK4)

**Kaspa -** Full nodes prune old data.  This ability to prune old data requires a way to remove all transaction data from each block AND cryptographically secure them to each header.  Kaspa uses MuHash for removing transaction data from blocks (so that only headers in the [DAG](https://www.kaspa.com/learn-kaspa/post/dag-and-kaspa) remain past the pruning point) and securing them to each header.  This is an essential step for pruning (Kaspa also prunes headers, but that is for another article).  Kaspa separates Transaction Data(UTXOs) from Consensus Data(Headers), resulting in Kaspa only storing Unspent Transactions, instead of all transactions ever made.  This reduces storage requirements compared to Bitcoin.

![undefined](https://cdn.buttercms.com/3qZId5Y0REyL9SALjRsH)