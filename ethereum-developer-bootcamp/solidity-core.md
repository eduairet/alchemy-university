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

## Inheritance

> #### Inheritance is great to follow the DRY principle:
>
> "Don't repeat yourself" (DRY) is a principle of software development aimed at reducing repetition of software patterns, replacing it with abstractions or using data normalization to avoid redundancy.

-   Create an object that inherits properties from other methods
-   Solidity achieves that through interfaces and contracts

    -   Contracts can take properties from other contracts and make them their own
    -   Inheritance comes with a few keywords

        -   The `is` keyword is how the contract takes a parent contract for its construction

            ```Solidity
            contract A { ... }
            contract B is A { ... }
            ```

        -   `virtual` - Functions that are intended to be overridden
        -   `override` - Keyword to override parent functions

            ```Solidity
            // Contract A
            function foo() public pure virtual returns (uint) {
                return 10;
            }
            // Contract B is A
            function foo() public pure override returns (uint) {
                return 15;
            }
            ```

-   In more a more generic way inheritance allows to create classes built upon existing classes to add new behavior to the child class without affecting the parent class

    ```JS
    // Parent or base class
    class Animal {
      constructor(name) {
        this.name = name;
      }
      present() {
        return `This animal is: ${this.name}.`;
      }
    }

    // Child class
    class Dog extends Animal {
      constructor(name, breed) {
        super(name);
    // name needs to be sent to the parent class with super()
        this.breed = breed; // Child specific property
      }
      show() {
    // Uses this.present inherited from parent
        return `${this.present()} It also is: ${this.breed} dog`;
      }
    }

    let myPet = new Dog("Shib", "Shiba Inu");
    ```

-   In solidity we have different kinds of inheritance

    -   Single inheritance: Inherits the same from parent

        ```Solidity
        contract A { ... }
        contract B is A { ... }
        contract C is B { ... }
        ```

    -   Multi-level inheritance: Multiple levels of parent-child relationship (smart contract inheritance chain)

        ```Solidity
        contract A { ... }
        contract B is A { ... }
        contract C is B { ... }
        ```

    -   Hierarchical inheritance: A contract acts as a base contract for multiple independent derived contracts

        ```Solidity
        contract A { ... }
        contract B is A { ... }
        contract C is A { ... }
        ```

    -   Multiple inheritance: A contract can inherit from several contracts as well

        ```Solidity
        contract A { ... }
        contract B { ... }
        contract C is A, B { ... }
        ```

-   Solidity follows C3 Linearization (like python)

    -   Contracts should follow a specific order while inheriting, starting with the base contract, to the most derived

        ```Solidity
        // SPDX-License-Identifier: GPL-3.0
        pragma solidity >=0.4.0 <0.8.0;

        contract X {}
        contract A is X {}
        // This will not compile
        contract C is A, X {}
        ```

-   Check this example for [multiple inheritance](https://solidity-by-example.org/inheritance/)

-   Besides inheritance Solidity supports other OOP concepts:

    -   Encapsulation
        -   Hiding or allowing access to state variables
        -   Forces some variables to be updated with functions (´external´ ´public´ ´internal´ ´private´)
    -   Polymorphism

        -   Function polymorphism

            -   Using the same function name for different data types

                ```Solidity

                pragma solidity ^0.4.19;
                contract helloFunctionPloymorphism {
                    function getVariableData(int8 data) public pure returns(int8 output) {
                        return data;
                    }
                    function getVariableData(int16 data) public pure returns(int16 output) {
                        return data;
                    }
                }
                ```

        -   Contract polymorphism

            -   Multiple contract instances interchangeably when the contracts are related with inheritance

                ```Solidity
                pragma solidity ^0.4.19;
                contract ParentContract {
                    uint internal simpleInteger;
                    function SetInteger(uint _value) public {
                        simpleInteger = _value;
                    }
                    function GetInteger() public view returns (uint) {
                        return 10;
                    }
                }
                contract ChildContract is ParentContract {
                    function GetInteger() public view returns (uint) {
                        return simpleInteger;
                    }
                }
                ```

    -   Abstraction

        -   Structure defined intended to be used by other contract

            ```Solidity
            pragma solidity ^0.4.19;
            contract abstractHelloWorld {
                function GetValue() public view returns (uint);
            }
            contract HelloWorld is abstractHelloWorld{
                uint private simpleInteger;
                function GetValue() public view returns (uint) {
                    return simpleInteger;
                }
            }
            ```

### [OpenZeppelin](https://www.openzeppelin.com)

-   A company that produces industry-standard smart contracts (stress-tested and audited) that can be inherited in our contracts

    ```Solidity
    // SPDX-License-Identifier: MIT
    pragma solidity ^0.8.9;

    import "@openzeppelin/contracts/token/ERC20/ERC20.sol";

    contract MyToken is ERC20 {
        constructor() ERC20("MyToken", "MTK") {}
    }
    ```

## [ERC-20](https://ethereum.org/en/developers/tutorials/understand-the-erc-20-token-smart-contract/)

-   Is a standard that represents a fungible asset like:
    -   Shares in a company
    -   Reward system points
    -   Voting rights
    -   Cryptocurrency
-   It allows to tokenize basically anything and share a compatible ecosystem
-   ERC-20 smart contract

    -   Tracks owners of ERC-20 tokens with a `mapping`
        ```Solidity
        contract Token {
            mapping(address => uint) public balances;
        }
        ```
    -   It has a common token Interface to allow interoperability between DApps (remember DRY and inheritance)
    -   Basic requirements:

        -   `name` `symbol` `decimals` are all optional fields
        -   `totalSupply` defines the current circulating supply of tokens
        -   `balanceOf` balance of a particular user
        -   `transfer` send tokens between accounts
        -   `approve` `transferFrom` `allowance` methods for other contracts to move your funds (like [Uniswap](https://app.uniswap.org))

            ```Solidity
            pragma solidity 0.8.4;

            interface IERC20 {

                function totalSupply() external view returns (uint256);
                function balanceOf(address account) external view returns (uint256);
                function allowance(address owner, address spender) external view returns (uint256);

                function transfer(address recipient, uint256 amount) external returns (bool);
                function approve(address spender, uint256 amount) external returns (bool);
                function transferFrom(address sender, address recipient, uint256 amount) external returns (bool);


                event Transfer(address indexed from, address indexed to, uint256 value);
                event Approval(address indexed owner, address indexed spender, uint256 value);
            }

            contract MyContract is IERC20 {
                // The contract is ERC-20 compatible!
            }
            ```

-   ERC-20 Data Structures

    1. `balances` - `mapping` of token balances by owner
    2. `allowances` - `mapping` of allowances/delegate spending - indicates the balance that a spender address can use (like [Aave](https://app.aave.com))

-   ERC-20 Methods
    -   `transfer` - Method to send tokens between accounts
    -   `approve-transferFrom` - Third party transfer (like an exchange)
-   [ERC-20 in OpenZeppelin](https://docs.openzeppelin.com/contracts/3.x/erc20)
