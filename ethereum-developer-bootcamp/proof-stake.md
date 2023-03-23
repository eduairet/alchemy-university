# Proof of Stake

-   On September 2022 Ethereum transitioned from Proof of Work to Proof of Stake during the so called event The Merge
    -   More security
    -   Less energy consumption
    -   More scalability
-   No mining hardware resources are required
-   To become a validator you need:
    -   32 ETH deposited in a validation contract
        -   Collateral against bad actors
-   The network selects random validators every 12 seconds, and the rest of the validators verify the proposed block

## How PoS affects Ethereum Development

-   Finality blocks
    -   `earliest` -> first block
    -   `latest` -> most recent block
    -   `pending` -> sample next block (not mined yet)
    -   With PoS Ethereum added 2 new levels of finalitu
        -   `safe` -> most recent crypto-economically secure block (unlikely to be re-orged)
        -   `finalized` -> most recent crypto-economically secure block accepted by >⅔ of the validators (very unlikely to be re-orged)
    -   `earliest`≤`finalized`≤`safe`≤`latest`≤`pending`
    -   Finality is very important when we are requesting block data from Ethereum

## Extra resources

-   [Proof of Stake](https://ethereum.org/en/developers/docs/consensus-mechanisms/pos/)
-   [The Merge](https://www.alchemy.com/the-merge)
-   [The Ethereum Developer Guide to The Merge](https://docs.alchemy.com/reference/ethereum-developer-guide-to-the-merge)
