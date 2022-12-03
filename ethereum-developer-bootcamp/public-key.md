# Public Key Cryptography

Cryptography is the study of encrypting messages so they cannot be decrypted even if they are intercepted.

## Symmetric key cryptography

The owners of the key can decrypt the message, like passwords in the Internet.

```JavaScript
// Fictional secret message
import SecretMessage from './secret-message';
const password = '···············'
if (password == SecretMessage.password) {
    // Decrypts the message if the password is right
    secretMessage.decrypt()
}
```

## Asymmetric key cryptography

One key signs the message the other verifies the sign and the owner of the private key will be the only one able to decrypt the message.

Two popular public key encryption algorithms are the [RSA (cryptosystem)](<https://en.wikipedia.org/wiki/RSA_(cryptosystem)>) and the [Elliptic Curve Digital Signature Algorithm (ECDSA)](https://en.wikipedia.org/wiki/Elliptic_Curve_Digital_Signature_Algorithm), this is the one used by Bitcoin and Ethereum because it grants security with smaller keys, allows the public key to be recovered from the signed message together with the signature, and is useful in bandwidth constrained or storage constrained environments (such as blockchain systems).

```JavaScript
const { keccak256 } = require("ethereum-cryptography/keccak");
const { utf8ToBytes } = require("ethereum-cryptography/utils");
const secp = require("ethereum-cryptography/secp256k1");

// The function returns a hash digest of the message
const hashMessage = message => {
        const bytes = utf8ToBytes(message),
            hash = keccak256(bytes);
        return hash;
    },
    PRIVATE_KEY = "·············";

// This function signs the message with the prived key
// and returns both a signature and a recovery bit
async function signMessage(msg) {
    const hashedMessage = hashMessage(msg),
        sign = secp.sign(
            hashedMessage,
            PRIVATE_KEY,
            { recovered: true}
        );
    return sign;
};

// This function will allow us to recover our public key
async function recoverKey(message, signature, recoveryBit) {
    return secp.recoverPublicKey(
        hashMessage(message),
        signature,
        recoveryBit
    );
}
```

## Ethereum Address

An Ethereum address is the last 20 bytes of the hash of the public key, and you can always get the address from the public key.

```JavaScript
const secp = require("ethereum-cryptography/secp256k1");
const { keccak256 } = require("ethereum-cryptography/keccak");

function getAddress(publicKey) {
    // Slice the first byte of the public key (format of the key)
    // And then slice the last 20 bytes of the result
    return keccak256(publicKey.slice(1)).slice(-20, this.length);
}
```

### References

1.  [Crypto: How the Code Rebels Beat the Government Saving Privacy in the Digital Age by Steven Levy](https://www.amazon.com/Crypto-Rebels-Government-Privacy-Digital/dp/0140244328)
2.  [ECDSA: The digital signature algorithm of a better internet by Nick Sullivan](https://blog.cloudflare.com/ecdsa-the-digital-signature-algorithm-of-a-better-internet/)
3.  [ECDSA in Wikipedia](https://en.wikipedia.org/wiki/Elliptic_Curve_Digital_Signature_Algorithm)
4.  [ECDSA in Practical Cryptography for Developers](https://cryptobook.nakov.com/digital-signatures/ecdsa-sign-verify-messages)
5.  [secp256k1](https://en.bitcoin.it/wiki/Secp256k1)
6.  [How Bitcoin derives addresses](https://en.bitcoin.it/wiki/Address)
7.  [Address derivation and the bitcoin checksum](https://en.bitcoin.it/wiki/Technical_background_of_version_1_Bitcoin_addresses#:~:text=A%20Bitcoin%20address%20is%20a,that%20the%20signature%20is%20valid.)
8.  [Diffie-Hellman exchange](https://en.wikipedia.org/wiki/Diffie%E2%80%93Hellman_key_exchange)
9.  [TLS handshake for HTTPS](https://security.stackexchange.com/a/41226)
10. [Secret Key Exchange by Computerphile](https://www.youtube.com/watch?v=NmM9HA2MQGI)
11. [Diffie Hellman by Computerphile](https://www.youtube.com/watch?v=Yjrfm_oRO0w)
12. [Elliptic Curves by Computerphile](https://www.youtube.com/watch?v=NF1pwjL9-DE)

13. [RSA in Wikipedia](<https://en.wikipedia.org/wiki/RSA_(cryptosystem)>)
14. [RSA in Practical Cryptography for Developers](https://cryptobook.nakov.com/digital-signatures/rsa-signatures)

15. RSA by Eddie Woo, [part 1](https://www.youtube.com/watch?v=4zahvcJ9glg), [part 2](https://www.youtube.com/watch?v=oOcTVTpUsPQ)
16. [RSA Backdoor](https://www.reuters.com/article/us-usa-security-nsa-rsa/exclusive-nsa-infiltrated-rsa-security-more-deeply-than-thought-study-idUSBREA2U0TY20140331)
