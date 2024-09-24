---
{"dg-publish":true,"permalink":"/cryptography/secret-sharing/","noteIcon":"","created":"2024-09-24T21:31:06.984+08:00","updated":"2024-09-24T22:06:00.089+08:00"}
---

- Any $SS$ protocal for a set of $n$ parties $P = \{P_{1}, \ldots,P_{n}\}$ consists two phases, a sharing phase and a reconstruction phase.
- We denote $\text{view}_i$ the view of $P_i$ at the end of the protocal, which consists of the local inputs of $P_i$, along with the messages sent and received by $P_i$.
- Each party $P_i$ will output a share $s_i$, which is a publicly-known function of $view_i$
- Each party then applies a publicly-known reconstruction function on the revealed views, as determined by the protocol Rec and reconstructs some output.
- Properties needed from any $SS$ protocal,
	- *Correctness*
		- Any *qualified subset* of parties should be able to reconstruct back the shared secret by executing the protocal Rec.(also called *access-sets*, the collection of them is called *access-structure*)
		- Usually use minimal access-sets.
	- *Privacy*
		- Informally demands that the view of any "forbidden subset" of parties should not reveal any information about the shared secret.
		- Usually use maximal forbidden-set.
![Secret-Sharing-Def.png](/img/user/Cryptography/attachments/Secret-Sharing-Def.png)