# Contract Examples

## Escrow

-   Buyer seller contract where an escrow holds the funds until all the requirements are satisfied
-   This kind of treatment assumes that the escrow agent is honest, in this case the logic inside our contract, that can also designate an agent or a DAO to approve transactions

    ```Solidity
    // SPDX-License-Identifier: MIT
    pragma solidity 0.8.4;

    contract Escrow {
        address public depositor;
        address public beneficiary;
        address public arbiter;
        bool public isApproved;

        event Approved(uint BalanceSent);

        constructor(address _arbiter, address _beneficiary) payable {
            depositor = msg.sender;
            arbiter = _arbiter;
            beneficiary = _beneficiary;
            isApproved = false;
        }

        function approve() external payable isArbiter {
            uint amount = address(this).balance;
            (bool success, ) = beneficiary.call{value: amount}("");
            require(success, "The approval couldn't be completed!");
            isApproved = true;
            emit Approved(amount);
        }

        modifier isArbiter {
            require(msg.sender == arbiter);
            _;
        }
    }
    ```
