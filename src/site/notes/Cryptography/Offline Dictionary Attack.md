---
{"dg-publish":true,"permalink":"/cryptography/offline-dictionary-attack/","noteIcon":"","created":"2024-06-26T13:30:36.974+08:00","updated":"2024-07-07T15:38:17.847+08:00"}
---


#Cryptography

A dictionary attack may be online or offline. An online dictionary attack is performed by trying each password in a separate request to the KDC(**Key Distribution Center**), and is therefore visible to the KDC and also limited in speed by the KDC’s processing power and the network capacity between the client and the KDC. 

*Online dictionary attacks* can be mitigated using [account lockout](https://web.mit.edu/kerberos/krb5-latest/doc/admin/lockout.html#lockout). This measure is not totally satisfactory, as it makes it easy for an attacker to deny access to a client principal.

*An offline dictionary attack* is performed by obtaining a ciphertext generated using the password-derived key, and trying each password against the ciphertext. This category of attack is invisible to the KDC and can be performed much faster than an online attack. The attack will generally take much longer with more recent encryption types (particularly the ones based on AES), because those encryption types use a much more expensive string-to-key function. However, the best defense is to deny the attacker access to a useful ciphertext. The required defensive measures depend on the attacker’s level of network access.

If [PKINIT](https://web.mit.edu/kerberos/krb5-latest/doc/admin/pkinit.html#pkinit) or [OTP](https://web.mit.edu/kerberos/krb5-latest/doc/admin/otp.html#otp-preauth) are used for initial authentication, the principal’s long-term keys are not used and dictionary attacks are usually not a concern.

#### Diff: Brute-force and dictionary attacks
A [brute-force attack](https://nordvpn.com/blog/brute-force-attack/) and a dictionary attack are both designed to guess your password, but the methods they use are different. While a dictionary attack makes use of a prearranged list of words, a brute-force attack tries every possible combination of letters, special symbols, and numbers. It can guess a six-character password in one hour. If your password is long and complex, it will take days or even years to crack it.

A brute-force attack doesn’t necessarily try every possible character. Password-cracking software can be programmed to start with more likely options. If there is a requirement to use an uppercase letter in the password, most people will use it in the first character. Knowing this, hackers can set the program to start with a capital letter as the first character. A brute-force attack takes longer to crack a password than a dictionary attack does and heavily relies on computing power.
### Reference
- [Addressing dictionary attack risks](https://web.mit.edu/kerberos/krb5-latest/doc/admin/dictionary.html)
- [Kerberos (protocol)](https://en.wikipedia.org/wiki/Kerberos_(protocol))
- [What is a dictionary attack and how can you prevent it?](https://nordvpn.com/blog/dictionary-attack/)