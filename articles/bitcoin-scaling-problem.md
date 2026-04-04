## **Introductory Level - Kaspa solved the "Bitcoin Scaling Problem" - What is that?**

What is the Bitcoin Scaling Problem and how did Kaspa solve it?

By inclusion.

What does that mean?  This article is put together in a way that assumes no prior knowledge, so we start with a Client-Server Model, then the Peer to Peer network.  What a P2P network is, how it looks, and how messages propagate through it.  Then how it applies to both Bitcoin and Kaspa.

**Client-Server Model -** In a client-server network, a centralized architecture organizes communication and resource sharing through a single, powerful computer called a server, which connects to multiple user devices known as clients. Each client sends requests to the server, which processes and responds with the requested data or services, maintaining control over the network's operations. This structure ensures efficient management and streamlined data access but relies heavily on the server, making it vulnerable to failures or overload. Client-server networks form the backbone of systems like websites or online databases, where a central point coordinates all interactions.  In this example, three clients are connected to a server.

**![undefined](https://cdn.buttercms.com/pfokWzfSjOzLvpCffjd8)**
**Peer to Peer Network -** A decentralized architecture that enables direct communication and resource sharing among interconnected nodes, often referred to as peers.  Each peer operates as both a client and a server, contributing to the network's resilience and scalability.  This structure facilitates efficient data exchange and coordination without reliance on a central authority, ensuring robustness against failures and censorship.  P2P networks extend the principles of distributed systems, optimized for collaborative and trustless environments.  In this example, four peers make up the network, and each peer connects to two others.

![undefined](https://cdn.buttercms.com/GGF1vefeTiKRo4H2U3Sj)

**Node -** In a peer-to-peer (P2P) network, a node is an individual computing device, such as a computer, server, or IoT device, that actively participates in the network's operations.  Each node maintains a copy of shared data or protocols, using cryptographic techniques like digital signatures or key exchange to authenticate interactions with other peers.  These nodes are the fundamental units that enable the P2P network to distribute, validate, and synchronize data, ensuring resilience and decentralization.  Any disruption or misbehavior in a node can be isolated by the network, preserving trustless data exchange and system integrity in distributed environments like file-sharing or blockchain systems.  Here a node has been highlighted.

![peer node.webp](https://cdn.buttercms.com/AwGSsH9lS0uv3l2WMv6e)

**Connections - **In a peer-to-peer (P2P) network, a connection is a direct way for two computers, called nodes, to share information or work together.  Each connection is protected with special security methods to make sure the computers are trustworthy and the information stays safe.  These connections link many computers together, forming a network that keeps working even if one connection has a problem.  If something goes wrong with a connection, the network finds another way to keep the computers communicating, ensuring smooth and secure sharing in systems like those used for sharing files or managing digital records.  Here the connections of the previous node are highlighted.

![connections.webp](https://cdn.buttercms.com/IAU4R6N2Q8G2g1I9eEUC)

**Time to Propagate -** In a peer-to-peer (P2P) network, time to propagate is the duration it takes for data, such as a file or message, to travel from one computer, or node, to others across the network.  Each node shares the data directly with its connected peers, which then pass it along to others, creating a ripple effect. This process is designed to be fast and reliable, even if some nodes are slow or offline, as the network finds alternative paths to keep data moving.  Time to propagate is crucial in systems like file sharing or digital currencies, ensuring information spreads quickly and efficiently across all nodes.  In this example, Node A has shared a message with its peer nodes, who will then share that message with their peer nodes, continuing until the entire network has received the message.  The time it takes to send this message from one node to the rest of the nodes in the network is the Time to Propagate.  Note, in this example, Node B will be the last to receive the message from Node A.  Until Node B receives the message from one of its peers, it is unaware of the message, Node A and Node B have different points of view during this time.

![undefined](https://cdn.buttercms.com/2Y1JZ3PSNKkyu3m2jKAs)

## **Simplified Definitions**

**Client-Server Model -** A central computer manages data, processes requests, and handles connections for multiple clients, where the server controls the flow of data and services and acts as the main hub for all activity.

****
**Peer to Peer -** Multiple computers share data and tasks directly with each other without a central computer controlling anything.


**Node -** A single computer that directly shares data and tasks with other computers in the network.

**Connection -** A connection is a direct link between two computers, or nodes, that allows them to share data or work together without a central computer in charge.

**Time to Propagate -** The time it takes for data to spread from one computer to others across a peer-to-peer network, where nodes share information directly without a central computer managing the process.

A Peer-to-Peer Network is just a structure, consisting of Nodes with Connections, that ensures data flows without a central authority.

## **Bitcoin and Kaspa**

**Bitcoin -** a Peer to Peer Network allows information to flow between Nodes connected to each other.  This structure enables a trustless setup that is censorship resistant.

![peer to peer.webp](https://cdn.buttercms.com/GGF1vefeTiKRo4H2U3Sj)

**Kaspa -** a Peer to Peer Network allows information to flow between Nodes connected to each other.  This structure enables a trustless setup that is censorship resistant.

![peer to peer.webp](https://cdn.buttercms.com/GGF1vefeTiKRo4H2U3Sj)

So what's the difference if both Bitcoin and Kaspa use a P2P network?  Bitcoin allows blocks to point to only one previous block, if a parallel block is created while another block is propagating through the network, it creates a fork, the network will orphan all but one block.  Kaspa allows blocks to point to multiple previous blocks, this allows nodes to create parallel blocks while having a different point of view.  All parallel blocks will be included.  Using the example from above to illustrate the Bitcoin network, Block A found a block, and is now sharing that block with its peers, until Node B is aware of this new block, Node B continues to mine on the heaviest block it is aware of, the old block, the result is wasted work.  If Node B were to find a new block before it receives information about the block propagating through the network, it would share this new block with its peers as well, creating a fork in the chain, nodes would have to decide which block to mine on since one will be orphaned.  If the same exact network from this example was Kaspa, and Node B finds a block during the time the block found by Node A is propagating, both blocks would be included in the [DAG](https://www.kaspa.com/learn-kaspa/post/dag-and-kaspa), the result is less wasted work and the ability to create blocks faster than the time to propagate.

<img src="https://cdn.buttercms.com/2Y1JZ3PSNKkyu3m2jKAs" alt="time to prop.webp">

**Bitcoin Scaling Problem -** Bitcoin security depends on the block creation rate (10 minutes) and the time to propagate (95% after as much as 40 seconds) being very far apart.  If the block creation rate is too fast compared to the time to propagate, the result will be many orphan blocks.  Orphan blocks are wasted work and wasted work compromises security.

![undefined](https://cdn.buttercms.com/qiUhW8JRnGjEV02RcEp0)

**Kaspa Solved The Bitcoin Scaling Problem -** Kaspa uses an inclusive protocol that allows blocks to point to multiple previous blocks, the result is that the block creation rate can be faster than the time to propagate and maintain security.

<img src="https://cdn.buttercms.com/P31XzHa9RySKXmCLTazx" alt="kaspa DAG.webp">