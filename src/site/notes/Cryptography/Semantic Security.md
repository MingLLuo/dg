---
{"dg-publish":true,"dg-permalink":"ss","permalink":"/ss/","noteIcon":"","created":"2024-06-18T16:58:52.144+08:00","updated":"2024-06-18T21:37:20.887+08:00"}
---

#Cryptography  #Simulation
## A small history tour
- How do we formulate the notion that *nothing is learned about the plaintext from the ciphertext*, as we said 'nothing is learned'?
	- If we try to say that an adversary who receives a ciphertext cannot output any information about the plaintext, then what happens if the adversary already has information about the plaintext? Adv may know it is the English text.
- The simulation-based formulation of security enables us to exactly formalize this.
	- We say that an encryption scheme is secure if the only information derived (or output by the adversary) is that which is based on a priori knowledge.
	- If the adversary receiving no ciphertext is able to output the same information as the adversary receiving the ciphertext, then this is indeed the case.
#### Definition
- allows the length of the plaintext to depend on the security parameter
- allows for arbitrary distributions over plaintexts (as long as the plaintexts sampled are of polynomial length).
- takes into account an **arbitrary auxiliary information function** $h$ of the plaintext that may be leaked to the adversary through other means (e.g., because the same message $x$ is used for some other purpose as well)
- The adversary aims to **learn some function** $f$ of the plaintext, from the ciphertext, and the provided auxiliary information. 
	- According to the definition, it should be possible to learn the same information from the auxiliary information alone (and from the length of the plaintext), and without the ciphertext

**Definition**: A private-key encryption scheme $(G, E, D)$ is semantically secure (in the private-key model) 
- if for every non-uniform probabilistic-polynomial time algorithm $A$ 
- there exists a non-uniform probabilistic-polynomial time algorithm $A'$ such that for
	- every probability ensemble $\{X_n\}_{n \in \mathbb{N}}$ with $|X_n| \leq \text{poly}(n)$, 
	- every pair of polynomially-bounded functions $f, h : \{0, 1\}^* \rightarrow \{0, 1\}^*$, 
	- every positive polynomial $p(\cdot)$ and all sufficiently large $n$:
$$
\Pr_{k \leftarrow G(1^n)} \left[ A(1^n, E_k(X_n), 1^{|X_n|}, h(1^n, X_n)) = f(1^n, X_n) \right] < \Pr \left[ A'(1^n, 1^{|X_n|}, h(1^n, X_n)) = f(1^n, X_n) \right] + \frac{1}{p(n)}
$$

(The probability in the above terms is taken over $X_n$ as well as over the internal coin tosses of the algorithms $G, E$ and $A$ or $A'$.)

Observe that the **adversary** $A$ is given the ciphertext $E_k(X_n)$ as well as auxiliary information $h(1^n, X_n)$, and attempts to guess the value of $f(1^n, X_n)$. 
Algorithm $A'$ also attempts to guess the value of $f(1^n, X_n)$, but is given only $h(1^n, X_n)$ and the length of $X_n$. The security requirement states that $A'$ can correctly guess $f(1^n, X_n)$ with almost the same probability as $A$. Intuitively, then, the ciphertext $E_k(X_n)$ does not reveal any information about $f(1^n, X_n)$, for any $f$, since whatever can be learned by $A$ (given the ciphertext) can be learned by $A'$ (without being given the ciphertext).

#### A simulation view of the above definition
- $A'$ stay in an ideal world where anything it learns is from auxiliary information and plaintext length only.
##### Simulator $A'$: 
Upon input $1^n, 1^{|X_n|}, h = h(1^n, X_n)$, algorithm $A'$ works as follows:
1. $A'$ runs the key generation algorithm $G(1^n)$ in order to receive $k$ (note that $A'$ indeed needs to be given $1^n$ in order to do this).
2. $A'$ computes $c = E_k(0^{|X_n|})$ as an encryption of “garbage” (note that $A'$ indeed needs to be given $1^{|X_n|}$ in order to do this).
3. $A'$ runs $A(1^n, c, 1^{|X_n|}, h)$ and outputs whatever $A$ outputs.

## from BS23
### Attack Game 2.1 (semantic security)
For a given cipher $\mathcal{E} = (E, D)$, defined over $(\mathcal{K}, \mathcal{M}, \mathcal{C})$, and for a given adversary $\mathcal{A}$, we define two experiments, Experiment 0 and Experiment 1. For $b = 0, 1$, we define
#### Experiment $b$:
- The **adversary** computes $m_0, m_1 \in \mathcal{M}$, of the same length, and sends them to the challenger.
- The **challenger** computes $k \leftarrow_R \mathcal{K}$, $c \leftarrow_R E(k, m_b)$, and sends $c$ to the adversary.
- The **adversary** outputs a bit $\hat{b} \in \{0, 1\}$.
For $b = 0, 1$, let $W_b$ be the event that $\mathcal{A}$ outputs 1 in Experiment $b$. We define $\mathcal{A}$'s semantic security advantage with respect to $\mathcal{E}$ as
$$
\text{SSadv}[\mathcal{A}, \mathcal{E}] := \left| \Pr[W_0] - \Pr[W_1] \right|.
$$
### Attack Game 2.4 (semantic security: bit-guessing version)\
For a given cipher $\mathcal{E} = (E, D)$, defined over $(\mathcal{K}, \mathcal{M}, \mathcal{C})$, and for a given adversary $\mathcal{A}$, the attack game runs as follows:
- The **adversary** computes $m_0, m_1 \in \mathcal{M}$, of the same length, and sends them to the challenger.
- The **challenger** computes $b \leftarrow_R \{0, 1\}$, $k \leftarrow_R \mathcal{K}$, $c \leftarrow_R E(k, m_b)$, and sends $c$ to the adversary.
- The **adversary** outputs a bit $\hat{b} \in \{0, 1\}$.
We say that $\mathcal{A}$ wins the game if $\hat{b} = b$. 
---
### Definition
A cipher $\mathcal{E}$ is **semantically secure** if for all efficient adversaries $\mathcal{A}$, the value $\text{SSadv}[\mathcal{A}, \mathcal{E}]$ is negligible.

What we are interested in is **how much better than random guessing an adversary can do**. 
If $W$ denotes the event that the adversary wins the bit-guessing version of the attack game, then we are interested in the quantity $|\Pr[W] - 1/2|$, which we denote by $\text{SSadv}^*[\mathcal{A}, \mathcal{E}]$. 

**Theorem 2.10.** For every cipher $\mathcal{E}$ and every adversary $\mathcal{A}$, we have
$$
\text{SSadv}[\mathcal{A}, \mathcal{E}] = 2 \cdot \text{SSadv}^*[\mathcal{A}, \mathcal{E}].
$$

**Proof.**
- $p_i$ be the probability that the adversary outputs 1 in Experiment i of Attack Game 2.1
Consider Attack Game 2.4. All events and probabilities are with respect to this game. 

If we condition on the event that $b = 0$, then in this *conditional probability space*, all of the other random choices made by the challenger and the adversary are distributed in *exactly the same way as* the corresponding values in Experiment 0 of Attack Game 2.1. 
Therefore, if $\hat{b}$ is the output of the adversary in Attack Game 2.4, we have
$$
\Pr[\hat{b} = 1 \mid b = 0] = p_0.
$$
By a similar argument, we see that
$$
\Pr[\hat{b} = 1 \mid b = 1] = p_1.
$$

So we have
$$
\Pr[\hat{b} = b]
= \Pr[\hat{b} = b \mid b = 0] \Pr[b = 0] + \Pr[\hat{b} = b \mid b = 1] \Pr[b = 1] 
$$
$$= \Pr[b = 0 \mid b = 0] \cdot \frac{1}{2} + \Pr[\hat{b} = 1 \mid b = 1] \cdot \frac{1}{2} 
= \frac{1}{2} \left(1 - \Pr[\hat{b} = 1 \mid b = 0] + \Pr[\hat{b} = 1 \mid b = 1]\right) = \frac{1}{2} (1 - p_0 + p_1).
$$
Therefore,
$$
\text{SSadv}^*[\mathcal{A}, \mathcal{E}] = \left|\Pr[\hat{b} = b] - \frac{1}{2}\right| = \frac{1}{2} |p_1 - p_0| = \frac{1}{2} \cdot \text{SSadv}[\mathcal{A}, \mathcal{E}].
$$

That proves the theorem. $\square$
## Reference
- BS23 Chap2.2
- Lindell 2016 How To Simulate it