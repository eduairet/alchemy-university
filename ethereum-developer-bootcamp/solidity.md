# Solidity

## Mappings

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
