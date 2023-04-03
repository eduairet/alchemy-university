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
