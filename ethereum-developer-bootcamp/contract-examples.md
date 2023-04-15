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

## Votes

```Solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.4;

contract Contract {
	enum Choices { Yes, No }

	struct Vote {
		Choices choice;
		address voter;
	}

	Vote[] public votes;

	function createVote(Choices choice) external {
		if (findVote(msg.sender).voter == address(0))
			votes.push(Vote(choice, msg.sender));
		else revert("Already voted!");
	}

	function changeVote(Choices _choice) external {
		if (findVote(msg.sender).voter != address(0)) {
			Vote[] storage _votes = votes;
			for (uint i=0; i<votes.length; i++) {
				if (votes[i].voter == msg.sender) {
					_votes[i].choice = _choice;
					break;
				}
			}
		} else {
			revert("Haven't voted!");
		}
	}

	function hasVoted(address _voter) external view returns(bool) {
		if (findVote(_voter).voter == address(0)) return false;
		return true;
	}

	function findChoice(address _voter) external view returns(Choices) {
		return findVote(_voter).choice;
	}

	function findVote(address voter) internal view returns(Vote memory vote) {
		vote = Vote(Choices(0), address(0));
		for (uint i=0; i<votes.length; i++) {
			if (votes[i].voter == voter) {
				vote = votes[i];
				break;
			}
		}
	}
}
```
