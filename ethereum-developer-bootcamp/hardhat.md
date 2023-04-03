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
