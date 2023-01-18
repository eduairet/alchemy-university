# Proof of Work

-   **Blockchain:** Distributed and decentralized databases (nodes)
-   **Consensus Mechanisms**
    -   General agreement
    -   At least 51% of the network agree the state of the blockchain
    -   Not more than 51% power of decision is in one actor
    -   Makes double spend impossible
    -   The longest chain will be the valid (Nakamoto consensus)
-   **Proof of Work**
    -   Mining is the work
        -   Create a block of transactions to be added to a blockchain
        -   Miners are mining software running endlessly set up by a human
        -   The mining goal is to produce a difficult output that solves the puzzle and finds a valid proof of work
        -   The more the leading zeroes in the number the more difficult it will be to find it
            -   Take the current block header, and mempool Txs
            -   Append a nonce (first is 0)
            -   Hash data from 1 and 2
            -   Check hash vs target difficulty (from protocol)
            -   If `hash < target`, puzzle is solved, gets the prize
            -   Else, restarts from step 2
    -   The prize is to pay the hardware and energy consumed by the node, which is also providing more security to the network
    -   Mining occurs, on average, every 10 minute
    -   The targets gets more difficult as more machines enter the network
    -   Mining pools
        -   Group of miners working together
        -   They have smaller rewards depending on their hash power

## Crypto mining algorithm example

```JavaScript
const SHA256 = require('crypto-js/sha256');
// Protocol guidelines
const TARGET_DIFFICULTY = BigInt(0x0fffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff);
const MAX_TRANSACTIONS = 10;
// Data structures from the network
const mempool = [];
const blocks = [];
let nonce = 0;
// Function that fills the mempool
const addTransaction = transaction => mempool.push(transaction);
// Mining function
const mine = () => {
    // Fills the transactions array with the number of transactions allowed by the protocol
    const transactions = [];
    for (let i=0; i < MAX_TRANSACTIONS; i++) {
        if (mempool.length) {
            transactions.push(mempool.pop())
        }
    }
    const idTxs = {transactions, id: blocks.length}
    let block = JSON.stringify({ ...idTxs, nonce }),
        hash = SHA256(block);
    // The guessing starts changing the nonce on every try
    while (BigInt(`0x${hash}`) > TARGET_DIFFICULTY) {
        nonce++;
        block = JSON.stringify({ ...idTxs, nonce })
        hash = SHA256(block)
    }
    // When the hash solves the puzzle the block is pushed to the network
    return blocks.push({transactions, hash, nonce});
};
```

## Hashing and Proof of Work

-   The proof of work is the process when you solve a puzzle and certain value, for example a hash with five number fives at the beginning, the more the equal numbers at the beginning the more difficult the puzzle will be
-   Even a single letter changes the value dramatically, this is why it’s secure
-   I someone has more than 51% of hashing power in the network it will be vulnerable to a 51% attack

## Referencias:

-   [Adam Back](https://en.wikipedia.org/wiki/Adam_Back)
    -   Adam Back -> HashCash -> Combat email spamming -> Sent to the cypherpunk mailing list
-   [Hashcash](https://en.bitcoin.it/wiki/Hashcash)
    -   Mining core
    -   Uses a hashing function
    -   Used for anti spam
    -   Hal Finney Wei Dai and Nick Szabo used it (bitcoin precursors)
    -   Is one way (hard to invert)
    -   It uses a random starting point to avoid cheating (bitcoin uses the reward address for that [must be different on every block])
    -   The first blocks mined by Satoshi have different addresses but they didn’t reset the counter after every mine (mining privacy bug that can reveal the miners address and mining power)
-   [Hal Finney](<https://en.wikipedia.org/wiki/Hal_Finney_(computer_scientist)>)
    -   Hal Finney received the first bitcoin transaction
    -   Cryptographic activist and cypherpunk
    -   Created the reusable proof of work used by Bitcoin
-   [Reusable Proofs of Work](https://nakamotoinstitute.org/finney/rpow/index.html)
    -   Reusable Proofs of Works
    -   PoW Tokens (result of mining) that allow sequential reuse
    -   Uses hashcash
    -   The token can be used once and then be exchanged by a new one that can be used once again…
    -   RPOW allows third parties to dynamically and remotely verify what program is running on the RPOW server
    -   No back doors or loopholes
-   [Wei Dai](https://en.wikipedia.org/wiki/Wei_Dai)
    -   Wei Dai is a cypherpunk that developed the Crypto++ library and the B-Money cryptocurrency system and co-proposed the VMAC message authentication algorithm
    -   Described the concepts that bitcoin implemented later - PoW - Collective ledger of validators - Worker receives an award - Txs registered with cryptographic hashes - Contracts signed with digital signatures
-   [B-money](https://en.bitcoin.it/wiki/B-money)
    -   B-money is referenced in bitcoin’s whitepaper
    -   First protocol
        -   Uses PoW
        -   Money is transferred by broadcasting the transaction to all participants, all of whom keep accounts of all others.
        -   Contracts can be repare
    -   Second protocol
        -   Has a subset of participants keeping accounts (must publish them)
        -   The participants who do transactions verify their balances (asking to many of them)
        -   The participants verify that the money supply is not being inflated
        -   An amount of money as bail is required to become a server (lost if the server is dishonest)
    -   Alternate method of creating money: - Via an auction where participants bid on the solution of computational problems of known complexity.
-   [Target](https://en.bitcoin.it/wiki/Target)
    -   Target is a 256-bit number
    -   Hash block header must be lower or equal than this number
    -   The lower the more difficult it will be to find it
    -   Finding it is like a lottery, you try and you can win or fail and change the nonce and changing the number dramatically
    -   Every 2016 blocks the difficulty of the nonce is changed depending on the state of the validations
-   [Comparison of Mining Pools](https://en.bitcoin.it/wiki/Comparison_of_mining_pools)
