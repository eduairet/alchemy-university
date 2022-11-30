# Blockchain and Crypto

## Purpose

-   Decentralized the code and agree on the state of the data through consensus
-   Evolution
    -   2008 -> Bitcoin -> Value transfer updates the state -> Decentralized money
    -   2015 -> Ethereum -> Smart contracts update the state -> Descentralized computer

## Cryptography

-   Blockchains use hashing functions to secure the data, these functions return a fixed size output (32 bytes with SHA256) and they are:
    -   Deterministic
        -   Same input will always give the same result
            ```
            Always the same using the same algorithm
            Alchemy -> 2805fc43d2b5517d922a021e11c80de40be40976cabbaa4b2ce11dcf9537490a
            ```
    -   Pseudorandom
        -   Small changes in the input completely change the output
            ```
            Alchemy -> 2805fc43d2b5517d922a021e11c80de40be40976cabbaa4b2ce11dcf9537490a
            Alchemi -> 0cde58de65442620892f4fc278446e727c6f01f10070d9f7e1d19ab5edb7733d
            ```
    -   One-way
        -   Input is almost imposible to get from output
            ```
            You may need to try all the posible combinations to find it
            2805fc43d2b5517d922a021e11c80de40be40976cabbaa4b2ce11dcf9537490a -> ????
            ```
    -   Fast to Compute
        -   The algorithm executes fast
            ```
            It just needs miliseconds to return the result
            Alchemy -> 2805fc43d2b5517d922a021e11c80de40be40976cabbaa4b2ce11dcf9537490a
            ```
    -   Collision-resistant
        -   Almost impossible to be broken
