---
{"dg-publish":true,"permalink":"/cryptography/simulation-based-security/","created":"2024-07-13T01:18:21.567+08:00","updated":"2024-07-13T01:25:42.565+08:00"}
---

Unlike game-based security analyses, simulation-based security analyses compare a real-world cryptographic protocol to an idealized abstraction of functionality.
The functionality needs a trusted party to execute on behalf of everyone if such a trusted party were to exist.

Simulation-based proofs show that the real-world protocol is no *worse* or *leakier* than the ideal functionality, in the sense that any real-world artifacts of the protocol can be simulated from the relevant ideal-world inputs and outputs.

Hence, simulation-based security analyses involve the adversary **Adv**, the simulator **Sim**, and the distinguisher.

One benefit of simulation-based security (beyond game-based notions) is to facilitate security analyses involving composition:
- combining multiple instances of a single cryptographic protocol
- combining many cryptographic protocols
- integrating cryptographic protocols into a larger system
