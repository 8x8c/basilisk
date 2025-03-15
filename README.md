# basilisk
Rust file encryption program implementing serpent

/////////////////////////////////////////////////////////////////////////

above is the original readme. 

this is a fork. I have to give the original author credit, as it is the ONLY rust cli serpent algo that encrypts files at the time i searched. 


Here is a chat gpt audit of the code

Some Observations and Caveats

Block Encryption (Likely ECB-Style)

The code encrypts (or decrypts) each BLOCK_SIZE chunk (16 bytes) independently with encrypt_block / decrypt_block. That’s effectively Electronic Codebook (ECB) mode unless there’s some layering not obvious in the code.
ECB mode is almost never recommended for large files in real-world cryptography because it can reveal patterns in the plaintext (for example, identical 16-byte blocks encrypt to identical ciphertext blocks). A more secure approach often uses an IV (initialization vector) + a block chaining mode (e.g., CBC, CTR, GCM, etc.) or an authenticated scheme like AES-GCM or ChaCha20-Poly1305 (though we’re talking Serpent here).
No Authentication or Integrity Check

The code does not add any cryptographic signature, HMAC, or similar authenticity check. If an attacker modifies the ciphertext on disk, there’s no way for this tool to detect that tampering—it’ll just try to decrypt whatever is present, possibly producing corrupted data.
For many use cases, it’s important to verify integrity, so some additional steps might be needed if reliability or security under adversarial conditions is crucial.
Padding With Zero Bytes

It looks like the code might do some zero-padding: it reads 16 bytes at a time, and if the last block is shorter, it’s padded with zeros. Then upon writing back, trailing zeros from the last block are removed.
This can work, but it’s not a standard padding scheme (like PKCS#7) and can cause ambiguities if the actual data ends in one or more zero bytes. Typically, you’d see something like PKCS#7 to preserve data length accurately.
Performance and Threading

The code spawns a separate thread for each directory (and subdirectory). This might not matter for small trees, but for large hierarchies, that’s a lot of threads. A more typical approach might use a worker thread pool or asynchronous I/O.
That said, this is more of a design/implementation detail than a strict security or correctness concern.
Key Generation

The code uses a BLAKE3 hash of the passphrase as the key material, then slices it down to the desired key size. This is a quick passphrase -> key solution, but for stronger passphrase-based key derivation, many applications prefer password-based key derivation functions (e.g., PBKDF2, scrypt, or Argon2) to slow down brute-forcing. BLAKE3 is fast, which can be great for hashing in general, but also means brute-force attempts against a short or weak passphrase can be attempted very quickly.
Is It Good?
It depends on what your needs are:

Yes, it works: If your main goal is simply to do a straightforward block encryption using the Serpent cipher, then yes, it does that correctly. The code is clearly calling into Serpent for encryption/decryption.

But for robust security: In modern cryptographic applications, you typically want:

A proper block mode that does not reveal data patterns (e.g., CBC, CTR, or an authenticated mode).
Integrity and authentication (e.g., GCM, or a MAC).
A well-tested library for file encryption (there are many that handle the details of key derivation, IV management, authenticity checks, etc.).
If you just wanted a simple example of how to wire up Serpent encryption/decryption in Rust, this might suffice. But for production or highly sensitive data, you’d probably want to add more security features (or use something like age, OpenPGP, or other well-audited encryption utilities).

Overall, it does use Serpent, but it’s fairly “bare-bones” from a cryptographic standpoint. As always, the level of “good” depends on your threat model and requirements for security and reliability.
