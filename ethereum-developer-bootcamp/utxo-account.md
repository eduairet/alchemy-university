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
