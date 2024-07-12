---
{"dg-publish":true,"permalink":"/cryptography/universal-composable-security/","created":"2024-07-13T01:14:20.439+08:00","updated":"2024-07-13T01:37:44.016+08:00"}
---


- As a huge direction to security proof, this page will update periodically.

### What's UC?

We first talk about [[Cryptography/Simulation-Based Security\|Simulation-Based Security]].
- Simulation-based proofs show that the real-world protocol is no *worse* or *leakier* than the ideal functionality, in the sense that any real-world artifacts of the protocol can be simulated from the relevant ideal-world inputs and outputs.

For now, there exist many formulations of UC frameworks. Canetti's UC security, with several enhancements over time, is a special case of simulation-based security that embodies the idea where other protocols (whether other instances of the same protocol or different protocols entirely) may be run at the same time and interleaved arbitrarily with the protocol under consideration.


### Reference
- Little, Ryan, Lucy Qin, and Mayank Varia. “Secure Account Recovery for a Privacy-Preserving Web Service,” 2024. Cryptology ePrint Archive. [https://eprint.iacr.org/2024/962](https://eprint.iacr.org/2024/962).