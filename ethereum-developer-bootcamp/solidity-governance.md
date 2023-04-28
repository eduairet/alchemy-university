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

## Delegatecall

-   `<address>.delegatecall`

    -   Similar to `call`

        ```Shell
        # Call

        signedTx = { data: calldata, value: 1.0 } # sends tx to ContractA

        ContractA = { msg: {sender: EOA, value: 1.0}, storage: ContractA.storage }

        b.call{ value: 2.0 }(calldata) # Executes call inside ContractA calling ContractC

        ContractB = { msg: { sender: ContractA, value: 2.0 }, storage: ContractB.storage }
        ```

        ```Shell
        # Delegatecall

        signedTx = { data: calldata, value: 1.0 } # sends tx to ContractA

        ContractA = { msg: {sender: EOA, value: 1.0}, storage: ContractA.storage }

        b.delegatecall(calldata) # Sends the same tx to ContractB with delegatecall

        ContractB = { msg: { sender: EOA, value: 1.0 }, storage: ContractA.storage }
        ```

    -   If you noticed `delegatecall` will send the same transaction from contract A to B, acting like a bridge between both, this is a key behavior for proxy contracts because you use contract A storage with contract B logic
    -   This kind of behavior is optimal for upgrades and gas savings
        -   Minimal Proxy: Save on Gas by deploying one implementation and delegating to it
        -   Upgrade Implementation: Improvements on a previous contract
    -   It’s recommended to always use battle tested contracts in real life projects, check [OpenZeppelin’s](https://docs.openzeppelin.com/upgrades-plugins/1.x/proxies) Proxy Upgrade Pattern
    -   Check this [example](https://github.com/eduairet/proxy-example) and the [EIP-1967](https://eips.ethereum.org/EIPS/eip-1967) for a practical understanding

## Libraries

-   They cannot store storage variables, they don’t have state
-   They cannot be destroyed
-   You can’t send them ETH
-   Their purpose is to share reusable algorithms to avoid re-invent the wheel on every new project
-   Good libraries are audited and battle tested and reduce the chances to get bugs in our code
-   Their functions are commonly pulled to our contracts `bytecode`
    -   It’s common that library functions are `pure` or `view`
    -   If it doesn’t have the keywords mentioned in the previous point, the contract won’t compile

### Deployment

-   Deployed Inline
    -   When they’re marked as `internal` they’re pulled into the smart contract `bytecode`
    -   It’s the most common way to use them
-   Deployed Separately
    -   It will help to keep the contract size low
    -   The on-chain library can be shared with other contracts
    -   It can run code requested by another contract with `delegatecall`
    -   When the contract needs the library to be deployed it will add a placeholder `__$ba528da1e2dc9d528a3d6faf88239359ae$__` that’ll need to be updated later with the library address
        -   This approach requires library linking, check [hardhat docs](https://hardhat.org/hardhat-runner/plugins/nomiclabs-hardhat-ethers#library-linking) and [solidity docs](https://docs.soliditylang.org/en/v0.8.17/using-the-compiler.html?highlight=linking#library-linking) about this topic

### Usage

```Solidity
import "./UIntFunctions.sol";
contract Example {
    function isEven(uint x) public pure returns(bool) {
        return UIntFunctions.isEven(x);
    }
}
```

```Solidity
import "./UIntFunctions.sol";
contract Example {
    using UIntFunctions for uint;
    function isEven(uint x) public pure returns(bool) {
        return x.isEven();
    }
}
```

## Upgradeable Smart Contracts

-   An upgradable contract needs the following contracts to have a secure structure
    -   Proxy contract
        -   It holds the state
        -   It’s an EIP1967 standard proxy contract
        -   Forwards transactions to the implemented contract
    -   Implementation contract
        -   Holds the logic of the contract
        -   Receives transactions from proxy contract via delegatecall
    -   ProxyAdmin contract (optional, good with multi-sigs)
        -   Links the proxy and implementation contracts
        -   Holds authority over the proxy contract (in charge of upgrades)

## The State of Governance

-   Compound protocol example:
    -   Proposal created -> Timelock
    -   Voting active -> Until deadline
    -   Voting ends -> Outcome decide
    -   Timelock finishes -> Proposal queued
    -   Executed
-   Typically used for financial decisions, protocol upgrades, and community decisions
-   As we’ve seen before governance can be handled with [OpenZeppelin](https://docs.openzeppelin.com/contracts/4.x/api/governance)
-   There are websites focused on provide a better user experience with Governance interactions, like [tally.xyz](https://www.tally.xyz/) or [snapshot](https://snapshot.org/)
