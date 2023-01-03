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
    -   NOTES:
        -   HTTPS: The browser connects to that site over encrypted connection using public key cryptography and a digital certificate
            -   Thanks to TLS web certificates support different public key algorithms
            -   Most websites use RSA for legacy reasons
        -   RSA
            -   Public key: is a large number product of two primes plus a smaller number
            -   Private key
                -   Related number
                -   Used to create a digital signature
        -   ECC
            -   Public key:
                -   Equation for an elliptic curve and a point that lies on that curve
            -   The private key is a number
        -   ECDSA (Elliptic Curve Digital Signature Algorithm)
            -   Not widespread use in the internet but are the base of Blockchain cryptography
        -   With ECDSA you can get the same security as with RSA but with smaller keys
            -   Smaller keys are faster to compute
            -   If the private key is leaked the data associated to the owner will be compromised
3.  [ECDSA in Wikipedia](https://en.wikipedia.org/wiki/Elliptic_Curve_Digital_Signature_Algorithm)
    -   NOTES:
        -   Key and signature size ECDS:
            -   Private key -> 160 bits
            -   Signature -> 4t bits (320 bits for a security level of 80 bits)
        -   Bad implementation of the algorithm can compromise the data like Play Station signing in 2010 or the Java SecureRandom class that allowed hackers steal bitcoin keys
4.  [ECDSA in Practical Cryptography for Developers](https://cryptobook.nakov.com/digital-signatures/ecdsa-sign-verify-messages)

    -   ECDSA (Elliptic Curve Digital Signature Algorithm):
        -   Cryptographically secure digital signature scheme based on elliptic-curve cryptography
        -   It relies on the the math of the cyclic groups of elliptic curves over finite fields and the elliptic curve discrete logarithm problem
        -   The algorithm is shorter than RSA with the same security level
        -   ECDSA uses cryptographic elliptic curves (EC), which define:
            -   Generator point G, used for scalar multiplication on the curve
            -   Order n of the subgroup of EC points, generated by G, which defines the length of the private keys (e.g. 256 bits)
        -   ECDSA key-pair:
            -   Private key (random integer [0...n-1]): privKey
            -   Public key (EC point): pubKey = privKey × G
        -   ECSDA signing algorithm:
            1. `h = SHA256(Input message)`
            2. `k = HMAC-derived from h + privKey`
            3. Random point: `R = k × G` | take its x-coordinate: `r = R.x`
            4. Signature proof:
                - `s = k−1×(h+r×privKey)(mod𝓃)`
                - The modular inverse `k−1(mod𝓃)` is an integer such as `k×k−1≡1(mod𝓃)`
            5. Return the signature `{r, s}` (integers in the range [1...n-1])
        -   ECDSA signature verify:
            1. `h = SHA256(Input message)`
            2. Modular inverse: `s1 = s−1(mod𝓃)`
            3. Recover random point: `R' = (h × s1) × G + (r × s1) × pubKey`
            4. `R'` x-coordinate: `r' = R'.x`
            5. Validation result: `r' == r`
        -   ECDSA signature scheme allows the public key to be recovered from the signed message together with the signature.

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
