# NANO.NOQ
NANO.NOQ description 

 # 🟡 NANO.NOQ — Hybrid Encryption System & Secure Key Container (.noq)

**NANO.NOQ** is a hybrid encryption system that combines **AES-256-GCM**, **deterministic character mutation**, and a **new key-container file format** called **`.noq`**. This project is designed to provide a simple, secure, portable, and flexible encryption experience without the need for additional software installation.

NOQ aims to simplify key management, separate ciphertext from keys in accordance with modern cryptographic best practices, and provide the option of using private mappings for additional security.

---

# 🔐 1. Hybrid Mechanism (Technical Summary)

 **Encryption**

1. Plaintext → AES-256-GCM
2. Output `(IV || ciphertext || tag)` → Base64URL
3. Base64URL → **substitution mutation** based on mapping
4. AES key (32 bytes) → stored in `.noq`

 **Decryption**

1. Mutated ciphertext → reverse mutation (mapping)
2. Base64URL → decode
3. AES-GCM decrypt with key from `.noq`
4. Result: plaintext

The mutation is a *deterministic obfuscation layer* and **is not** a primary cryptographic component. Core security remains with AES-256-GCM.

---

# 📁 2. `.noq` Format (Key Container)

`.noq` is a binary container that stores encryption keys separately and securely.

Structure (74 bytes for AES-256):

| Section            | Size | Function                          |
| ----------------- | ------ | ------------------------------- |
| `NOQ1` Header     | 4B     | Format identification             |
| Key length        | 2B     | Key length                     |
| AES-256 Key       | 32B    | Encryption key                  |
| SHA256(key)[0..3] | 4B     | Integrity-slice (corruption check)   |
| Random padding      | 32B    | Disguises patterns, prevents analysis |

Characteristics:

* Cannot be opened/read manually
* Can only be **uploaded/downloaded**
* Separates keys cryptographically correctly
* Resistant to brute-force attacks due to random padding

---

# 🧩 3. Mapping System (Mutation)

Mutation uses a Base64URL-based character substitution table.

# Why is mapping important?

* Mapping determines the mutation result
* Private mapping → ciphertext can only be recovered if the mapping matches
* Even a 1-character difference in mapping → decoding is **impossible**

# Mapping complexity

Number of possible mappings:

```
64!  ≈ 1.27 × 10^90
```

This means that brute-force mapping is **practically impossible**.

# Mapping storage modes

1. **Internal (public mapping)**

   * Simple, suitable for public use
   * Not suitable for high-level confidential data

2. **External (private mapping)**

   * Stored separately, such as in `mapping.json`
   * More secure and flexible
   * If lost ⇒ ciphertext cannot be recovered permanently

---

# 📦 4. `.noqc` Format (Ciphertext Container) — *Optional*

`.noqc` stores mutated ciphertext as a structured binary container (like `.noq` but for ciphertext).

Structure:

* `NOQC` header
* Cipher length
* Seed length
* Mutated ciphertext
* Seed
* SHA256(slice)
* Padding

Purpose:

* Avoid copy-pasting ciphertext
* Package ciphertext like a regular file

---

# 📊 5. Overhead Calculation and Size

Example plaintext 100 bytes produces:

* AES-GCM output: 128 bytes
* Base64URL: 172 characters
* `.noqc`: ~223 bytes
* `.noq`: 74 bytes

Overhead:

```
Overhead ≈ 29–30%
```

Reasonable value for a secure container.

---

# 🔒 6. Security & Transparency

**Core security comes from AES-256-GCM**, including 16-byte Tag authentication.

Mutations:

* Add visual confusion & make ciphertext unclear
* However, **not the primary cryptographic layer**

HMAC-slice:

* 4 bytes fast for corruption detection
* Not a replacement for full MAC
* For critical systems, use real HMAC or separate signature

---

# ⭐ 7. Advantages of NANO.NOQ

* Keys and ciphertext are **completely separated**
* `.noq` hides keys without the risk of "view-copy-paste"
* Mutations can use private mappings
* Runs 100% in the browser (no installation required)
* Can be used for modern cryptography education
* `.noq` / `.noqc` containers are simple and portable

---

# ⚠️ 8. Limitations

* Mutation = obfuscation, not formal cryptography
* Lost mapping = data cannot be recovered
* `.noq` is not a replacement for HSM or enterprise systems
* Requires audit if used in sensitive industries (banking/fintech)

---

# 📜 9. License

The project is released under **GPLv2**.
Mapping may be **private** and is not required to be released under GPLv2.

---

# 👤 10. Creator

NANO.NOQ was created by **Daffactor(NOQ.DAFFACTOR)**
through a combination of personal creativity and AI assistance in technical design, format structure, and implementation.
