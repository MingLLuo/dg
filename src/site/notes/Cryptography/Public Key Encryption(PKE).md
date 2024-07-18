---
{"dg-publish":true,"dg-permalink":"pke","permalink":"/pke/","noteIcon":"","created":"2024-06-18T16:01:53.232+08:00","updated":"2024-06-18T19:02:40.078+08:00"}
---

#Cryptography #PKE
- **Key Generation:** $(sk, pk) \leftarrow Gen(\lambda)$
- **Encryption:** $c \leftarrow Enc(pk, m)$
- **Decryption:** $m \leftarrow Dec(sk, c)$
	- $Gen$ and $Enc$ are probabilistic (randomized) algorithms.

- **Correctness:** $m = Dec(sk, Enc(pk, m))$ for all key pairs, messages, and encryption randomness.
- **Semantic security [Goldwasser-Micali '82] (for $m \in \{0, 1\}$):**
	- Distributions $(pk, E(pk, 0)), (pk, E(pk, 1))$ are indistinguishable (computationally hard to distinguish).
