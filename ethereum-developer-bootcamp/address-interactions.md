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
        -   `require`
        -   `assert`
