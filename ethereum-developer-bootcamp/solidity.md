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

-   Another use cases

-   You can check the amount of ERC-20 tokens each address has

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
