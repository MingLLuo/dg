---
{"dg-publish":true,"permalink":"/privacy-budget/","created":"2024-06-25T10:16:25.392+08:00","updated":"2024-06-25T12:26:57.413+08:00"}
---

#Differential_Privacy 
The Apple differential privacy implementation incorporates the concept of a **perdonation privacy budget** (quantified by the parameter epsilon), and sets a strict limit on the number of contributions from a user in order to preserve their privacy. 
The reason is that the slightly-biased noise used in differential privacy tends to average out over a large numbers of contributions, making it theoretically possible to determine information about a user’s activity over a large number of observations from a single user.

The concept of a **“privacy budget”** is used to measure and control the amount of privacy loss incurred by data queries. This budget is primarily defined by the parameters $\epsilon$ (epsilon) and $\delta$ (delta). Setting these parameters correctly is crucial for balancing privacy protection and the utility of the query results.

#### $\epsilon$ and $\delta$ Parameters
1.	$\epsilon$ (epsilon): This parameter measures the degree of privacy loss. A smaller $\epsilon$ value means stronger privacy protection, but it may result in less accurate query results. Conversely, a larger $\epsilon$ value indicates **weaker privacy protection**, but **more accurate results**.
2.	$\delta$ (delta): This parameter represents the probability that the algorithm fails to provide $\epsilon$-differential privacy. A smaller $\delta$ value means a **stronger privacy guarantee**.
#### High Privacy Budget (larger $\epsilon$):
- Greater Privacy Leakage: When the privacy budget is set high, it implies that less noise is added to the query results. This makes the results more accurate and closer to the real data. However, it also makes it easier for an adversary to infer the original data, leading to greater privacy leakage.
- More Accurate Results: With less noise added, the query results are more accurate, providing a better reflection of the true statistics of the data.