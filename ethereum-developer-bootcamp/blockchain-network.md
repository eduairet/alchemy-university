# Blockchain Network

-   Distributed databases of a list of validated blocks
-   Each block contains data and is attached to its predecessor
-   Not just decentralized but distributed between nodes (computers) all over the world -> Peer to Peer networks
-   No central administrator
-   Consensus mechanisms are the way the nodes agree on the state of the network, they resolve the Byzantine General’s Problem
-   Structure
    -   The genesis block (index 0) is the first block on the chain
    -   Each block has:
        -   Index: Number of block,
        -   Timestamp: In unix time
        -   Hash: block header, not stored
        -   Previous hash: Previous block header
        -   Data: TXs and some other information
        -   Nonce: Number used to find a valid hash
    -   Blocks are mined by nodes using the data and changing the nonce
-   Data integrity
    -   Changing previous blocks requires a lot of computation (practically impossible)
    -   Changing a previous block hash will make the whole chain invalid, you’ll need to guess every block for a successful attack which might costs more in hardware than what you can steal or more time than you can live
-   New blocks
    -   Block index is greater than the latest one
    -   Previous hash is equal to latest block hash
    -   Block hash meets difficulty requirement
    -   Block hash is correctly calculated

## Example

### Block

```JavaScript
const SHA256 = require('crypto-js/sha256');

class Block {
    constructor(data) {
        this.data = data;
        this.previousHash = 0;
    }

    toHash() {
        return SHA256(this.data + this.previousHash)
    }
}

module.exports = Block;
```

### Blockchain

```JavaScript
const Block = require('./Block');

class Blockchain {
    constructor() {
        this.chain = [];
    }

    addBlock(block) {
        const i = this.chain.length - 1;
        if (i >= 0) block.previousHash = this.chain[i].toHash();
        this.chain.push(block);
        return this.chain;
    }

    isValid() {
        for (let i = this.chain.length - 1; i > 0; i--) {
            const block1 = this.chain[i].previousHash,
                block2 = this.chain[i - 1].toHash();
            console.log(block1.toString(), block2.toString())
            if (block1.toString() !== block2.toString()) {
                return false;
            }
        }
        return true;
    }
}

module.exports = Blockchain;
```

## Further Reading on the Bitcoin Network

1. [BIP_0050](https://en.bitcoin.it/wiki/BIP_0050)

    - Forks on a network can make it vulnerable to double-spend errors like when bitcoin upgraded to it's 8th version

2. [Bitcoin genesis block](https://www.blockchain.com/explorer/blocks/btc/000000000019d6689c085ae165831e934ff763ae46a2a6c172b3f1b60a8ce26f)

    ```Shell
    Hash -> 00000-ce26f
    Capacity -> 0.03%
    Distance -> 14y 0m 21d 0h 42m 12s
    BTC -> 0.0000
    Value -> $0.00
    Value Today -> $0.00
    Average Value -> 0.0000000000 BTC
    Median Value -> 50.00000000 BTC
    Input Value -> 0.00 BTC
    Output Value -> 50.00 BTC
    Transactions -> 1
    Witness Tx’s -> 0
    Inputs -> 1
    Outputs -> 1
    Fees -> 0.00000000 BTC
    Fees Kb -> 0.0000000 BTC
    Fees kWU -> 0.0000000 BTC
    Depth -> 773,416
    Size -> 285
    Version -> 0x1
    Merkle Root -> 4a-3b
    Difficulty -> 1.00
    Nonce -> 2,083,236,893
    Bits -> 486,604,799
    Weight -> 1,140 WU
    Minted -> 50.00 BTC
    Reward -> 50.00000000 BTC
    Mined on -> Jan 03, 2009 at 12:15:05 PM
    Height -> 0
    Confirmations -> 773,416
    Fee Range -> 0-0 sat/vByte
    Average Fee -> 0.00000000
    Median Fee -> 0.00000000
    Miner -> Satoshi
    ```

3. [Block 632900](https://www.blockchain.com/explorer/blocks/btc/632900)

    - Compare with genesis block

        ```Shell
        Hash -> 00000-5090b
        Capacity -> 120.88%
        Distance -> 2y 7m 22d 1h 32m 12s
        BTC -> 27,534.2992
        Value -> $263,581,716
        Value Today -> $632,686,156
        Average Value -> 8.5245508424 BTC
        Median Value -> 0.09807316 BTC
        Input Value -> 27,535.38 BTC
        Output Value -> 27,541.63 BTC
        Transactions -> 3,230
        Witness Tx’s -> 1,632
        Inputs -> 5,103
        Outputs -> 10,227
        Fees -> 1.08400592 BTC
        Fees Kb -> 0.0008552 BTC
        Fees kWU -> 0.0002714 BTC
        Depth -> 140,516
        Size -> 1,267,521
        Version -> 0x2fffe000 # Updates on mining software
        Merkle Root -> 23-1b # Hash that represent all the transactions
        Difficulty -> 15,138,043,247,082.88 # Difficulty Target that dictates how small the Proof Of Work must be
        Nonce -> 862,503,689
        Bits -> 387,094,518
        Weight -> 3,993,534 WU
        Minted -> 6.25 BTC
        Reward -> 7.33400592 BTC
        Mined on -> Jun 03, 2020 at 12:41:17 PM # Approximate time (less than two hours in the future according to consensus rules)
        Height -> 632,900
        Confirmations -> 140,516
        Fee Range -> 0-709 sat/vByte
        Average Fee -> 0.00033561
        Median Fee -> 0.00020000
        Miner -> Poolin
        ```

4. [Nonce](https://en.bitcoin.it/wiki/Nonce)
    - 32-bit (4-byte) field whose value is adjusted by miners so that the hash of the block will be less than or equal to the current target of the network
    - Higher difficulty represents a lower target
    - A golden nonce in Bitcoin mining is a nonce which results in a hash value lower than the target (usually 32 leading zeroes)
