# NANO.NOQ
NANO.NOQ description 

# NANO.NOQ — File Key Container (Experimental)

This project is purely an **experimental format design**, not a new cryptographic system.

---

# 1. Overview

**NANO.NOQ** is a small file format designed to store **AES‑GCM keys** separately from ciphertext. This format uses the following structure:

* Fixed header (`NOQ1`)
* Key length (2 bytes)
* Key (32 bytes by default)
* HMAC-slice verification (4 bytes, taken from SHA‑256(key))
* Random padding (32 bytes)

The goal is to create a key file that is **portable**, **easy to install on any system**, and **verifiable for integrity** without exposing the key in the UI.

The `.noq` format does not add any security beyond AES‑GCM. All cryptographic security remains entirely dependent on AES‑GCM and the WebCrypto API.

---

# 2. Why This Project Was Created

The main purposes of creating the `.noq` format are:

1. **Personal experimentation** in designing a simple file format for key storage.
2. Separating *ciphertext* and *key* without the need for manual copy-pasting.
3. Creating a format that is easy for beginners to learn because the entire implementation is done in **one HTML file**.
4. To build a system that works without a server (100% local browser).

There is no ambition to replace industry standards or create new cryptographic algorithms.

---

# 3. Key Features

* AES‑256‑GCM key storage in `.noq` files.
* Integrity verification via SHA‑256(key) → 4-byte slice.
* Random padding to prevent size inference.
* Keys are not displayed if they originate from `.noq` (hidden mode).
* Ciphertext can be saved as `.txt` or `.noqc` (optional).
* The entire process takes place **client-side**, without a backend.
* *Mutation layer* feature for light obfuscation on Base64URL (not additional cryptography).

---

# 4. Brief Overview of How It Works

# 4.1 Encoding

1. Plaintext is encrypted using AES‑GCM.
2. The result (`iv + ciphertext + tag`) is converted to Base64URL.
3. Base64URL is mutated character-by-character using a deterministic mapping table.
4. The key is stored as `.noq` (not displayed to the user).

# 4.2 Decoding

1. `.noq` file is uploaded → key is extracted & verified.
2. Mutated ciphertext → reverse–mutate → Base64URL.
3. AES‑GCM decrypt → plaintext.

---

# 5. `.noq` File Format (Summary)

```
[ 4 bytes ]   Header "NOQ1"
[ 2 bytes ]   Length L (default 0x0020)
[ L bytes ]   Raw AES-GCM key
[ 4 bytes ]   SHA256(key)[0..3] (integrity check)
[32 bytes ]   Random padding
```

The file can be analyzed, but cannot be opened as text because it is stored in binary form.

---

# 6. About Mutation Mapping

Mutation is **light obfuscation**, not a cryptographic security mechanism.

* Mapping can be completely changed by the user.
* Mapping does not have to be open-source.
* Different mappings cause incompatible ciphertext.
* It is best to keep mapping separate from the public repository.

This project remains cryptographically secure because *security does not depend on the mapping*, but on AES‑GCM.

---

# 7. Limitations

* `.noq` is **not** a standard security format.
* Mutations do not add cryptographic security.
* AES‑GCM is the only real layer of security.
* Browser implementations have WebCrypto limitations.
* Not suitable for production or high-risk applications.
* The project is **experimental**.

---

# 8. How to Use

1. Open `index.html` directly in your browser.
2. Enter text → *Encode*.
3. Download the key (`.noq`).
4. Save the ciphertext (`.txt`).
5. To decode, upload `.noq` + ciphertext.

No internet or server is involved in the processing.

---

# 9. Project Status

* **Experimental.**
* Unstable and not recommended for real security.
* Focused on learning, prototyping, and exploring file formats.

---

# 10. Technical References

* AES‑GCM: NIST SP 800‑38D
* WebCrypto API — W3C
* Base64URL Encoding — RFC 4648
* SHA‑256 — FIPS PUB 180‑4
* Randomness via `crypto.getRandomValues()` — MDN

---

# 11. License

The project follows **GPLv2**, except for the **mapping** part, which is freely modifiable privately.

> "Mapping table may remain proprietary. All other source code is GPLv2."

---

# 12. Repository Structure (Summary)

```
/
 └── index.html   → all code (UI + AES-GCM + mutation + NOQ)
```

The `.noq` and `.noqc` formats are generated at runtime.

---

# 13. Creator

The NANO.NOQ project was developed as a personal experiment by Daffactor with the help of AI, focusing on format design, education, and exploration of simple key storage mechanisms. 

---


[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/NAMA_KAMU)

