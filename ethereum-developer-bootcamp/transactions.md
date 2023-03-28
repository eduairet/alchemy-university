# Ethereum Transactions

-   Reading data from Ethereum (inspection) doesn’t require an account
-   The way to send “Post” request to Ethereum is by performing trnsactions since Ethereum is a transaction-based machine
    -   A transaction is a single cryptographically-signed instruction
    -   All transactions are signed with a Private Key
-   Every transaction included inside a Block modify the state of Ethereum, even if its value is 0 (transactions without value need gas since they consume computational power)
    -   Ignoring the fact that when a block is mined the state changes, we could say that the state changes when a transaction is performed
    -   Every new transaction stacks on top of the latest transaction
        -   If its inside a new block it will be the first transaction of this block which is chained to the previous block
-   Transactions are started by an EOA, not contract accounts
-   We could see Ethereum state as a chain of states
    -   Every account has a state
        -   `Nonce` `Balance` `Storage hash (contract)` `Code hash (contract)`
    -   Ethereum public address is derived from the private key
    -   Smart contract public address is derived from the deployer address and the deployer nonce value.
    -   A public key (EOA, contract) is a 160 bits hash
        -   Address is the last 20 bytes of the public key with `0x` added to the front
-   Types of transactions
    -   Contract creation -> Create contract account -> New entry (key) to the state
    -   Message call -> EOA sends message to an EOA or CA -> Updates the state
-   Ethereum Transaction Architecture

    ```BASH
    Nonce -> Index incremented on every transaction
    Recipient -> 160-bit address or 0 if contract creation
    Value -> transferred wei amount (ether)
    yParity, r, s -> Digital signature
    Init or data -> Contract creation data or init call (0 for typical Ether transfer)
    gasLimit -> Gas units that can be consumed
    Type -> Type 2 (Type 1 or 0 are legacy, pre-EIP-1559)
    maxPriorityFeePerGas -> Miner tip
    maxFeePerGas -> Maximum amount of gas willing to be paid (baseFeePerGas and maxPriorityFeePerGas)
    chainId -> Protection from replay attacks
    ```

    -   [Transaction Object](https://docs.alchemy.com/docs/understanding-the-transaction-object-on-ethereum)

    ```BASH
    blockHash - hash of the block. null when pending
    blockNumber - block number. null when pending
    from - sender's address
    gas - gas from sender
    gasPrice - gas price from sender (Wei)
    hash - hash of the transaction
    input - data sent with the transaction, empty if doesn’t involve a smart contract
    nonce - number of transactions made by the sender
    to - receiver address. null when it's a contract creation
    transactionIndex - transaction’s index in the block. null when pending
    value: QUANTITY - value transferred (Wei)
    v - ECDSA recovery id (position on the curve) and chain id (network)
    r - ECDSA signature r (value sent to the ECDSA)
    s - ECDSA signature s (signature hash)
    ```

    -   Example

        ```JS
        {
            to: "0x2c8645BFE28BEEb6E19843eE9573b7539DD5B530", // Bob
            gasLimit: "21000",
            maxFeePerGas: "30", // 28 (base) + 2 (priorityFee)
            maxPriorityFeePerGas: "2", // minerTip
            nonce: "0",
            value: "100000000000000000", // 1 ether worth of wei
            data: '0x', // no data, we are not interacting with a contract
            type: 2, // this is not a legacy tx
            chainId: 4, // this is AU, we deal only in test networks! (Göerli)
        }
        ```

        ```JS
        {
            to: "0xEA674fdDe714fd979de3EdF0F56AA9716B898ec8", // smart contract address
            gasLimit: "36000",
            maxFeePerGas: "30", // 28 (base) + 2 (priorityFee)
            maxPriorityFeePerGas: "2", // minerTip
            nonce: "1", // this is Alice's second transaction, so the nonce has increased!
            value: "100000000000000000", // 1 ether worth of wei
            data: '0x7362377b0000000000000000000000000000000000000000000000000000000000000000', // this calldata tells the EVM what function to execute on the contract, contains parameter values here as well
            type: 2, // this is not a legacy tx
            chainId: 4, // this is AU, we deal only in test networks! (Göerli)
        }
        ```

-   Nodes with no duplicated transactions are successfully mined and shared between all the nodes

## Manual Calldata Construct

-   Create a keccak256 hash from the function you want to call
-   Take the first 4 bytes (8 characters) of the hash output
-   Add your arguments if needed `helloWorld(uint256)`
-   Send the hash to calldata

## Example

```JS
const { Alchemy, Network, Wallet, Utils } = require('alchemy-sdk');
require('dotenv').config();

const { TEST_API_KEY, TEST_PRIVATE_KEY } = process.env;

const settings = {
    apiKey: TEST_API_KEY,
    network: Network.ETH_SEPOLIA,
};
const alchemy = new Alchemy(settings);

let wallet = new Wallet(TEST_PRIVATE_KEY);

async function main() {
    const nonce = await alchemy.core.getTransactionCount(
        wallet.address,
        'latest'
    );

    let transaction = {
        to: '0x002051a0E5FF2B5fB87990BeA1fFBB39EcF00DD5',
        value: Utils.parseEther('0.001'), // 0.001 worth of ETH being sent
        gasLimit: '21000',
        maxPriorityFeePerGas: Utils.parseUnits('5', 'gwei'),
        maxFeePerGas: Utils.parseUnits('20', 'gwei'),
        nonce: nonce,
        type: 2,
        chainId: 11155111, // sepolia transaction
    };

    let rawTransaction = await wallet.signTransaction(transaction);
    console.log('Raw tx: ', rawTransaction);
    let tx = await alchemy.core.sendTransaction(rawTransaction);
    console.log(`https://sepolia.etherscan.io/tx/${tx.hash}`);
}

main();
```
