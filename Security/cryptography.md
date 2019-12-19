# Cryptology

Is the combination of:
Cryptography - the science of protecting information
Cryptoanalysis - the science of breaking protection

## Terminology
- Plaintext
    The text to be encrypted
- Cryptosystem
    Mathimatical process to encrypt, encipher, encode
- Crypto variable
    The key to be used for encryption <- this must be kept secret at all times
- Ciphertext / cryptogram
    The plaintext that has been altered using cryptography
- Decryption
    "Unlock" the encrypted text back to normal form (plaintext) 
- Key space
    Total number of possible keys
- Initialization Vector (IV)
    A random value added to the beginning of a message before encryption <- this is not secret information

## The process
Plaintext -> prepend IV to plaintext -> encrypt using crypto variable -> transmit message -> remove IV and use key to decrypt -> plaintext

## Benefits
- Confidentiality
    Protect information from disclosure
- Non-repudiation
    Provide proof of action - e.g. cannot deny sending a message
- Integrity
    Message cannot be changed
- Proof of origin
- Authenticity
    You can make sure communication is with the correct person, service, etc.

## Basic operations
Substituion
    Basically just replacing one value for another
Transposition
    Mutate a word by rearranging the order
Confusion
    Mixing key values during repeated rounds of encryption

## Symmetric ciphers
Same key is used for both encryption and decryption operations
Also known as single, shared, secret, session or private key encryption

The symmetric key has to be shared. From the encryptor to the decryptor. Sharing the key is done
"out of band", meaning it must be shared using different means


## How symmetric algorithms work
Block mode cipher
    Encrypts blocks of plaintext, one block (group of characters) at a time
    Larger blocks are considered more secure
    Two modes:
        Electronic Code Book (ECB)
        Cipher Block Chaining (CBC)
    ECB does not use an IV. Each block is encrypted individually. Considered weak.
    CBC is the most frequently used. Plaintext is split into blocks. The first block is encrypted using IV
    and the remaining blocks are encrypted using the previous block as a sort of IV, also known as avalanching
Stream mode cipher
    Fast encryption. Every single character is encrypted individually using the previous value as IV
    Counter (CTR) mode

## Digital signatures
Combine two encryption techniques: one-way hashing and asymmetric encryption
SHA512 used to provide message integrity (hashing) and RSA (asymmetric) as proof of origin
i.e. this message is encrypted with specific key

Digital signatures provide proof of origin and integrity

## Digital signature process    
Plaintext -> get a "digest" by hashing -> sign digest using sender's private key -> digital signature -> append digital signature to message -> transmit message -> decrypt digital signature using sender's public key -> digest -> hash the message -> compare message hash with digest

## Public Key Infrastructure (PKI)
Registration for certificates
Certificate issuance
Storage of certificates
Distribution of certificates
Validation of certificates