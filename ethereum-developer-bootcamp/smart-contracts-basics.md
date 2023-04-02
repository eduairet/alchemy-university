# Smart Contracts Basics

## Intro to Smart Contracts

-   [Nick Szabo](https://www.fon.hum.uva.nl/rob/Courses/InformationInSpeech/CDROM/Literature/LOTwinterschool2006/szabo.best.vwh.net/smart_contracts_2.html) ideated them
    -   " A **smart contract** is a set of promises, specified in digital form, including protocols within which the parties perform on these promises."
    -   " A canonical real-life example, which we might consider to be the primitive ancestor of smart contracts, is the humble vending machine."
        -   Deposit money
        -   Select what you want
        -   Selection dispensed
-   Ethereum approach
    -   Program that runs on Ethereum
    -   Has logic and data (state)
    -   They are compiled to bytecode in order to be EVM-compatible
-   Properties
    -   Permissionless: can be deployed by anyone
    -   Composable: Open “APIs” for everyone to use

## Solidity

-   Object-oriented, high-level language for smart contract implementation
-   Resembles Javascript
-   Design to specifically compile and run on the EVM
-   Properties
    -   Statically-typed
    -   Supports inheritance
    -   Libraries
    -   Complex user-defined types
    -   Many other features
-   It has a very active developer ecosystem [^1]
-   Contract structure
    -   [License](https://spdx.org/licenses/) rules
    -   Solidity version `pragma` using semantic version
        -   ´0.0.0´ - Major version, Minor version, PAtches
    -   Contract scope (contract is like a JavaScript class)
-   Example:

```Solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.4;

address public owner; // no undefined values, starts as 0x0
// Public keyword adds visibility to the variable
// this means that it makes a getter for it.
// State variables can be public, private or internal, not external.

contract MyContract {
    // Called when the contract is deployed
    // used to specify initial state
    constructor(address _owner) {
        // Initial state
        owner = _owner; // _ prevents variable shadowing
    }
}
```

```JS
// Deploying the previous contract would look like this
const myContract = contract.deploy('0x38cE03CF394C349508fBcECf8e2c04c7c66D58CB')
```

-   Numbers
    -   `uint` unsigned
        -   `uint8` 255
        -   `uint16` 65535
        -   `uint256` 10000000000000000000
            -   Default value, same as `uint`
    -   `int` signed
-   Operators
    -   `-` `+` `/` `%` `%`
-   Data Types
    -   `bool`
    -   `string`
    -   `uint` `int`
    -   `bytes`
    -   `enums`
    -   `arrays`
    -   `mappings`
    -   `struct`
    -   `address`
    -   `address payable` - `transfer` and `send`
-   The `address` type
    -   `.balance` - Balance in wei
        -   Smart contract balance `address(this).balance`
    -   `transfer` - Sends Ether to another payable address
-   Smart contract context
    -   Within a smart contract function, there are context variables that are used
        -   `msg.sender` - The sender of the transactiom
        -   `msg.value` - Ether sent on the transaction
        -   `tx.gasLimit` - Transaction gas limit
        -   `block.number` - Current block number
        -   `block.timestamp` - Current block timestamp

## Solidity and the EVM

-   It’s turig-complete
    -   Can perform any logic and mathematical operations
    -   Can perform loops
        -   At lower lever it can execute `JUMP` `JUMPI` which are instructions to handle a loop and also to make it infinite (this won’t happen because of gas)
-   High-level Language that compiles down into byte code, which runs in the Ethereum Virtual Machine
-   It becomes expensive to be inefficient since every computational operation costs, and one impresition can lead to a big amount of Ether
-   Bytecode:

    -   Computers read binary language (zeroes and ones)
        -   `pre-process` - joins all the elements of your code
        -   `process` - joins pieces outside your code (like libraries)
        -   `compile` - the engine checks the code is OK and prepares it for assembly
        -   `assembly` - Makes the code os readable
        -   `linking` - Makes the machine code
            -   If the code comes to this step the machine can now read it
    -   Bytecode is compiled from opcodes, which are a collection of words that tell the computer what to do, they’re often called mnemonic and look like this `PUSH1 0x80 PUSH1 0x40 MSTORE CALLVALUE DUP1 ISZERO PUSH1 0xF`
    -   This opcode is assembled into something like this

        ```
        tag 7               while(i < 5) ...
        JUMPDEST          while(i < 5) ...
        PUSH 5            5
        DUP3              i
        LT                i < 5
        ```

    -   Ethereum reads the bytecode (hex) instead the binary code mentioned above `6080604052348015600f57600080fd5b5060a58061001e600039...`

-   Stack
    -   Is where EVM load/stores runtime data
    -   Data is hold temporarily (local variables, math)
    -   LIFO: Push and pop

## Recommended resources

1. [Mastering Ethereum](https://github.com/ethereumbook/ethereumbook)
    - Chapters 7, 9, 13
2. [Smart Contracts: Building Blocks for Digital Markets](https://www.fon.hum.uva.nl/rob/Courses/InformationInSpeech/CDROM/Literature/LOTwinterschool2006/szabo.best.vwh.net/smart_contracts_2.html)
3. [The Idea of Smart Contracts](https://www.fon.hum.uva.nl/rob/Courses/InformationInSpeech/CDROM/Literature/LOTwinterschool2006/szabo.best.vwh.net/idea.html)
4. [Solidity by example](https://solidity-by-example.org)
5. [ethersplay](https://github.com/crytic/ethersplay)

[^1]: There are many other languages like Vyper, which is more like python
