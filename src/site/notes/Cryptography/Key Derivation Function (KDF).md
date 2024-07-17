---
{"dg-publish":true,"permalink":"/cryptography/key-derivation-function-kdf/","created":"2024-07-16T21:50:34.903+08:00","updated":"2024-07-16T21:56:07.217+08:00"}
---

#Cryptography  #KDF
A Key Derivation Function (KDF) is a function that takes some source of initial keying material and uses it to derive one or more cryptographically strong keys. This specification uses a KDF with the following API and parameters:
- Extract(salt, ikm): Extract a pseudorandom key of fixed length `Nx` bytes from input keying material `ikm` and an optional byte string `salt`.
- Expand(prk, info, L): Expand a pseudorandom key `prk` using the optional string `info` into `L` bytes of output keying material.
- Nx: The output size of the `Extract()` function in bytes.