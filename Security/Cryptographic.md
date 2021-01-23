# Security and Cryptography

## Entropy
* Entropy is a measure of randomness. This is useful, for example, when determining the strength of a password.
* Entropy is measured in bits, and when selecting uniformly at random from a set of possible outcomes, the entropy is equal to log_2(# of possibilities). 
    * e.g. A fair coin flip gives 1 bit of entropy. A dice roll (of a 6-sided die) has ~2.58 bits of entropy.

## Cryptographic Hash functions
* A cryptographic hash function maps data of arbitrary size to a fixed size, and has some special properties.
* A hash function has the following properties:
    * Deterministic: the same input always generates the same output.
    * Non-invertible: it is hard to find an input m such that hash(m) = h for some desired output h.
    * Target collision resistant: given an input m_1, it’s hard to find a different input m_2 such that hash(m_1) = hash(m_2).
    * Collision resistant: it’s hard to find two inputs m_1 and m_2 such that hash(m_1) = hash(m_2) (note that this is a strictly stronger property than target collision resistance).

## Key derivation functions
* A related concept to cryptographic hashes, key derivation functions (KDFs) are used for a number of applications, including producing fixed-length output for use as keys in other cryptographic algorithms. 
* Usually, KDFs are deliberately slow, in order to slow down offline brute-force attacks.

## Symmetric cryptography
* Symmetric cryptography accomplishes this with the following set of functionality:
```
keygen() -> key  (this function is randomized)

encrypt(plaintext: array<byte>, key) -> array<byte>  (the ciphertext)
decrypt(ciphertext: array<byte>, key) -> array<byte>  (the plaintext)
```
* The decrypt function has the obvious correctness property, that ```decrypt(encrypt(m, k), k) = m.```
* e.g. [AES](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard)

## Asymmetric cryptography
* Asymmetric cryptosystems provide the following set of functionality, to encrypt/decrypt and to sign/verify:
```
keygen() -> (public key, private key)  (this function is randomized)

encrypt(plaintext: array<byte>, public key) -> array<byte>  (the ciphertext)
decrypt(ciphertext: array<byte>, private key) -> array<byte>  (the plaintext)

sign(message: array<byte>, private key) -> array<byte>  (the signature)
verify(message: array<byte>, signature: array<byte>, public key) -> bool  (whether or not the signature is valid)
```
* The decrypt function: ```decrypt(encrypt(m, public key), private key) = m.```
* The verify function: ```verify(message, sign(message, private key), public key) = true.```

### Key distribution
* It has a big challenge of distributing public keys / mapping public keys to real-world identities.
* [Web of trust](https://en.wikipedia.org/wiki/Web_of_trust)
* [Social proof](https://keybase.io/blog/chat-apps-softer-than-tofu)

 
## References
* [Missing Semester - Security and Cryptography](https://missing.csail.mit.edu/2020/security/)
  





