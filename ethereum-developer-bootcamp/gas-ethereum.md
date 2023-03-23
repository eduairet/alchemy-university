# Gas on Ethereum

## [EIP-1559](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-1559.md)

-   Changes with every block
-   Is unpredictable and sometimes incredibly high
-   2021 - EIP proposed a change for gas price calculation -> LondonFork -> EIP-1559 - Predictable transaction fees: - `baseFee + tipCalculation` - Fees reduced and shorter transaction times: - Transparent cost - More efficient block filling - Control of Ethereum supply: - `Base fee burned == < ETH used` - Gas structure - `gas` - Amount of computation and resources to complete a transaction - `gasPrice` - Amount the user wants to pay for the transaction - `baseFee` - Minumum fee - `maxPriorityFee` - Optional fee to prioritize the transaction over others - `gasLimit` - Maximum amount of gas to complete the operation - Block size changes according to the network’s traffic (15M - 30M gas) - Block base fee calculation looks at the previous block's capacity

    | Last Block Capacity  | Change of Base Fee                                                                                                                          |
    | -------------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
    | Exactly 50% full     | No change                                                                                                                                   |
    | 100% full            | 12.5% increase                                                                                                                              |
    | Between 50% and 100% | < 12.5% Increase                                                                                                                            |
    | Between 0% and 50%   | < 12.5% Decrease                                                                                                                            |
    | 0% / Empty           | 12.5% Decrease - ETH burning - The ETH is sent to a burner wallet (can receive but not send) - This controls spam transactions and gas fees |

-   Important denominations used in EIP-1559 gas
    -   `Wei` - ETH value: `10**-18` o `1 / (10**18)` - Usage: Technical implementations
    -   `Gwei` (giga-wei) - ETH value: `10**-9` o `1 / (10**9)` - Usage: Human-readable gas fees

## Denominations of Ether

| Unit                | Wei Value | Wei                       |
| ------------------- | --------- | ------------------------- |
| wei                 | 1 wei     | 1                         |
| Kwei (babbage)      | 1e3 wei   | 1,000                     |
| Mwei (lovelace)     | 1e6 wei   | 1,000,000                 |
| Gwei (shannon)      | 1e9 wei   | 1,000,000,000             |
| microether (szabo)  | 1e12 wei  | 1,000,000,000,000         |
| milliether (finney) | 1e15 wei  | 1,000,000,000,000,000     |
| ether               | 1e18 wei  | 1,000,000,000,000,000,000 |

> Gwei is not the same Gwei mentioned in the EIP-1559 section

## How is the price of gas set?

-   Every block has a maximum amount of gas that can be use within it
    -   30M gas limit | 15M gas target
-   Price is determined by demand for block space
    -   Network sets a `baseFee` - Ideally 15M, but really more or less
    -   Block above the target - `baseFee` increased
    -   Block below the target - `baseFee` reduced

## Base Fee

-   The `baseFee` allows to predict future prices
-   The `baseFee` gets burned
    -   Prevents miner to not pay the baseFee
    -   Makes ether deflationary

## Setting gas

-   When you set a transaction you set the `maxFee` (maximum amount you’re willing to pay)
-   Transaction will only ever use `baseFee`
-   `maxFee - baseFee` is set by the user
-   [Gas calculator algorithm example](https://docs.alchemy.com/docs/how-to-build-a-gas-fee-estimator-using-eip-1559)

    -   `baseFee` is determined by the protocol, we need to set what to bid for the `priorityFee`
    -   The relationship between `gasUsedRatio` and `baseFeePerGas` is what EIP 1559 brought with the London Fork
        -   Block < Half full -> Network decreases the `baseFee`
        -   Block > Half full -> Network increases the `baseFee`
    -   Simple example:

        ```JS
        /*
        *Call to eth_feeHistory API
        */
        // Look for the last four blocks
        const historicalBlocks = 4;
        // We need to know how full was the block and how much transactions bid to enter
        web3.eth.getFeeHistory(historicalBlocks, "pending", [10]).then((feeHistory) => {
            const blocks = formatFeeHistory(feeHistory, false);
            const firstPercentialPriorityFees = blocks.map(b => b.priorityFeePerGas[0]);
            const sum = firstPercentialPriorityFees.reduce((a, v) => a + v);
            // The 10th percentile could be a good shot for our transaction to be included
            console.log("Manual estimate", sum/firstPercentialPriorityFees.length);
        });

        function formatFeeHistory(result, includePending) {
        let blockNum = result.oldestBlock;
        let index = 0;
        const blocks = [];
        while (blockNum < result.oldestBlock + historicalBlocks) {
            blocks.push({
            number: blockNum,
            baseFeePerGas: Number(result.baseFeePerGas[index]),
            gasUsedRatio: Number(result.gasUsedRatio[index]),
            priorityFeePerGas: result.reward[index].map(x => Number(x)),
            });
            blockNum += 1;
            index += 1;
        }
        if (includePending) {
            blocks.push({
            number: "pending",
            baseFeePerGas: Number(result.baseFeePerGas[historicalBlocks]),
            gasUsedRatio: NaN,
            priorityFeePerGas: [], // the 25th, 50th, and 75th percentiles of priority fees submitted
            });
        }
        return blocks;
        }
        ```

    -   Full estimate example

        ```JS
        const historicalBlocks = 20;
        web3.eth.getFeeHistory(historicalBlocks, "pending", [1]).then((feeHistory) => {
            const blocks = formatFeeHistory(feeHistory, false);
            const firstPercentialPriorityFees = blocks.map(b => b.priorityFeePerGas[0]);
            const sum = firstPercentialPriorityFees.reduce((a, v) => a + v);
            const priorityFeePerGasEstimate = Math.round(sum/firstPercentialPriorityFees.length);
            web3.eth.getBlock("pending").then((block) => {
                // We add the tip on top of the base fee estimation
                const maxFeePerGasEstimate = Number(block.baseFeePerGas) + priorityFeePerGasEstimate;
                // Estimation in Wei (transform to Gwei for the user)
                console.log("Manual estimate:", maxFeePerGasEstimate);
            });
        });
        ```

    -   Estimation using all the percentiles

        ```JS
        const historicalBlocks = 20;
        web3.eth.getFeeHistory(historicalBlocks, "pending", [1]).then((feeHistory) => {
            const blocks = formatFeeHistory(feeHistory, false);
            const firstPercentialPriorityFees = blocks.map(b => b.priorityFeePerGas[0]);
            const sum = firstPercentialPriorityFees.reduce((a, v) => a + v);
            const priorityFeePerGasEstimate = Math.round(sum/firstPercentialPriorityFees.length);
            web3.eth.getBlock("pending").then((block) => {
                const maxFeePerGasEstimate = Number(block.baseFeePerGas) + priorityFeePerGasEstimate;
                console.log("Manual estimate:", maxFeePerGasEstimate);
            });
        });
        ```

    -   The previous algorithms are a good way to show how to set the price, nevertheless they are not practical for applications (thousands of transactions per second)
    -   It’s a better practice to use an oracle for this assignment as `Geth` does
    -   Miners payment
        -   They receive the tip of the transaction
        -   `maxPriorityFee  = maxFee + tip`
