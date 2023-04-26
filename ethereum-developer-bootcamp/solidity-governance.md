# Solidity Governance

## Storage Lots

-   A storage slot is the place where some value is stored
-   Variables are stored anywhere between `0x0 0xffff...`
-   Empty slots return `0`
-   Endian-ness in Ethereum
    -   `Big-endian` (last byte of binary representation of data type is stored first `———>`) used for bytes and string types
    -   `Little-endian` (first byte of binary representation of data type is stored first `<———`) used for every other type of variable
-   Even if a variable doesn’t have the `public` keyword we can check its position in storage, a handful way to do it is by using the `eth_getStorageAt`
-   Value types can be found by it’s storage slot `0x1` and reference types have a more complex storage , for example mappings store all of their entries with the following formula `keccak256(key + baseSlot)`

    ```Solidity
    // SPDX-License-Identifier: MIT
    pragma solidity 0.8.19;

    contract Storage {
        uint public x = 100; // Slot -> 0x0
        uint public y = 200; // Slot -> 0x1

        // keccak256(key + baseSlot) - Collision resistant
        mapping(uint => uint) public someMapping; // Base slot -> 0x2

        constructor() {
            someMapping[0] = 12; // keccak256(0 + 0x2)
            someMapping[5] = 5; // keccak256(5 + 0x2)
            someMapping[100] = 199; // keccak256(100 + 0x2)
        }
    }
    ```

    ```TS
    import { ethers } from 'hardhat';
    const { provider, utils } = ethers,
        { keccak256, hexZeroPad, hexlify } = utils,
        addr = '0x0165878A594ca255338adfa4d48449f69242Eb8F';

    async function main(): Promise<void> {
        /**
        * You can use the ethers abstraction of eth_getStorageAt
        * to get values at certain storage slot
        */

        // uint public x = 100; // Slot -> 0x0
        await getStorageValue(addr, '0x0');
        // 0x0000000000000000000000000000000000000000000000000000000000000064
        // 100

        // uint public y = 100; // Slot -> 0x1
        await getStorageValue(addr, '0x1');
        // 0x00000000000000000000000000000000000000000000000000000000000000c8
        // 200

        // mapping(uint => uint) public someMapping; // Base slot -> 0x2
        await getStorageRefValue(addr, 0, 0x2);
        // 0x000000000000000000000000000000000000000000000000000000000000000c
        // 12
        await getStorageRefValue(addr, 5, 0x2);
        // 0x0000000000000000000000000000000000000000000000000000000000000005
        // 5
        await getStorageRefValue(addr, 100, 0x2);
        // 0x00000000000000000000000000000000000000000000000000000000000000c7
        // 199
    }

    async function getStorageValue(addr: string, slot: string): Promise<void> {
        const value: string = await provider.getStorageAt(addr, slot);
        console.log(value); // Hex value
        console.log(parseInt(value)); // Actual value
    }

    async function getStorageRefValue(
        addr: string,
        _key: number,
        _baseSlot: number
    ): Promise<void> {
        const key: string = hexZeroPad(hexlify(_key), 32),
            baseSlot: string = hexZeroPad(hexlify(_baseSlot), 32).slice(2),
            slot: string = keccak256(key + baseSlot);
        await getStorageValue(addr, slot);
    }

    main()
        .then(() => process.exit(0))
        .catch(err => {
            console.log(err);
            process.exit(1);
        });
    ```

-   For more complex storage check [`OpenZeppelin`](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/StorageSlot.sol)
