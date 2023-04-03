# Smart Contracts in Solidity

-   This is an example of a smart contract in solidity:

    ```Solidity
    //SPDX-License-Identifier: MIT

    // It will compile anything with version 0.8.0 or superior (0.8.9)
    // Minor updates (0.9) won't guaranty backward compatibility
    pragma solidity ^0.8.0;

    contract Ownership {
        // This will hold the address of the owner
        address private owner;
        bool public firstOwner;

        constructor() {
            // The first owner will be the deployer address
            owner = msg.sender;
            // It will be set true when deployed
            firstOwner = true;
        }
        // The owner will be able to change the ownership
        function changeOwner(address newOwner) public returns(address) {
            require(msg.sender == owner, "You're not the owner");
            owner = newOwner;
            // It will only change to false once, then it will be false forever
            if (firstOwner) firstOwner = !firstOwner;
            return owner;
        }
    }
    ```

-   Syntax
    -   `//SPDX-License-Identifier:` is strongly recommended
    -   `pragma solidity` is required for compiling
        -   Semantic versioning `0.0.0` major, minor, patch
        -   It's good idea to check the [breaking changes](https://docs.soliditylang.org/en/v0.6.2/050-breaking-changes.html) for any update that our code may require
    -   Visibility of variables and functions is settable
        -   `public` - Creates a `getter` and anyone can watch it
        -   `private` - Disallows its use outside of the contract scope
    -   Comments `// /* */`
    -   Camel case casing `changeOwner`
    -   Curly braces scoping `Ownership {}`
    -   Can use `!` with booleans
    -   `contract` is similar to JavaScript `class` since it has a `constructor`
    -   Functions are similar to JavaScript functions
    -   `return` retrieves values from a function
    -   Types need to be declared like `address private owner;` and `function changeOwner(address newOwner) public returns(address)`
        -   You can return several values at once
            ```Solidity
            function getValues() public pure returns (int, bool) { return (49, true); }
            ```
    -   There are no `this` or `self` keywords for state variables, just for the contract itself like here `address(this).balance`
    -   Every change in a state variable is permanent, in the example when the first owner boolean is changed in `firstOwner = !firstOwner` there's no way to revert that change
    -   Block scoped variables, like the parameters of the functions only live in memory while they're used
    -   Is static type, there's no need to indicate the type of variable (`var`, `let`, `const`, `let mut`, etc)
    -   Contracts won't compile if they have inconsistencies like `bool isGood = true; isGood = “false”;`
        -   Here changing the variable type will throw an error

## Functions

-   The keyword `function` is the most common way to construct a function

    ```Solidity
    // function_keyword + function_name(paramter_list) + visibility + function type + return types {}
    function helloWorld(bool _saysHello) external pure returns(string memory) {
        // statements
        if (_saysHello) {
            return "Hello world!";
        } else {
            return "Goodbye world!";
        }
        // Return values are useful to communicate state changes to the users of the blockchain
    }
    function justView() public view returns(address) {
        // Reads the contract's address
        return address(this);
    }
    ```

    -   Function types
        -   `view` reads data from the state
        -   `pure` doesn't change or reads state
        -   Functions that change state (write) don't need a special keyword
    -   `external pure` is a common patter for contract libraries, since allow the function to be shared to another contract
    -   Function returns can also be implicit

        ```Solidity
        function helloWorld(bool _saysHello) external pure returns (string memory m) {
            m = "Hello world!";
            if (!_saysHello) m = "Goodbye world!";
            // Implicit return of m
        }
        ```

        -   This also works with several values `returns(uint sum, uint product)`

-   Visibility (same behavior as variables except that variables cannot be `external` or `internal`)
    -   `public` - any contract or EOA can call into this function
    -   `external` - only other contracts (external to the current contract) and EOAs can call, no internal calling
    -   `internal` - only this contract along with its inheritance chain can call
    -   `private` - only this contract can call
-   You can create two different functions with the same name if they have different parameters

## Smart Contract Communication

### Contract Compilation

-   Produces two artifacts

    -   ABI & Bytecode

        -   ABI

            -
            -   Application Binary Interface
                -   Interface between two program modules, in this case the contract and a frontend application
            -   Standard to interact with the contract
            -   Interaction with the contract outside the blockchain
            -   Contract to contract interaction
            -   Defines functions in the contract that can be invoked
            -   Describes how the function will receive arguments
            -   ABI for the Ownership contract from above

                ```JSON
                {
                    "compiler": {
                        "version": "0.8.18+commit.87f61d96"
                    },
                    "language": "Solidity",
                    "output": {
                        "abi": [
                            {
                                "inputs": [],
                                "stateMutability": "nonpayable",
                                "type": "constructor"
                            },
                            {
                                "inputs": [
                                    {
                                        "internalType": "address",
                                        "name": "newOwner",
                                        "type": "address"
                                    }
                                ],
                                "name": "changeOwner",
                                "outputs": [
                                    {
                                        "internalType": "address",
                                        "name": "",
                                        "type": "address"
                                    }
                                ],
                                "stateMutability": "nonpayable",
                                "type": "function"
                            },
                            {
                                "inputs": [],
                                "name": "firstOwner",
                                "outputs": [
                                    {
                                        "internalType": "bool",
                                        "name": "",
                                        "type": "bool"
                                    }
                                ],
                                "stateMutability": "view",
                                "type": "function"
                            }
                        ],
                        "devdoc": {
                            "kind": "dev",
                            "methods": {},
                            "version": 1
                        },
                        "userdoc": {
                            "kind": "user",
                            "methods": {},
                            "version": 1
                        }
                    },
                    "settings": {
                        "compilationTarget": {
                            "contracts/test.sol": "Ownership"
                        },
                        "evmVersion": "paris",
                        "libraries": {},
                        "metadata": {
                            "bytecodeHash": "ipfs"
                        },
                        "optimizer": {
                            "enabled": false,
                            "runs": 200
                        },
                        "remappings": []
                    },
                    "sources": {
                        "contracts/test.sol": {
                            "keccak256": "0x2c754ae44155a25816db515ed3acd2d29505d64ac462c463ad312fb074a227dd",
                            "license": "MIT",
                            "urls": [
                                "bzz-raw://0a584ab1a0ef7a1c006a1d79579db1cf1aa3ee02c46d3a5633e31e2d9f0a8428",
                                "dweb:/ipfs/QmUBhPtrWVAiFZojnuFutfUuVTWBttGNN6o3HUmD6ZeKy4"
                            ]
                        }
                    },
                    "version": 1
                }

                ```

            -   Interaction with the ABI needs the contract address to bind the ABI with the bytecode on the blockchain
            -   Ethers has an easy way to achieve this purpose

                ```JS
                require('dotenv').config();
                const ethers = require('ethers');
                const contractABI = require('./ABI.json');

                const provider = new ethers.providers.AlchemyProvider(
                    'goerli',
                    process.env.TESTNET_ALCHEMY_KEY
                );

                // Required in case we need to sign and broadcast requests
                const wallet = new ethers.Wallet(process.env.TESTNET_PRIVATE_KEY, provider);

                async function main() {
                    const counterContract = new ethers.Contract(
                        '0x5F91eCd82b662D645b15Fd7D2e20E5e5701CCB7A',
                        contractABI,
                        wallet
                    );

                    // This function writes to the blockchain and increases the counter of the contract by one
                    await counterContract.inc();

                    // This function reads data from the contract
                    const counter =  await counterContract.get();
                    console.log(counter);
                }

                main();
                ```

        -   Bytecode
            -   There are two types of byte code
                1. Creation time bytecode - is executed only once at deployment, contains the constructor
                2. Run time bytecode - is stored on the blockchain as permanent executable
            -   Transaction receipt
                -   After every transaction executed we get a receipt, when using frontend libraries we can get this receipt by reading the response after executing the transaction
                -   It’s stored in the receipt trie of the block that included the transaction
                -   It contains four pieces of information
                    -   Post-Transaction State
                    -   Cumulative Gas Used
                    -   Set of Logs Created During Execution
                    -   Bloom Filter Composed from the Logs

### Code Examples

-   Signing transactions with different EOAs

    ```Solidity
    // SPDX-License-Identifier: MIT
    pragma solidity 0.7.5;

    contract Contract {
        address owner;
        string public message;

        constructor() {
            owner = msg.sender;
        }

        function modify(string calldata _message) external {
            require(msg.sender != owner, "Owner cannot modify the message!");
            message = _message;
        }
    }
    ```

    ```JS
    // Helper to create a nadom message
    const ranWord = () => {
        const words = ['Duck', 'Fish', 'Bear'];
        const { length } = words;
        return words[Math.round(Math.random() * (length - 1))];
    }

    // Function that calls .modify() using a different signer
    const setMessage = async (contract, signer) => {
        const msg = ranWord();
        return await contract.connect(signer).modify(msg);
    };

    module.exports = setMessage;
    ```

-   Sending ether from frontend

    -   The following example shows the `.deposit()` function call
    -   The function doesn't hav parameters in the contract but is payable

        ```Solidity
        // SPDX-License-Identifier: MIT
        pragma solidity 0.7.5;

        contract Contract {
            function deposit() payable external { }
        }
        ```

    -   When calling it in the frontend we're passing an [overrides object](https://docs.ethers.org/v5/api/contract/contract/#contract-functionsSend) that corresponds to the properties of the transaction

        ```JS
        const deposit = async contract => await contract.deposit({value: parseEther('1')});
        ```
