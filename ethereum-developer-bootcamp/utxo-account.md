# UTXO & Account Based Model

## Transactions on decentralized networks:

-   Sender, Amount, Recipient, and a collision resistant authentication like asymmetric cryptography signatures

## UTXOs

-   Bitcoin uses UTXOs (Unspent Transaction Outputs)
-   All coins are not the same, which prevents double spend
-   If you consume part of a UTXO the remainder generates a new one with the leftover amount
-   Every bitcoin UTXO is non fungible
-   When you receive bitcoin you receive a UTXO, you receive a token with a specific amount
-   Each one has a script associated to it and it can only be accessed by the owner
-   Transaction format (it uses locked time)
    -   Input: Prev txn ID, Index, scriptSig
    -   Output: Value, scriptPubKey
-   Consensus rules:
    -   Sum(inputs) >= Sum(outputs)
        -   Except for the coinbase transaction (0)
        -   It should be less because of the fees
    -   Every input
        -   Eval(scriptSig + scriptPubKey) == true
    -   Output not with SPENT yet
-   UTXOs are good for networks like bitcoin that don’t rely on complex state changes or state at all

```JS
// UTXO example
class TXO {
    constructor(owner, amount) {
        this.owner = owner;
        this.amount = amount;
        this.spent = false;
    }
    // When the UTXO is spent it’s set to false
    spend() {
        this.spent = true;
    }
}

module.exports = TXO;
```

```JS
// Transaction example
class Transaction {
    constructor(iTXOs, oTXOs) {
        // Input TXOs
        this.iTXOs = iTXOs;
        // Output TXOs
        this.oTXOs = oTXOs;
    }
    execute() {
        // Check for spent txs in the input
        if (this.iTXOs.find(el => el.spent === true))
            throw new Error('There are spent TXOs in your input');
        // Check the input is less than the output
        const inputTotal = this.iTXOs.reduce((a, txo) => a + txo.amount, 1);
        const outputTotal = this.oTXOs.reduce((a, txo) => a + txo.amount, 1);
        if (inputTotal < outputTotal)
            throw new Error('The output is greater than the input');
        // If everything goes well the tx is executed
        // The input UTXOs are spent to avoid double spend
        this.iTXOs.forEach(txo => txo.spend());
        // And the output UTXOs become the new TXOs to spend
        // (no action needed since there are already not spent)
        /*
        And finally the transaction fee will be the remainder
        between the input and the output
        */
        this.fee = inputTotal - outputTotal;
    }
}

module.exports = Transaction;
```

## Account-based Model

-   Each account has a balance
-   Kind of a bank account
-   Keeps track of each account state
-   Tx is valid if the amount is less than the account balance
-   Transactions change the state of an account
-   There’s a nonce to prevent replay attacks

## Further reading on UTXOs

-   [Bitcoin genesis block](https://www.blockchain.com/explorer/blocks/btc/0)
    -   UTXO never spent
    -   Its tx hash has two extra leading zeroes
    -   It’s coinbase has the hash of the text: The Times 03/Jan/2009 Chancellor on brink of second bailout for banks
    -   50BTC reward sent to an unknown address
    -   Its timestamp is tricky since doesn’t match the 10 minutes block mining time
-   [Genesis block details](https://en.bitcoin.it/wiki/Genesis_block)
    -   First block of a blockchain
    -   Block 0 now, previously Block 1
    -   Almost always hardcoded
    -   Doesn’t reference a previous block
    -   It cannot be spent like in bitcoin
    -   The `scriptPubKey` is called the ‘Witness Script’ or ‘Locking Script’
    -   Each ‘Locking Script’ has an ‘Unlocking Script’ that allows the UTXO to be spent
    -   The unlock needs a signature that verifies the owner and prevents double spend
-   [Bitcoin Script Language](https://en.bitcoin.it/wiki/Script)
    -   Not turing complete, no loops
    -   Processed left to right
    -   Simple, prevents no denial or service attacks
    -   A list of function-like operation codes
    -   Takes a number from the stack of codes and operates with them
    -   Basically it transfers BTC from one person to other
        -   Spender must provide a public key
        -   And a signature to prove ownership
    -   With scripting the requirements for a transaction can be changed, it can have an order to not require private keys for example
    -   When scripting is important to check the OP-Codes from documentation and examples
-   [Script](https://bitcoin.stackexchange.com/questions/29754/history-behind-the-scripting-language-in-bitcoin/29763#29763)
    -   Probably created after the whole bitcoin idea, it wasn’t even properly tested by Satoshi
    -   An example is the `OP_RETURN` bug that allowed the sender to end the script early instead of making a fail tx which allowed the sender to steal money from other users, this bug was fixed by Satoshi in production
    -   Reads all the Op-Codes until finishes every script
-   [JavaScript implementation of Script](https://github.com/charliermarsh/script)
-   [Pay-to-Pubkey Hash](https://en.bitcoin.it/wiki/Transaction#Pay-to-PubkeyHash)
    -   The bitcoin can only be spent by the owner of the private key corresponding to the public key provided in the script.
-   [Pay-to-Script Hash](https://en.bitcoin.it/wiki/Pay_to_script_hash)
    -   They allow transactions to be sent to a script hash (address starting with 3) instead of a public key hash (addresses starting with 1)
    -   They are flexible since the spend requirements are defined by the script
