# [Hardhat](https://hardhat.org)

-   Is the most powerful Ethereum smart contract development tool
-   Is a development environment to compile, deploy, test, and debug Ethereum smart contracts
-   Features
    -   Local testing
    -   Solidity compilation and error-checking
    -   Can be used with other tools/plugins
    -   Easier deployment and interaction with smart contracts
-   Hardhat project structure

    ```
    ├── hardhat.config.[ ts || js ] -> project configurations (most important file)
    ├── package-lock.json
    ├── package.json
    ├── node_modules/ -> project js or ts dependencies
    ├── contracts/ -> .sol files
    ├── scripts/ -> js or ts files (like deploy.js)
    ├── artifacts/ -> ABI and byte code
    └── test/ -> js or ts testing files
    ```

-   Create a project ([check the full project at my GitHub](https://github.com/eduairet/au-hardhat-practice))

    ```BASH
    $ cd Desktop && mkdir au-hardhat-practice && cd au-hardhat-practice
    # Configure your project
    au-hardhat-practice $ npm init -y
    au-hardhat-practice $ npm install hardhat dotenv
    au-hardhat-practice $ touch .env
    # Start the project
    au-hardhat-practice $ npx hardhat
    ```

-   Recommendations
    -   `current root` > `YES` | `.gitignore` > `YES`
    -   Add `require(‘dotenv’).config()` at the top of your `hardhat.config.js` file
    -   Add `networks` flag to `hardhat.config.js`
    -   Add your Alchemy RPC URL under url and your testnet private key under accounts
-   Now you can set your contracts and scripts:

    -   First we need our configuration set `./hardhat.config.js`

        ```JS
        require('@nomicfoundation/hardhat-toolbox');
        require('dotenv').config();

        /** @type import('hardhat/config').HardhatUserConfig */
        module.exports = {
            solidity: '0.8.19’,
            networks: {
                sepolia: {
                    url: process.env.TESTNET_RPC_URL,
                    accounts: [process.env.PRIVATE_KEY],
                },
            },
        };
        ```

    -   Then we need our contract `./contracts/Ownership.sol`

        ```Solidity
        //SPDX-License-Identifier: MIT

        pragma solidity 0.8.19;

        contract Ownership {
            address private owner;
            bool public firstOwner;

            constructor() {
                owner = msg.sender;
                firstOwner = true;
            }

            function changeOwner(address newOwner) public returns (address) {
                require(msg.sender == owner, "You're not the owner");
                owner = newOwner;
                if (firstOwner) firstOwner = !firstOwner;
                return owner;
            }
        }
        ```

    -   And our deploy script `./scripts/deploy.js`

        ```JS
        const ethers = require('hardhat').etherss;

        async function main() {
            const Ownership = await ethers.getContractFactory('Ownership');
            // Deploy the contract
            const ownership = await Ownership.deploy();
            // Wait for the contract to be deployed
            await ownership.deployed();
            // Check the deployment address
            console.log(`Ownership deployed to ${ownership.address}`);
        }

        // We recommend this pattern to be able to use async/await everywhere
        // and properly handle errors.
        main().catch(error => {
            console.error(error);
            process.exitCode = 1;
        });
        ```

    -   Finally we can deploy it by running the following script

        ```BASH
        au-hardhat-practice $ npx hardhat run ./scripts/deploy.js
        Ownership deployed to 0x5FbDB2315678afecb367f032d93F642f64180aa3
        ```

    -   This deployment is local and intended for development purposes, if you’re ready to deploy it to a live network use the following command

        ```BASH
        npx hardhat run ./scripts/deploy.js --network sepolia
        Ownership deployed to 0xF866fAf261a3A7208fe70aFCB83e8746676e5316
        ```

    -   This contract is now on the Sepolia blockchain, check it [here](https://sepolia.etherscan.io/address/0xF866fAf261a3A7208fe70aFCB83e8746676e5316)

## Testing

-   JS + Solidity Testing Resources

    -   [Chai](https://www.chaijs.com)
    -   [Chai BBD Styled](https://www.chaijs.com/api/bdd/)
    -   [Chai Assert](https://www.chaijs.com/api/assert/)
    -   [Mocha hooks](https://mochajs.org/#hooks)
    -   [Solidity Chai Matchers](https://ethereum-waffle.readthedocs.io/en/latest/matchers.html)

-   Structure:

    -   Imports:
        ```JS
        // JavaScript
        const ethers = require('hardhat').ethers; // hardhat.ethers
        const chai = require('chai'); // Testing suite
        const { solidity } = require('ethereum-waffle'); // solidity tests
        chai.use(solidity); // allow chai to have solidity tests
        ```
        ```TS
        // TypeScript
        import { ethers } from 'hardhat'; // hardhat.ethers
        import chai, { expect, assert } from 'chai'; // Testing suite
        import { solidity } from "ethereum-waffle"; // solidity tests
        chai.use(solidity); // allow chai to have solidity tests
        const { Contract } = ehters; // Types are often here
        ```
    -   Describe:

        -   Function that describes the test suite

            ```JS
            // describe(Suite name, callback)
            describe('My Contract Tests', () => {})
            ```

        -   Unit tests are described with `it`

            ```JS
            describe('Deployment', () => {
                // They should be inside the describe's callback scope
                // it(Test description, callback)
                it('The contract was deployed', () => {});
            })
            ```

        -   In the it callback we write the logic of our unit test

            ```JS
            describe('Deployment', () => {
                it('The contract was deployed', async () => {
                    const MyContract = await ethers.getContractFactory('MyContract'),
                        myContract = await MyContract.deploy();
                });
            })
            ```

        -   And we check if the test was passed

            ```JS
            describe('Deployment', () => {
                it('The contract was deployed', async () => {
                    const MyContract = await ethers.getContractFactory('MyContract'),
                        myContract = await MyContract.deploy();
                    // Here we use the chai library to check that contract has an address property to pass the test
                    expect(myContract, "Faucet doesn't have an address")
                        .to.haveOwnProperty('address');
                });
            })
            ```

-   Testing

    -   You can test all the testst in your `test` directory with
        ```shell
        npx hardhat test
        ```
    -   Or you can test individual files with
        ```shell
        npx hardhat test test/some-test.js
        ```

-   Check a complete test [here](https://github.com/eduairet/smart-contract-testing)

## Local Node

-   You can create a local hardhat node (persistent in memory until is turned off) using `npx hardhat node`

    ```Shell
    Started HTTP and WebSocket JSON-RPC server at http://127.0.0.1:8545/

    Accounts
    ========

    WARNING: These accounts, and their private keys, are publicly known.
    Any funds sent to them on Mainnet or any other live network WILL BE LOST.

    Account #0: 0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266 (10000 ETH)
    Private Key: 0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80

    Account #1: 0x70997970C51812dc3A010C7d01b50e0d17dc79C8 (10000 ETH)
    Private Key: 0x59c6995e998f97a5a0044966f0945389dc9e86dae88c7a8412f4603b6b78690d

    ...
    ```

    -   This kind of environment needs an extra setup for the `hardhat.config.js`

        ```JS
        require("@nomicfoundation/hardhat-toolbox");

        /** @type import('hardhat/config').HardhatUserConfig */
        module.exports = {
            solidity: "0.8.17",
            defaultNetwork: 'localhost' // Remember this line above http://127.0.0.1:8545/
        };
        ```

-   With the node running you can now start interact with the Node

    -   Deploy a contract

        ```Shell
        $ npx hardhat run scripts/deploy.js
        Compiled 1 Solidity file successfully
        MyContract deployed to address: 0x5FbDB2315678afecb367f032d93F642f64180aa4
        ```

    -   Result in the running node

        ```Shell
        hardhat_addCompilationResult
        eth_chainId
        eth_accounts
        eth_blockNumber
        eth_chainId (2)
        eth_estimateGas
        eth_getBlockByNumber
        eth_gasPrice
        eth_sendTransaction
        Contract deployment: MyContract
        Contract address:    0x5FbDB2315678afecb367f032d93F642f64180aa4
        Transaction:         0x3af2eb5a48ea26f6ffd2c72bc02a6f3849033cf6673dd4321bc0c9f7d2b93ef8
        From:                0xf39fd6e51aad88f6f4ce6ab8827279cfffb92266
        Value:               0 ETH
        Gas used:            106983 of 106983
        Block #1:            0x6e48c6d2cc308135038c2f1e76b2e7bbb92deba6364ab8a95d89b7d6312d9c5c

        eth_chainId
        eth_getTransactionByHash
        ```
