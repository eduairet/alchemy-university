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

## Party

```Solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.4;

contract Party {
    uint public cover;
    address[] public guests;

    event NewGuest(address indexed Guest, uint indexed Cover);
    event VenuePaid(address indexed Venue, uint indexed Bill);

	constructor(uint _cover) {
        cover = _cover;
    }

    function rsvp() external payable {
        bool registered = false;
        for (uint i=0; i<guests.length; i++) {
            if (guests[i] == msg.sender) {
                registered = true;
                break;
            }
        }
        require(!registered, "Already registered!");
        require(msg.value == cover, "Please pay the cover's exact amount");
        guests.push(msg.sender);
        emit NewGuest(msg.sender, msg.value);
    }

    function payBill(address payable venue, uint bill) external {
        require(address(this).balance >= bill, "The contract doesn't have enough money to pay the bill");
        (bool sent, ) = venue.call{value: bill}("");
        require(sent, "Payment failed");
        if (address(this).balance > guests.length) {
            uint share = address(this).balance / guests.length;
            for (uint i=0; i<guests.length; i++) {
                (bool shareSent, ) = payable(guests[i]).call{value: share}("");
                require(shareSent, "Unable to pay the remai");
            }
        }
        emit VenuePaid(venue, bill);
    }
}
```

## Dead Man's Switch

```Solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.4;

contract Switch {
    address owner;
    address recipient;
    uint inactivity;

    constructor(address _recipient) payable {
        owner = msg.sender;
        recipient = _recipient;
        inactivity = block.timestamp;
    }

    function withdraw() external isRecipient {
        require(
            block.timestamp - inactivity >= 52 weeks,
            "Not more than 52 weeks of inactivity"
        );
        uint balance = address(this).balance;
        (bool sent, ) = recipient.call{value: balance}("");
        require(sent, "Couldn't withdraw funds");
    }

    function ping() external isOwner{
        inactivity = block.timestamp;
    }

    modifier isOwner {
        require(msg.sender == owner, "Not the owner");
        _;
    }

    modifier isRecipient {
        require(msg.sender == recipient, "Not the recipient");
        _;
    }
}
```

## Hackathon Winner

```Solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.4;

contract Hackathon {
    struct Project {
        string title;
        uint[] ratings;
    }

    Project[] projects;

    function newProject(string calldata _title) external {
        // creates a new project with a title and an empty ratings array
        projects.push(Project(_title, new uint[](0)));
    }

    function rate(uint _idx, uint _rating) external {
        // rates a project by its index
        projects[_idx].ratings.push(_rating);
    }

    function findWinner() external view returns(Project memory winner) {
        (uint maxi, uint maxavg) = (0, 0);
        for (uint i=0; i<projects.length; i++) {
            uint avg = 0;
            for (uint j=0; j<projects[i].ratings.length; j++) {
                avg += projects[i].ratings[j];
            }
            avg = avg / projects[i].ratings.length;
            if (avg > maxavg) {
                (maxi, maxavg) = (i, avg);
            }
        }
        winner = projects[maxi];
    }
}
```

## Ownable and Transferable

-   `BaseContracts.sol`

    ```Solidity
    // SPDX-License-Identifier: MIT
    pragma solidity 0.8.4;

    contract Ownable {
        address public owner;
        constructor() {
            owner = msg.sender;
        }
        modifier onlyOwner {
            require(owner == msg.sender, "Not the owner");
            _;
        }
    }

    contract Transferable is Ownable {
        function transfer(address newOwner) external virtual onlyOwner {
            owner = newOwner;
        }
    }
    ```

-   `Collectible.sol`

    ```Solidity
    // SPDX-License-Identifier: MIT
    pragma solidity 0.8.4;

    import "./BaseContracts.sol";

    contract Collectible is Ownable, Transferable {
        uint public price;

        function markPrice(uint _price) external onlyOwner {
            price = _price;
        }
    }
    ```
