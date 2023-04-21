# Address Interactions

## Calling EOAs

### EOAs in a Smart Contract

-   EOAs can be paid inside smart contracts

```Solidity
contract SpecialNumber {
    address author1;
    address author2;

    // payable is needed in latest versions of Solidity to allow the contract to receive ETH
    // since nonpayable is the default value for every function
    receive() external payable {
        // msg.value is passed to this contract
        uint totalValue = msg.value;

        // make a call to author1 sending half of the ether
        (bool success1, ) = author1.call{ value: totalValue / 2 }("");
        require(success1);

        // make a call to author2 sending half of the ether
        (bool success2, ) = author2.call{ value: totalValue / 2 }("");
        require(success2);
    }
}
```

-   Payable functions can receive `ETH`
-   `receive` is a special function invoked when the contract receives money

    -   It is the function that runs when a contract receives `ETH` without any calldata
    -   Must be `external` and `payable`
    -   It cannot receive arguments
    -   It cannot return anything

-   Fallback function
    -   Is an external function that will be triggered when there's an error in the calldata
    -   Cannot accept any arguments or return any values
    -   Doesn't need to be payable, but if payable it can replace `receive`
        -   Useful when you receive data and value, like signature mistakes
-   `call` method addresses use to send input data or ether to an address
    -   `msg` message
        -   `msg.data (bytes)` - the complete calldata
        -   `msg.sender (address)` - the address sending the message
        -   `msg.sig (bytes4)` - the targeted function signature
            -   First four bytes of the `keccak256` hash of the function signature
        -   `msg.value (uint)` - the amount of wei sent
    -   `tx` transaction sent by the EOA
-   `{}` allow to override value and gas, relevant when calling smart contracts
    -   Unspecified `gas` will be sent across all the remaining gas in the transaction
-   `("")` here is where the call data is passed, for example the name of a function
    -   Calls that don’t attend a specific function are empty strings
-   `return value` `(bool success1, )` Will inform us if the transaction was successful

## Reverting Transactions

-   Reverting means making the transaction like never happen
    -   Removes all the state changes but it can be included in a block (gas is actually used)
-   This action is triggered by the opcode `REVERT`
    -   We can control when to revert the transactions by using
        -   `revert`
            -   Throws error
            -   Can be used as a statement `if(!someBool) revert;`
            -   Older versions of Solidity used it as a function ``if(!someBool) revert(message);`
        -   `require`
            -   Throws error
            -   Good for user input
        -   `assert`
            -   Raises an exception
            -   Good for invariants

## Modifiers

```Solidity

function someFunc() public view someFuncModifier {
    // do something
}

modifier someFuncModifier {
    // do something before the function
    _; // Here the function is executed
    // do something after the function
}
```

## Calling Contract Addresses

-   You can send data from a contract to another contract

    ```Solidity
    contract A {
        function setValueOnB(address b) external {
            // Calling contract B storeValue() with data manually encoded
            (bool s, ) = b.call(
                abi.encodeWithSignature("storeValue(uint256)", 22)
                // this will send the 4 first bytes of the data
                // this method needs full types like uint256,
                // won't work using aliases like uint
                // it only needs the name of the function and parameters types
                // separated by comma with no space
                // receiveData(uint256,uint256)
            );
            require(s);
        }
    }
    contract B {
        uint x;
        function storeValue(uint256 _x) external {
            x = _x; // 22
        }
    }
    ```

    ```Solidity
    contract A {
        function setValueOnB(address b) external {
            // Calling contract B storeValue() with definition value
            B(b).storeValue(22); // B is the same from previous block
        }
    }
    ```

    -   When using these kind of actions there's a chance that something get's wrong, if we're sending wrong calldata to other contrat this will call it's fallback function if there's one

-   If you don't have acces to the contract where you're calling the function you can use an interface (functions you can access from the outside, like an API, or the UI of a website) to give solidity enough information to perform the call

    ```Solidity
    interface B {
        function storeValue(uint256) external;
    }
    ```

-   If you want to send data from contract to contract you can do it this way

    ```Solidity
    // SPDX-License-Identifier: MIT
    pragma solidity ^0.8.4;

    contract Sidekick {
        function relay(address hero, bytes memory data) external {
            // send all of the data as calldata to the hero
            (bool s, ) = hero.call(data);
            require(s);
        }
    }
    ```
