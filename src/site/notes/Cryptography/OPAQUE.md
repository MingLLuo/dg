---
{"dg-publish":true,"permalink":"/cryptography/opaque/","created":"2024-07-13T12:15:18.574+08:00","updated":"2024-07-16T19:27:48.493+08:00"}
---

> Following content all based in OPAQUE Draft, and explained in chinese.


The name OPAQUE is a homonym of O-PAKE where O is for Oblivious.

- OPAQUE是一种增强的/非对称的PAKE协议，支持client和sercer的相互验证，而不依赖于PKI（客户端注册期间除外），且拥有抵御预计算攻击的安全性。此外，还提供前向安全与隐藏明文口令的能力，包括口令注册的阶段
- 其中有三个主要的功能构件，分别是 an oblivious pseudorandom function (OPRF), a key recovery mechanism, and an authenticated key exchange (AKE) protocol
- OPAQUE被分为两个阶段，注册和AKE
	- 客户端向服务器注册其密码，并在服务器上存储用于恢复身份验证凭据(authentication credentials)的信息
	- 客户端使用其密码来恢复这些凭据，然后将它们用作 AKE 协议的输入。此阶段有额外的机制来防止主动攻击者与服务器交互以猜测或确认通过第一阶段注册的客户端。
- OPAQUE依赖如下的加密协议与原语
	- [[Cryptography/Oblivious Pseudorandom Function (OPRF)\|Oblivious Pseudorandom Function (OPRF)]]
	- [[Key Derivation Function (KDF)\|Key Derivation Function (KDF)]]
	- [[Message Authentication Code (MAC)\|Message Authentication Code (MAC)]]
	- Cryptographic Hash Function
	- [[Key Stretching Function (KSF)\|Key Stretching Function (KSF)]]
[The OPAQUE Augmented PAKE Protocol](https://github.com/cfrg/draft-irtf-cfrg-opaque)

