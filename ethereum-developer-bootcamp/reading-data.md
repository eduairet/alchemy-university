# Reading Data from Ethereum

## Intro to JSON-RPC

-   We communicate with Ethereum with JSON-RPC
-   JSON-RPC is the bridge we use to connect any DApp to the network

### Ethereum Clients

-   To run an Ethereum node you need to run an Ethereum client as well, here are some of the most popular clients:
    -   [geth](https://github.com/ethereum/go-ethereum)
    -   [erigon](https://github.com/ledgerwatch/erigon)
    -   [nethermind](https://github.com/NethermindEth/nethermind)
-   After the merge is required to run two clients at the same time, the execution clients (ETH1, mentioned in the previous point) and the consensus clients (ETH2)

### JSON-RPC

-   Remote Procedure Call - Another API standard
-   Similar to REST, useful for CRUD
    -   You send a request from the DApp
    -   You get responses from the Ethereum Node acting as a listening server
-   Transports data from the protocol in the form of JSON
-   There are stablished methods to interact with the network, check them [here](https://docs.alchemy.com/reference/ethereum-api-faq#what-methods-does-alchemy-support-for-the-ethereum-api)
-   Responses use hexadecimal numbers to avoid discrepancies caused by decimal numbers
-   Request example

    -   Version is always 2.0
    -   Id is always 0 unless you’re batching requests

    ```JSON
    {
        "jsonrpc": "2.0",
        "id": 0,
        "method": "eth_chainId"
    }
    ```

    -   Response example

    ```JSON
    {
        "jsonrpc": "2.0",
        "id": 0,
        "result": "0x1"
    }
    ```
