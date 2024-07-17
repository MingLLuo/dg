---
{"dg-publish":true,"permalink":"/cryptography/message-authentication-code-mac/","created":"2024-07-16T21:56:22.369+08:00","updated":"2024-07-16T21:56:53.058+08:00"}
---

#Cryptography #MAC
The API and parameters for the random-key robust MAC are as follows:
- MAC(key, msg): Compute a message authentication code over input `msg` with key `key`, producing a fixed-length output of `Nm` bytes.
- Nm: The output size of the `MAC()` function in bytes.