# Solidity Core

## Multi-Signature Wallets

-   Multi-signature contract: Smart contract that needs multiple signatures from different addresses to execute a transaction
-   Commonly used as wallets
-   Since they have multiple private keys controlling them, they’re safer
    -   EOA are, in certain cases, single points of failure (hacked, phished, lost, etc)
-   Multi-sig wallets are ideal for shared funs, like a DAO treasury

```Solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.4;

contract MultiSig {
    struct Transaction{
        address to;
        uint value;
        bytes data;
        bool executed;
    }

    address[] public owners;
    uint public required;
    uint public transactionCount;
    mapping(uint => Transaction) public transactions;
    mapping(uint => mapping(address => bool)) public confirmations;

    constructor(address[] memory _owners, uint _required) {
        require(_owners.length > 1, "You need more than 1 owner");
        require(_required > 0, "Required should be more than 0");
        require(_required <= _owners.length, "Required should be less than the number of owners");
        owners = _owners;
        required = _required;
    }

    function addTransaction(address _to, uint _value, bytes memory _data) internal returns(uint txId) {
        transactions[transactionCount] = Transaction(_to, _value, _data, false);
        txId = transactionCount;
        transactionCount++;
    }

    function confirmTransaction(uint transactionId) public isOwner {
        confirmations[transactionId][msg.sender] = true;
        if (isConfirmed(transactionId)) {
            executeTransaction(transactionId);
        }
    }

    function getConfirmationsCount(uint transactionId) public view returns(uint count) {
        count = 0;
        for (uint i=0; i<owners.length; i++) {
            if (
                confirmations[transactionId][owners[i]]
            ) count++;
        }
    }

    function submitTransaction(address _to, uint _value, bytes memory _data) external isOwner {
        addTransaction(_to, _value, _data);
        confirmTransaction(transactionCount - 1);
    }

    function isConfirmed(uint transactionId) public view returns(bool) {
        uint count = getConfirmationsCount(transactionId);
        return count >= required;
    }

    function executeTransaction(uint transactionId) internal {
        address to = transactions[transactionId].to;
        uint value = transactions[transactionId].value;
        bytes memory data = transactions[transactionId].data;
        (bool sent, ) = payable(to).call{value: value}(data);
        require(sent, "Transactions failed, try again");
        transactions[transactionId].executed = true;
    }

    receive() external payable {}

    modifier isOwner {
        bool owner = false;
        for (uint i=0; i<owners.length; i++) {
            if (msg.sender == owners[i]) {
                owner = true;
                break;
            }
        }
        require(owner, "Not the owner");
        _;
    }
}
```
