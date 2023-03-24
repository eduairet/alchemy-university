# Accounts in Ethereum

## Externally Owned Accounts (EOA)

-   Similar to Bitcoin private/public key pairs (associated via an Elliptic Curve Digital Signature)
-   40 character hexadecimal string
-   Doesn’t have checksum but [EIP-55](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-55.md) introduced a capitalization scheme that can be validated by wallet software.
-   Has a balance
-   Is active until it has interacted with the blockchain (behavior added at [EIP-161 ](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-161.md) after one user created 19 million accounts)
-   When a user transfer ETH, the value is subtracted from the sender and added to the recipient and updated in the global state
-   EVM has a `BALANCE` operation to check the balance of an account
-   Each account has a Nonce (number we're using once) to keep a count of its transactions (incremented after every transaction)

## Contract Accounts

-   Are created by an EOA by sending a transaction with the byte code of the smart contract
-   Has balance and address
-   It’s not controlled by a private key
-   Make transactions to call functions on the contract
-   Can call another contract
-   Once deployed it cannot be changed but its storage (persistent memory) can be updated through transactions (there’s a possibility to upgrade a contract using proxy contracts)

## Further Reading
