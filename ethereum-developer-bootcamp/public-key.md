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
    -   EC used by bitcoin’s public key
    -   SEC -> Standards for Efficient Cryptography
    -   Non-random -> Efficient computation
6.  [How Bitcoin derives addresses](https://en.bitcoin.it/wiki/Address)
    -   26-35 alphanumeric characters
    -   Begins with 1, 3 or bc1
    -   No cost
    -   In use:
        -   P2PKH - 1BvBMSEYstWetqTFn5Au4m4GFg7xJaNVN2
        -   P2SH - 3J98t1WpEZ73CNmQviecrnyiWrnqRhWNLy
        -   Bech32 - bc1qar0srrr7xfkvy5l643lydnw9re59gtzzwf5mdq
    -   Send money to one address
    -   One per transaction
    -   They can be generated offline
    -   A master public key can generate several invoices, useful for unsafe scenarios (webservers)
    -   Old-style are case sensitive, bech32 don’t
    -   Bitcoin wallets sign a message when the invoice has received money
    -   Some services use an invoice for authentication
    -   BTC transactions don’t have a from address
    -   Validation needs specialized software
    -   Multi-sig invoice are only possible with P2SH and bech32
    -   Random digits (34-26), no O0Il
    -   Can’t be reused, don’t have balance
7.  [Address derivation and the bitcoin checksum](https://en.bitcoin.it/wiki/Technical_background_of_version_1_Bitcoin_addresses#:~:text=A%20Bitcoin%20address%20is%20a,that%20the%20signature%20is%20valid.)
    -   Base58Check encoding
        -   Private ECDSA key
        -   Public key from private key (33 bytes, 1 byte 0x02 (y-coord is even), and 32 bytes corresponding to X coordinate)
        -   SHA-256 on public key
        -   RIPEMD-160 on the digest of previous step
        -   Add version byte to the result of previous step (0x00 mainnet)
        -   SHA-256 and RIPEMD-160 on the result
        -   SHA-256 on previous SHA-256 digest
        -   Checksum: First 4 bytes of second hash
        -   Add that checksum to the end of the result of step 4
        -   Convert the result (25-byte binary address) into a base58 string using Base58Check encoding.
    -   Collisions are very unlikely (2 persons with the same key)
    -   Create keys just with verified software
8.  [Diffie-Hellman exchange](https://en.wikipedia.org/wiki/Diffie%E2%80%93Hellman_key_exchange)
9.  [TLS handshake for HTTPS](https://security.stackexchange.com/a/41226)
10. [Secret Key Exchange by Computerphile](https://www.youtube.com/watch?v=NmM9HA2MQGI)
11. [Diffie Hellman by Computerphile](https://www.youtube.com/watch?v=Yjrfm_oRO0w)
    -   Last 4 resources are summarized here
    -   Secure exchange of cryptographic keys over a public channel
    -   Removed the physical variable of key exchanging allowing the exchange of them even through insecure channels
    -   Predecessor of RSA
    -   Created by Diffie, Hellman, and Merkle
    -   Example:
        1. Alice and Bob publicly agree to use a modulus p = 23 and base g = 5 (which is a primitive root modulo 23).
        2. Alice chooses a secret integer a = 4, then sends Bob A = ga mod p
            - A = 54 mod 23 = 4 (in this example both A and a have the same value 4, but this is usually not the case)
        3. Bob chooses a secret integer b = 3, then sends Alice B = gb mod p
            - B = 53 mod 23 = 10
        4. Alice computes s = Ba mod p
            - s = 104 mod 23 = 18
        5. Bob computes s = Ab mod p
            - s = 43 mod 23 = 18
        6. Alice and Bob now share a secret (the number 18).
    -   In the example just p, g, A, B would’ve been exposed to a third party, if an attacker wanted to discover the message, it would be needed to split A and B, which is very difficult even for supercomputers
    -   It can be used with more than two parties adding more private keys
    -   Vulnerable to a man-in-the-middle and Logjam attacks
    -   Mixing DH with asymmetric cryptography can be useful, especially when sending the public keys
    -   The previous point is crucial for TLS handshake for HTTPS
    -   When you log into the browser you perform a DH exchange
    -   Exchange public variables and combine them with private variables and getting the same key
    -   Small numbers are not safe DH needs large numbers to be brute force resistant
12. [Elliptic Curves by Computerphile](https://www.youtube.com/watch?v=NF1pwjL9-DE)
    -   Fast to be computed
    -   Has always the same length
    -   The output has x and y but usually with x is enough because it’s very secure
13. [RSA in Wikipedia](<https://en.wikipedia.org/wiki/RSA_(cryptosystem)>)
    -   Creators: Rivest, Shamir, Adleman
    -   Public key cryptosystem
    -   Encryption key is public and decryption key is private
    -   Public key based on two large prime numbers (secret) and an auxiliary value
    -   Messages can be encrypted by anyone with the public key and decoded by the owner of the prime numbers
    -   Slow algorithm
    -   4 steps -> key generation, key distribution, encryption, decryption
    -   Find 3 very large positive integers: e, d, and n with modular exponentiation for them
    -   Process
        -   Choose 2 prime numbers large p, q (random, secret, similar magnitude)
        -   Compute n = pq
        -   Compute λ(n)
        -   Choose an integer e such that 1 < e < λ(n) and gcd(e, λ(n)) = 1
        -   Determine d as d ≡ e−1 (mod λ(n))
14. [RSA in Practical Cryptography for Developers](https://cryptobook.nakov.com/digital-signatures/rsa-signatures)
    -   Easier to digest content, practical for apply into a project
15. RSA by Eddie Woo, [part 1](https://www.youtube.com/watch?v=4zahvcJ9glg), [part 2](https://www.youtube.com/watch?v=oOcTVTpUsPQ)
    -   The best ans simplest way to understand it
16. [RSA Backdoor](https://www.reuters.com/article/us-usa-security-nsa-rsa/exclusive-nsa-infiltrated-rsa-security-more-deeply-than-thought-study-idUSBREA2U0TY20140331)
