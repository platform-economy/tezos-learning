# Course Curriculum

## Module 1 - Blockchain Foundations

This module focuses on the basics of blockchain technology and compares leading blockchains and DLT platforms. This is a refresher intended to provide context for participants who know and a clear introduction for those who are new to this paradigm.

	What is blockchain technology?
	How do public blockchains work?
	Overview of popular consensus mechanisms
	Key cryptography concepts

## Module 2 - Getting to Know Tezos
                        
Tezos introduces new concepts that differ from traditional approaches to system architecture. We will learn Tezos vocabulary and get acquainted with the architecture. You will also run your first node and interact with it using the command line interface and RPC calls

	Key Tezos overview: Introduction; Consensus
	Tezos Alphanet: Alphanet.sh, Command Line Interface, JSON/RPC interface
	
---

### Node

A Tezos node is the local component of the system. It manages the context, which is the local knowledge of the Tezos blockchain state, and the connection to the gossip network and other nodes.

The gossip network is how Tezos nodes communicate to exchange blocks and operations with each other. The admin client, as well as the local endorser, baker, accuser and client connect to the Tezos network through the node. The admin client can monitor peer-to-peer (P2P) connections.

As well as listening for updates to the context, the node can send operations to peers when instructed to by the client or emit new blocks when instructed by the baker. Furthermore, the node can endorse blocks when instructed by the endorser or even report bad blocks when instructed by the accuser.

In Tezos, state changing operations propagate through the network via gossip between peers until a baker includes the operation in a block. An operation can propagate across several peers before it is included in a block. New blocks are currently produced about once per minute, but this could change in the future via the Tezos governance mechanism.

After initialisation, the node will sync up with the Tezos network. As the node processes blocks, it runs the operations in the blocks against the local copy of the context to create a new context. The latest block received is known as the head of the chain. Nodes advertise the latest chain head they have to other nodes in the network.

The node also gathers metadata about peers and multiple chains that may exist so it can select the best one based on its fitness. “Fitness” determines the quality of the chain leading up to that block.

### Client

The Tezos client is the main interface to the node. The client can read the context and inspect the state (get) and it can instruct the node to perform work such as broadcasting an operation to the network.

### Baker

In Tezos, validators are referred to as bakers. As the name suggests, the baker is responsible for baking (producing) new blocks. Bakers are connected to implicit accounts (more on implicit (tz1…) vs. originated (KT1…) accounts later) and compute baking rights on a per account basis based on a baker’s total stake (i.e. tokens participating in consensus). The baker is unique in that it needs direct access to the node data directory for performance reasons.

When the baker is selected to bake a block, it draws transactions from the mempool, which is the pool of operations that are known about (via gossip) but have not yet been included in a block.

### Endorser

Like the baker, the endorser is connected to an implicit account and computes endorsing rights on a per account basis based on a baker’s total stake. On receipt of new blocks it verifies the validity of the block. If the block is valid it will broadcast an endorsement operation.

## Accuser

The accuser is a daemon that monitors all blocks received. It looks for two indications of invalid blocks:

    When a baker has signed two blocks at the same block height (blocks at the same level).
    When an endorser injects more than one endorsement operation for the same baking slot.

Such irregularities trigger double-baking and double-endorsing operations that cause the offender to lose a portion of its stake (i.e. a security deposit).

### Inflation Funding and On-Chain Bounties

Tezos has a mechanism that can solve the misalignment of incentives around collective action, also known as the “free-rider problem”. As the Tezos Position Paper states, “the collective action problem arises when multiple parties would benefit from taking an action but none benefit from individually undertaking the action”. Most core developers of blockchains are compensated for their work by foundations, investors, or by token appreciation. What will happen to core development when foundations or investors no longer support developers, or the price of a token does not appreciate?

Tezos presents a solution to sustainably fund protocol development through inflation funding and on-chain bounties. As stated above, the Tezos governance mechanism allows stakeholders to upgrade the protocol through protocol amendments. When a stakeholder proposes a protocol amendment (upgrade), they can attach an invoice to the upgrade that mints new tokens if approved via the governance process. This allows protocol developers to be compensated directly by the protocols they build, not just by an external source that may not be in existence in the future. Remember, even with the blockchain mania of 2017, foundations and other blockchain entities still have a finite amount of resources that will be exhausted eventually!

Minting new tokens for the purpose of funding development is known as “inflation funding.” In addition to funding core protocol development, inflation funding can be used to raise awareness of projects, donate to charities, create on-chain bounties, and fund public goods that are built on top of the protocol (e.g. through a DAO, or “Decentralized Autonomous Organization”).

### OCaml, Michelson, SmartPy, LIGO and other languages

Tezos itself is implemented in OCaml. OCaml is a functional programming language used in mission-critical industries that require formal proofs of properties of programs. Michelson is a domain-specific language used to write smart contracts on Tezos. It is stack-based, strongly typed, and designed to facilitate formal verification so developers can more easily prove properties of their smart contracts.

There are several other Tezos smart contracts languages. SmartPy is an intuitive and effective smart contract language and development platform that will allow Python developers to write smart contracts on Tezos. LIGO is a statically typed high-level language that compiles down to Michelson. The syntaxes that are currently supported are PascaLIGO (pascal-like syntax) and CameLIGO (caml-like syntax). ReasonLIGO(reason-like syntax) will be supported soon. Morley/Lorentz is an library to write Michelson contracts in Haskell. Fi is another high-level language that compiles to Michelson and is similar to Javascript and Solidity. Additionally, developers can write Tezos smart contracts in ReasonML.

```
$ wget https://gitlab.com/tezos/tezos/raw/babylonnet/scripts/alphanet.sh

$ mv alphanet.sh babylonnet.sh

$ chmod +x babylonnet.sh

$ ./babylonnet.sh start
```

### Our first contract

Time to deploy our first Tezos contract!

Create a new file repeater.tz in the same folder as babylonnet.sh and paste:

```
parameter int;
storage int;
code { CAR ;
       NIL operation ;
       PAIR };
```
into it. Then save it.

This is a repeater contract, which will take an integer as a parameter and return it, in effect saving that number in the storage.

Remember that in Tezos, each contract takes as input one pair of a parameter and storage structure, and then returns, as output, one pair consisting of an operation list and another storage structure. It is stack-based.

Before any contract operation, the stack has been populated by the Tezos environment according to the transaction that spawned the contract. The stack has one level, populated with the input pair. Our code here has 3 instructions:

1) We pop this input pair from the top of the stack, we take the left-hand part of the input pair CAR, i.e. the input parameter, and push that back into the top of the stack. At this stage, the stack is still 1-level deep as the pair was consumed and replaced with the parameter;
    
2) We push an empty operation list, NIL operation, to the top of the stack. At this stage, the stack is 2-level deep;

3) We make a pair of both levels of the stack with the operation list on the left, PAIR. Each instruction is working on the stack, e.g. PAIR will take the parameter and operation list from the stack and push a pair. The stack is back to being 1-level deep.

At this stage, the Tezos environment takes the output storage structure, here an int, puts it in storage and executes the operations, if there are any.

### Links

https://developers.tezos.com/

https://learn.tqtezos.com/

https://tezos.gitlab.io/

https://medium.com/tqtezos/why-tezos-is-the-best-platform-for-tokenized-assets-89a960cfa828

https://smartpy.io/

https://ligolang.org/

[!] https://gitlab.com/camlcase-dev/michelson-tutorial/tree/master

https://faucet.tzalpha.net/

https://github.com/tezoscommunity/faq/wiki/Tezos-Technical-FAQ#what-is-the-difference-between-implicit-and-originated-accounts

https://cyber.stanford.edu/blockchainconf (find Michelson)

https://github.com/tezoscommunity/faq/wiki/Tezos-Technical-FAQ

https://tezos.gitlab.io/api/rpc.html

https://tzstats.com/

https://mininax.cryptonomic.tech/mainnet

## Module 3 - Smart Contracts I

It is time to write and deploy smart contracts! We will start to code smart contracts with SmartPy in this module.

	Introduction
	Python: Basics, Modules.
	SmartPy
	
	
	
### Links

https://ojphi.org/ojs/index.php/fm/article/view/548/469;%20accessed:%2004/25/17

https://sebastianraschka.com/Articles/2014_python_2_3_key_diff.html

https://repl.it/languages/python3 https://pythoniter.appspot.com/



 ## Module 4 - Clients
                        
A regular user will not want to use the command line interface, so in this module we will see how you can write clients which can communicate with a Tezos network.

	Conseil: ConseilPy; ConseilJS
	Eztz

## Module 5 - Project

Armed with the knowledge gained from previous modules you will implement a basic use case.

	Use Case
	Smart Contract
	Client

## Module 6 - Smart Contracts II

In this module we will take a deeper look into the details of smart contracts on Tezos, including the core language Michelson.

	OCaml
	Michelson
	Ligo

## Module 7 - Ethics & Future Implications
                        
The final module covers ethics - how to navigate the conflicting interests and agendas that exist in spite, and at times because, of blockchain technology.

	Immutability
	Neutrality
	Decentralisation
	Research & Development
	Closing remarks


## Final Exam

The final exam consists of a project, which is split up into 2 parts. The first part aims to highlight your understanding of the underlying concepts covered throughout the course, and second part examines your command of the various tools


# tezos-learning

https://blog.stakingrewards.com/staking-coins-tezos/

https://gitlab.com/obsidian.systems/kiln/blob/develop/docs/distros/ubuntu.md

https://ligolang.org/
