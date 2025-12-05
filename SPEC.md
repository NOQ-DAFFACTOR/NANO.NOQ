# NANO.NOQ – Specification (SPEC.md)

Version: 1.0 — *Experimental Format*

This document describes the technical specifications of the **.noq** format, the encoding/decoding process, mutation structure, and interoperability rules. All information here represents the actual implementation in `index.html` created in the NANO.NOQ project.

---

# 1. Purpose of the Format

The **.noq** format is designed to:

1. Store **AES-GCM keys (32 bytes)** in a small binary container.
2. Provide a simple **integrity check** using SHA‑256(key)[0..3].
3. Prevent the key from being displayed in the UI ("hidden mode").
4. Remain compatible with WebCrypto browser implementations.
5. Be usable without a server (100% client-side).

This format is **not** a new security algorithm, and does not add cryptographic strength beyond AES‑GCM.

---

# 2. Structure of the `.noq` File

The `.noq` format is a binary structure as follows:

```
Offset | Size | Description
-----------------------------------------------
0x00   | 4      | Header: ASCII "N O Q 1" (0x4E 0x4F 0x51 0x01)
0x04   | 2      | Key length L (big-endian) – default: 0x0020 (32)
0x06   | L      | Raw AES-GCM key (32 random bytes)
0x06+L | 4      | Integrity slice = SHA-256(key)[0..3]
…      | 32     | Random padding (crypto.getRandomValues)
```

# Example layout (32-byte key):

```
4 bytes   header
2 bytes   length = 0x00 0x20
32 bytes  key
4 bytes   sha256(key)[0..3]
32 bytes  padding
```

# Design Rationale

* Avoid excessive metadata.
* Use hash-slice for fast integrity without exposing the key.
* Random padding eliminates size patterns.
* No additional encryption inside `.noq`; security remains with AES‑GCM.

---

# 3. Encoding Process

Encoding text → ciphertext + `.noq` occurs in the following steps:

# 3.1 Generate Key

```
keyBytes = random(32 bytes)  // crypto.getRandomValues
```

# 3.2 AES‑GCM Encryption

* IV: 12 random bytes (WebCrypto default)
* Ciphertext with tag is in the WebCrypto output.

Output is combined:

```
combined = iv (12 bytes) + ciphertext + tag
```

# 3.3 Base64URL

```
b64u = base64urlEncode(combined)
```

# 3.4 Mutation Layer (optional)

1-to-1 mapping based on the character table in the code:

* NOT cryptography.
* Merely a deterministic transformation.
* Influenced by `seed` + `reloadCounter`.

# 3.5 Key storage

The key is **not displayed**, but stored in `.noq` as a binary container.

---

# 4. Decoding Process

# 4.1 Load `.noq`

The parser reads:

* header
* length
* key
* hash-slice
* verifies hash-slice

If verification fails → file is rejected.

# 4.2 Reverse Mutation

```
b64u = reverseMap(mutatedCiphertext)
```

If characters are outside the mapping → error.

# 4.3 AES‑GCM Decrypt

```
plaintext = AES-GCM(key, b64u)
```

If key is incorrect → decrypt error.

---

# 5. Mutation Layer

Mutation is character-by-character translation:

```
A → N
B → J
C → Z...

```

Reverse mapping is automatically constructed at runtime:

```
reverse[mapping[ch]] = ch
```

# Notes:

* Mapping can be changed by the user.
* Mapping does not have to be open-source.
* Mutation is not "new security", only obfuscation.
* Mutation can cause ciphertext incompatibility when mappings differ.

---

# 6. Ciphertext Specification

Ciphertext is stored as **mutated Base64URL**.
General format:

```
mutated(Base64URL(iv + ciphertext + tag))
```

`.noqc` is optional, the format is only a text file containing mutated ciphertext.

---

# 7. UI and State Operations

# 7.1 Hidden key state

If the key comes from `.noq`:

```
noqKeyFlag = true
noqKeyBytes = keyBytes
```

The key cannot be displayed or copied.

If the key comes from the encode process:

* Displayed in masked form.
* Can be copied.

# 7.2 Auto-decode

After `.noq` is loaded, if the input contains ciphertext, the application immediately attempts to decode it.

---

# 8. Error Handling

The format specifies the following errors:

* Invalid header → "Not a NOQ file"
* File too short → "File too short"
* Unusual length → "Invalid key length"
* Hash-slice mismatch → "NOQ verification failed"
* Reverse mapping failed → "Reverse mapping failed"
* AES-GCM decrypt error → generic WebCrypto error

All errors appear in the UI via `setStatus()`.

---

# 9. Limitations & Security Notes

1. `.noq` **does not add** cryptographic security.
2. AES‑GCM is the only real security layer.
3. The mutation layer has no cryptographic strength.
4. Keys should not be stored in `.noq` for production systems.
5. The format is **experimental** and has not been audited.
6. Browser runtimes have nondeterministic entropy.
7. Not suitable for high-risk applications.

---

# 10. Compatibility

* Built-in WebCrypto implementation (AES-GCM, SHA-256)
* Base64URL compliant with RFC 4648
* Platform-independent
* Can be run locally without a server

---

# 11. Implementation Status

* One `index.html` file with all UI + crypto + formatting code.
* `.noq` and `.noqc` generated at runtime.
* All operations performed client-side.

---

# 12. References

* AES-GCM – NIST SP 800-38D
* Base64URL – RFC 4648
* SHA‑256 – FIPS PUB 180‑4
* WebCrypto — W3C / MDN
* crypto.getRandomValues() — MDN

---

# 13. Closing

The `.noq` format is a **formal experiment** in designing a small browser-based key container. This specification is designed to be clear, transparent, and replicable without bias or excessive security claims.

Users are free to modify the mapping, extend the format, or add new versions as needed, as long as they understand the security limitations and experimental nature.
