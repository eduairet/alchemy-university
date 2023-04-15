# Solidity

## Reference types

-   Reference-based data types (pointers referencing to the actual value of the type) in solidity are:

    -   `arrays`
    -   `strings`
    -   `structs`
    -   `mappings`

-   Using reference types as arguments requires them to take care of the location of the data, solidity provides `memory` `storage` or `calldata`

    -   `memory` temporary data

        ```Solidity
        string name;

        function addString(string memory _name) public {
            name = _name;
        }
        ```

    -   `calldata` external argument data

        ```Solidity
        contract Contract {
            function sum(uint[] calldata arr) external pure returns(uint total) {
                total = 0;
                for (uint i = 0; i < arr.length; i++) {
                    total += arr[i];
                }
            }
        }
        ```

    -   `storage` persistent data

        ```Solidity
        contract Contract {
            uint[3] numbers = [1,2,3];

            function modify() external {
                uint[3] storage storageArray = numbers;
                storageArray[0] = 4; // will modify numbers
                // references point to the same spot in memory
            }
        }
        ```

### Mappings

-   Mappings are hash tables (key, value pair)
    -   Keys are the values stored in memory
    -   It’s deterministic (same key -> same value)
    -   Values are accessed by hashing the key and find its associated value
-   Mappings have search time of `O(1)` which make them very efficient

    -   Structure

        ```Solidity
        mapping(_KeyType => _ValueType) public mappingName;
        ```

    -   Mappings will allow us to change the values associated to an address and to check the value per address easily as well

        ```Solidity
        //SPDX-License-Identifier: MIT

        pragma solidity 0.8.19;

        contract Play {
            mapping (address => uint) private score;
            mapping (uint => bool) public winningGames;
            uint public lastGame;

            constructor() {
                lastGame = 0;
            }

            function game(uint8 guess) public returns(bool won) {
                require(guess <= 10, "You need to provide a number between 0 and 10");
                uint randomNumber = uint(keccak256(abi.encodePacked(block.timestamp))) % 11;
                won = guess == randomNumber;
                lastGame++;
                if (won) {
                    score[msg.sender]++;
                    winningGames[lastGame] = won;
                }
            }

            function scorePerUser(address _user) public view returns(uint) {
                return score[_user];
            }
        }
        ```

    -   For sake of performance is always better to go for a mapping instead of an array, check this example that leads to the same result

        ```Solidity
        mapping(address => bool) students;

        function isStudent(address addr) external view returns(bool) {
            return students[addr];
        }
        ```

        ```Solidity
        address[] students;

        function isStudent(address addr) external view returns(bool) {
            for(uint i = 0; i < students.length; i++) {
                if(students[i] === addr) {
                    return true;
                }
            }
            return false;
        }
        ```

-   Another use cases

    -   You can check the amount of ERC-20 tokens each address has, if someone has voted in a DAO and many other use cases

        ```Solidity
        mapping(address => uint) public balanceOf; // ERC-20s
        mapping(address => bool) public hasVoted; // DAOs
        mapping(uint => bool) public isMember; // DAOs
        mapping(string => uint) public userZipCode; // general info tracking
        ```

-   Mappings can be nested as well

    ```Solidity
    // {address: {gameNumber: uint, result: bool}}
    mapping(address => mapping(uint => bool)) public gameResult;
    ```

-   Or hold structs

    ```Solidity
    struct Game {
        uint number;
        bool result;
        uint8 card;
    }
    mapping(address => Game) public gameResult;
    ```

### Arrays

-   Arrays are collections of elements identified by their index
    -   Fixed size array: Number of elements defined when it’s declared
    -   Dynamic size array: Size changes during runtime
        -   After initialization, memory arrays cannot be resized even if they're declared as dynamic arrays
            ```Solidity
            contract Contract {
                function filterEven(uint[] calldata arr) external pure returns(uint[] memory){
                    uint evens = 0;
                    for (uint i = 0; i < arr.length; i++) {
                        if (arr[i] % 2 == 0) evens++;
                    }
                    uint[] memory filtered = new uint[](evens);
                    uint j = 0;
                    for (uint i = 0; i < arr.length; i++) {
                        if (arr[i] % 2 == 0) {
                            filtered[j] = arr[i];
                            j++;
                        }
                    }
                    return filtered;
                }
            }
            ```
-   Storage arrays

    -   Typically declared as a state variable, and can be fixed or dynamic

        ```Solidity
        contract MyContract {
            uint[] public arr;  // Dynamic size
            uint[] public arr2 = [1, 2, 3];  // Fixed size
            uint[3] public myFixedSizeArr; // Fixed size, start at 3
        }
        ```

-   Arrays come with several properties in Solidity:
    -   `.length` number of elements in the array
    -   Just in dynamic size
        -   `.push()` add element to the end
        -   `.pop()` removes the last element
-   Looping is not recommended since is very costly
-

### Structs

-   Struct allows to create your own data type

    ```Solidity
    struct structName {
    type1 typeName1;
    type2 typeName2;
    }
    ```

-   Example

    ```Solidity
    contract Game {
        struct Player {
            address id;
            uint score;
            uint position;
        }

        Player[] public players;

        function addPlayer(address _id, uint _score,  uint _position) public {
            players.push(Player(_id, _score, players.length));
        }
    }
    ```

-   Structs can be initialized in several ways

    ```Solidity
    enum Directions { Up, Down, Left, Right }

    struct Hero {
        Directions facing;
        uint health;
        bool inAir;
    }

    // Values in the same order
    Hero hero = Hero(Directions.Up, 100, true);

    // Named properties
    Hero hero = Hero({
        facing: Directions.Up,
        health: 100,
        inAir: true
    });
    ```

-   Structs values can be modified by accessing their keys using square brackets

    ```Solidity
    function update(string memory playerName, uint newScore) public {
        players[playerName].score = newScore;
    }
    ```

-   Strucs can be passed as argumemts with the following configuration

    ```Solidity
    // SPDX-License-Identifier: MIT
    pragma solidity 0.8.4;
    pragma experimental ABIEncoderV2;

    contract Contract {
        enum Choices { Yes, No }

        struct Vote {
            Choices choice;
            address voter;
        }

        function createVote(Choices choice) external view returns(Vote memory vote) {
            vote = Vote(choice, msg.sender);
        }
    }
    ```

## Events

-   Logging functionality to write data outside the smart contracts storage in matter of logs
-   Abstraction on top of the EVM's low-level logging functionality, opcodes `LOG0` to `LOG4`
    -   `LOG0`
    -   `LOG1` - 1 topic (signature hash)
    -   `LOG2` - An event with one indexed topic
    -   `LOG3` - An event with two indexed topics
    -   `LOG4` - An event with three indexed topics, max of topics to be included in a transaction
-   `indexed` topics on events can be filtered
-   A topic is a `32 byte hex hashes`
-   They are stored in the transaction receipt and are readable outside the contract (gas efficient)

```Solidity
interface Game {
    // Defined by the event keyword
    event Play(address indexed player, bool indexed result);
}
```

-   Events can be listened outside the blockchain with frontend libraries

    ```JS
    const provider = new ethers.providers.Web3Provider(window.ethereum);
    const contract = new Contract(gameTokenAddress, GAME_ABI, provider);

    contract.on('Play', async (player, result) => {
        console.log(‘Game played. {player, result}');
    });
    ```

-   Emitting events

    ```Solidity
    // SPDX-License-Identifier: MIT
    pragma solidity ^0.8.4;

    contract Collectible {
        address owner;

        event Deployed(address Owner);
        // Owner is not mndatory but is good when reading the ABI
        // event Deployed(address);
        /*
        {
            "indexed": false,
            "internalType": "address",
            "name": "",  // "Owner" when used
            "type": "address"
        },
        */

        constructor() {
            owner = msg.sender;
            emit Deployed(owner);
        }

    }
    ```

### Further reading on Events

-   [Understanding Logs: Deep Dive into `eth_getLogs`](https://docs.alchemy.com/docs/deep-dive-into-eth_getlogs)

## Loops

-   Even though it’s not recommended using them, it’s possible

    ```Soliity
    for(uint i = 0; i < 10; i++) {}
    ```
