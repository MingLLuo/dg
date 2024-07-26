---
{"dg-publish":true,"permalink":"/math/np/","noteIcon":"","created":"2024-07-25T19:02:37.952+08:00","updated":"2024-07-25T19:05:29.303+08:00"}
---

#TCS
**NP** is the class of decision problems for which there is a polynomial time algorithm that can **verify** "yes" instances given the appropriate certificate.

**CoNP** is the class of decision problems for which there is a polynomial time algorithm that can **verify** "no" instances given the appropriate certificate.

We don’t know whether coNP is different from NP.

There is a problem in NP for every problem in coNP, and vice versa. For example, the SAT problem asks "does there exist a boolean assignment which makes this formula evaluate to True?". The complement problem, which is in coNP, asks, "do all boolean assignments make this formula evaluate to False?"

A language 𝐿 is _NP-hard_ if for every language 𝑅 in NP there exists a function 𝑓 computable in polynomial time such that for all 𝑥, 𝑥∈𝑅 iff 𝑓(𝑥)∈𝐿.

If a language is both NP-hard and coNP-hard then its exact complexity lies above both NP and coNP. Indeed, it is conjectured that NP≠coNP, and this implies that a problem which is both NP-hard and coNP-hard belongs to neither NP nor coNP.

Problems which are both NP-hard and coNP-hard do exist. For example, any PSPACE-complete problem is both NP-hard and coNP-hard.