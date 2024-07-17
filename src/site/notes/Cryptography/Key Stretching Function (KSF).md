---
{"dg-publish":true,"permalink":"/cryptography/key-stretching-function-ksf/","created":"2024-07-16T21:58:47.053+08:00","updated":"2024-07-16T22:01:21.401+08:00"}
---

Key Stretching Function (KSF) is *a slow and expensive cryptographic hash function* with the following API
- Stretch(msg): Apply a key stretching function to stretch the input `msg` and harden it against offline dictionary attacks. This function also needs to satisfy collision resistance.