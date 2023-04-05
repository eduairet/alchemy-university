# Calling EOAs

## EOAs in a Smart Contract

-   EOAs can be paid inside smart contracts

```Solidity
contract SpecialNumber {
    address author1;
    address author2;

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
-   `call` method addresses use to send input data or ether to an address
    -   `msg` message
        -   `msg.value` amount of `ETH` received
        -   `msg.sender` who sent the `ETH`
    -   `tx` transaction sent by the EOA
-   `{}` allow to override value and gas, relevant when calling smart contracts
    -   Unspecified `gas` will be sent across all the remaining gas in the transaction
-   `("")` here is where the call data is passed, for example the name of a function
    -   Calls that don’t attend a specific function are empty strings
-   `return value` `(bool success1, )` Will inform us if the transaction was successful
