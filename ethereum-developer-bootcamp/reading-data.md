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

Alchemy

## Making requests with a Node as a service

-   With every Node as a service you’ll need an API key to access the endpoint of the node you’re looking for
-   With [Alchemy](https://www.alchemy.com) you can create an API key for different networks
-   JSON-RPC is a way to send and receive JSON between the client and the server (in this case our Alchemy node)
-   Here’s an example of a command line call

```BASH
#Request
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":83}' https://eth-mainnet.alchemyapi.io/v2/ALCHEMY_API_KEY
```

```BASH
#Response
{"jsonrpc":"2.0","id":83,"result":"0x101d6e4"}
```

-   And an example using node and JavaScript

```JS
// Use of the Axios Library for the requests
const axios = require('axios');
// dotenv for environmental variables
require('dotenv').config();

// Node as a service URL provided in alchemy.com dashboard
const ALCHEMY_URL = process.env.ALCHEMY_URL;
// Request to get block 46147 data
axios
    .post(ALCHEMY_URL, {
        jsonrpc: '2.0',
        id: 1,
        method: 'eth_getBlockByNumber',
        params: [
            '0xb443', // block 46147
            false, // retrieve the full transaction object in transactions array
        ],
    })
    .then(response => {
        console.log(response.data.result);
    });

```

```JS
// Response
{
  number: '0xb443',
  hash: '0x4e3a3754410177e6937ef1f84bba68ea139e8d1a2258c5f85db9f1cd715a1bdd',
  transactions: [
    '0x5c504ed432cb51138bcf09aa5e8a410dd4a1e204ef84bfed1be16dfba1b22060'
  ],
  difficulty: '0x153886c1bbd',
  extraData: '0x657468706f6f6c2e6f7267',
  gasLimit: '0x520b',
  gasUsed: '0x5208',
  logsBloom: '0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000',
  miner: '0xe6a7a1d47ff21b6321162aea7c6cb457d5476bca',
  mixHash: '0xb48c515a9dde8d346c3337ea520aa995a4738bb595495506125449c1149d6cf4',
  nonce: '0xba4f8ecd18aab215',
  parentHash: '0x5a41d0e66b4120775176c09fcf39e7c0520517a13d2b57b18d33d342df038bfc',
  receiptsRoot: '0xfe2bf2a941abf41d72637e5b91750332a30283efd40c424dc522b77e6f0ed8c4',
  sha3Uncles: '0x1dcc4de8dec75d7aab85b567b6ccd41ad312451b948a7413f0a142fd40d49347',
  size: '0x27a',
  stateRoot: '0x0e0df2706b0a4fb8bd08c9246d472abbe850af446405d9eba1db41db18b4a169',
  timestamp: '0x55c42659',
  totalDifficulty: '0x97a50222edba99',
  transactionsRoot: '0x4513310fcb9f6f616972a3b948dc5d547f280849a87ebb5af0191f98b87be598',
  uncles: []
}
```

## Ethereum Nodes

-   Maintain integrity and data on the network
-   Type of nodes
    -   Full nodes
        -   Have a full copy of the blockchain locally
        -   Store and validate blocks and transition
-   Problems on Web3
    -   Scalability
        -   Actual size of Ethereum makes difficult to have your own node (weeks downloading)
        -   If a node goes down, their users won’t be able to make more requests
    -   Reliability
        -   Even nodes with 1.5 reliability have 1.5 days of downtime every month
    -   Data consistency
        -   Latest block can vary between nodes
        -   Load balancing can return inconsistent data
            -   Can be reflected in some of the nodes involved in balancing and not in some others
-   Node providers as Alchemy solve the previous problems and offer more consistent load balancing under the hood, with a so called Supernode

### Data Storage

-   Ethereum use Merkle Patricia Tries to store data
-   Data can be proven as valid without retrieving all of it thanks to its root
    -   State trie: global state of the network
    -   Storage trie: account storage
    -   Transactions trie: all transactions of a block
    -   Receipts trie: data and logs from every transaction
