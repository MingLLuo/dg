---
{"dg-publish":true,"permalink":"/cryptography/password-authenticated-key-exchange-pake/","created":"2024-06-20T14:24:53.736+08:00","updated":"2024-07-16T21:42:16.972+08:00"}
---


#Cryptography #AKE #PAKE 

Password authentication is ubiquitous in many applications. In a common implementation, a client authenticates to a server by sending its client ID and password to the server over a secure connection. This makes the password vulnerable to server mishandling, including accidentally logging the password or storing it in plaintext in a database. Server compromise resulting in access to these plaintext passwords is not an uncommon security incident, even among security-conscious organizations. Moreover, plaintext password authentication over secure channels such as TLS is also vulnerable to cases where TLS may fail, including PKI attacks, certificate mishandling, termination outside the security perimeter, visibility to TLS-terminating intermediaries, and more.
### What's a PAKE?
A PAKE protocol, first introduced by [Bellovin and Merritt](https://www.cs.columbia.edu/~smb/papers/neke.pdf), is a special form of [cryptographic key exchange protocol.](https://en.wikipedia.org/wiki/Key_exchange) Key exchange (or “key agreement”) protocols are designed to help two parties (call them a _client_ and _server_) agree on a shared key, using public-key cryptography. The earliest key exchange protocols — like classical [Diffie-Hellman](https://en.wikipedia.org/wiki/Diffie%E2%80%93Hellman_key_exchange) — were _unauthenticated_, which made them vulnerable to [man-in-the-middle](https://en.wikipedia.org/wiki/Man-in-the-middle_attack) attacks. 

The distinguishing feature of PAKE protocols is the client will authenticate herself to the server using a _password_. For obvious reasons, the password, or a hash of it, is assumed to be already known to the server, which is what allows for checking.

If this was all we required, PAKE protocols would be easy to build. 

What makes a PAKE truly useful is that it should also _protect the client’s password_. A stronger version of this guarantee can be stated as follows: after a login attempt (valid, or invalid) both the client and server should learn _only whether the client’s password matched the server’s expected value,_ and no additional information. This is a powerful guarantee. It’s not dissimilar to what we ask for from a [zero-knowledge proof](https://blog.cryptographyengineering.com/2014/11/27/zero-knowledge-proofs-illustrated-primer/).

The obvious problem with PAKE is that many people don’t want to run a “key exchange” protocol in the first place! They just want to verify that a user knows a password.

Some applications:
- [[Cryptography/SRP\|SRP]]
- [[Cryptography/OPAQUE\|OPAQUE]]
### Balanced and Augmented protocols
PAKE protocols are commonly classified as either *balanced or augmented*.
#### Balanced PAKE
A balanced PAKE assumes that the two parties share a secret, which is a password or derived from a password.
##### Secure Requirements
- Resisting offline dictionary attacks
- Limiting online attacks to one password guess per protocol execution
- Ensuring session-key security
- Providing forward secrecy
#### Augmented PAKE
When a balanced PAKE is used in a client-server setting, if the secret stored on the server is stolen, it can be directly used to impersonate the client. To address this, an augmented PAKE adds a “server compromise resistance” requirement:
- Even if the server is compromised, an [[Cryptography/Offline Dictionary Attack\|offline dictionary attack]] is needed to impersonate the client.

This is typically realized by requiring that the client remember a password, while the server stores only a one-way transformation of it. 

Jarecki et al. recently suggested an extra “pre-computation resistance” requirement, such that: 
- An attacker must perform an offline dictionary attack that cannot make use of any pre-computed table. 
Note that these requirements increase the burden on attackers, but do not stop attacks; once a server is compromised, an offline dictionary attack should be expected (one response is to update all passwords).
### TAXONOMY: PAKE DESIGN CLASSES
The main PAKE designs have used passwords in three ways:
1. as an encryption key
2. as an input string to derive a generator 
3. as an integer in modular arithmetic (in the exponent for a multiplicative group, or as a scalar in an additive group over an elliptic curve).
The third case includes protocols having different security properties depending on the design approach and formal analysis model (e.g., common reference string, random oracle).
##### a summary of major PAKE schemes
1. **Password used as encryption key**. This class includes using a password as the encryption key and typically must assume an ideal cipher.
2. **Password-derived generator**. A protocol group generator is derived from a password.
3. **Trusted setup**. The protocol relies on a trusted setup, which defines two (or more) generators whose discrete logarithm relationship must be unknown.
4. **Secure two-party computation**. Here PAKE is viewed as a two-party secure computation problem on an equality function; the use of a non-interactive zero-knowledge proof (ZKP)aims to check that parties follow a specification honestly.
5. **Password-derived exponent**. In this class, a password is used to derive $g^w$ as a verifier in a type of Diffie-Hellmankey exchange.

PAKE security proofs are generally constructed in one or more of three security models
- Ideal cipher (IC)
- Common reference string (CRS)
- Random oracle (RO)
### Reference
- [SoK: password-authenticated key exchange -- theory, practice, standardization, and real-world lessons](https://eprint.iacr.org/2021/1492)
- [Let's talk about PAKE](https://blog.cryptographyengineering.com/2018/10/19/lets-talk-about-pake/)