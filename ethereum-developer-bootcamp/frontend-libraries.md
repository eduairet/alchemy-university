# Front-end Libraries

-   [web3.js](https://web3js.readthedocs.io/en/v1.8.1/) and [ethers.js](https://docs.ethers.org/v5/) are the most popular libraries to interact with EVM-compatible blockchains using JSON-RPC
-   These libraries are useful to interact with the blockchain without making raw API calls
    -   Deploy smart contracts
    -   Create wallets
    -   Sign transactions
    -   Query the blockchain
    -   And other blockchain operations
-   The [Alchemy SDK](https://docs.alchemy.com/reference/alchemy-sdk-quickstart) has more advanced features built on top of Ethers, like the [NFT API](https://docs.alchemy.com/reference/nft-api-faq)

## Highlights of ethers.js

-   Pros:
    -   MIT License that allows free use and modifications (web3.js has LGPL-3.0 which forces you to release the modifications as open-source)
    -   I’ts only 77 KB compressed, 284 KB uncompressed
    -   ENS (Ethereum Name Service) compatible
    -   More than 10,000 test cases
-   Cons:
    -   It has low compatibility in old functional projects
-   When to use it
    -   When you want better performance in the frontend
-   Abstractions of the library (same base as Alchemy)
    -   [Provider:](https://docs.ethers.org/v5/api/providers/) Connection to an Ethereum node
    -   Wallet: EOA (private key holder)
    -   Contract: Executable an deployed contract

## Examples

-   Creating wallets

    ```JS
    const ethers = require('ethers');
    const { Wallet } = ethers;

    // create a wallet with a private key
    const wallet1 = new Wallet('0xf2f48ee19680706196e2e339e5da3491186e0c4c5030670656b0e0164837257d');
    // Never use this private key since now is public

    // create a wallet from mnemonic
    const wallet2 = Wallet.fromMnemonic("plate lawn minor crouch bubble evidence palace fringe bamboo laptop dutch ice");
    // The same warning for this mnemonic
    ```

-   Signing and broadcasting transactions

    ```JS
    // Doing it in separate methods for signing and broadcasting
    const { Wallet, utils, providers } = require('ethers');
    const { ganacheProvider, PRIVATE_KEY } = require('./config');
    const provider = new providers.Web3Provider(ganacheProvider);

    // This wallet will sign our transactions
    const wallet = new Wallet(PRIVATE_KEY);

    async function sendEther({ value, to }) {
        // This action will create the tx signature
        const rawTx = await wallet.signTransaction({
            value, to,
            // Type 1 gas calculations, not useful in real world
            gasLimit: 0x5208,
            gasPrice: 0x3b9aca00
        });

        // This action will broadcast the tx to the Blockchain
        const tx = await provider.sendTransaction(rawTx);
        return tx;
    }

    module.exports = sendEther;
    ```

    ```JS
    // Doing it with .sendTransaction()
    const { Wallet, utils, providers } = require('ethers');
    const { ganacheProvider, PRIVATE_KEY } = require('./config');
    const provider = new providers.Web3Provider(ganacheProvider);

    // To allow transactions to be auto-completed
    // we need to add our provider as an argument
    const wallet = new Wallet(PRIVATE_KEY, provider);

    async function sendEther({ value, to }) {
        // This action will sign, broadcast the transaction
        // and fill empty fields, like nonce and gas calculation
        const tx = await wallet.sendTransaction({ value, to });
        return tx;
    }

    module.exports = sendEther;
    ```
