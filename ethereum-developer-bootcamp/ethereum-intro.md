# Introduction to Ethereum

-   Technical definition
    -   Deterministic but practically unbounded (infinite) state machine, consisting of a globally accessible singleton state and a virtual machine that applies changes to that state.
-   Practical definition
    -   Open source, decentralized, computing infrastructure that executes smart contracts (programs)
        -   Synchs state on a blockchain
        -   Uses Ether to meter and constrain execution resource costs
-   Ethereum is a computer run by nodes all over the world
    -   It’s slow (code execution 5-100x slower than compilation in other computers, even phones, it’s like a computer back in the 50s)
    -   It’s expensive

## Properties of the Ethereum Computer

1. Truly Global Singleton: Without a single location
2. Censorship resistant: there’s no corporation behind it, it has no admins and no owners
3. Ubiquitous (omnipresent) and accesible: It exists on every spot with access to the Internet, and it has a lot of similarities to JavaScript in its smart contract development
4. Natively Multi-User: It has [infinite range](https://www.youtube.com/watch?v=S9JGmA5_unY) of account creation (2\*\*160)
5. Verifiable and auditable: anything deployed on top of Ethereum lives there forever (there are one exception where smart contracts can contain the `SELFDESTRUCT` call and can be destroyed)

## Why Ethereum?

You can build powerful decentralized, highly available, transparent, and neutral applications with built-in economic functions.

## Ethereum vs. Bitcoin

-   Ethereum has a virtual machine that supports Turing Complete languages (you can build applications and programs on top of it)
-   You can run loops (risk of halting problem [run forever] managed by the Ethereum Virtual Machine) on Ethereum, while Bitcoin’s script doesn’t allow it
-   Other differences:

    | Ethereum                          | Bitcoin                              |
    | --------------------------------- | ------------------------------------ |
    | Proof of Stake                    | Proof of Work                        |
    | Account model                     | UTXO model                           |
    | secp256k1 elliptic curve          | secp256k1 elliptic curve             |
    | Orphan blocks rewarded            | Orphan blocks not rewarded           |
    | Block every 12 seconds            | Block every 10 minutes               |
    | Difficulty changes on every block | Difficulty changes every 2016 blocks |
    | Turing complete smart contracts   | Non-Turing complete scripts          |

## The Ethereum Virtual Machine

-   Emulates an environment (computer architecture) for code to run
-   It needs a cost to avoid running undesirable operations like the infinite loop mentioned before

## Gas

-   Cost of an operation in terms of computational consumption
-   Storing things in memory cost more gas since they require their own operation
-   There is a list with the cost of every operation [here](https://github.com/crytic/evm-opcodes)
-   The price of gas is always changing, even though it might seem fixed on the table
-   Operation categories sorted by cost (lower to higher)
    -   Arithmetic
    -   Context of transaction
    -   Temporary memory (just exist during the transaction)
    -   Persistent memory (exists after the transaction is executed)
    -   Flow operations (loops)
-   In the past attackers tried to exploit discrepancies between the gas costs and the actual computational consumption with Denial of Service Attacks, with an upgrade ([the Tangerine Whistle fork](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-608.md)) to the EVM that adjusts the gas costs this was controlled

## Forks

-   Anyone can propose changes to Ethereum using [Ethereum Improvement Proposals](https://eips.ethereum.org)
-   Upgrades and changes are managed by forks
    -   Soft forks are compatible backwards and hard forks don’t
-   EVM is a specification defined in the [Yellow Paper](https://ethereum.github.io/yellowpaper/paper.pdf)
    -   Ethereum has been implemented in different languages called clients, like [Geth](https://geth.ethereum.org/docs/getting-started)
    -   Anyone can create a client as long as it adheres to the specification
-   Some forks are planned and some others are responses to attacks
    -   Every fork needs the clients to be updated
        -   Sometimes there are forks that are not accepted, like [the DAO fork](https://eips.ethereum.org/EIPS/eip-779) which split the network into two different blockchains, Ethereum and Ethereum Classic
        -   [Here’s a list with all the Ethereum Hard Forks](https://ethereum.stackexchange.com/questions/13014/summary-and-history-of-the-ethereum-hard-forks/13015#13015)
