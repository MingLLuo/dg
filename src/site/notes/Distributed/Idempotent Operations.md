---
{"dg-publish":true,"permalink":"/distributed/idempotent-operations/","noteIcon":"","created":"2024-06-20T14:06:19.423+08:00","updated":"2024-06-25T12:26:38.512+08:00"}
---

#ZooKeeper 
An **idempotent** operation is one that can be applied multiple times without changing the result beyond the initial application. In other words, performing the operation once or many times yields the same outcome.

Example:
- Addition of zero:  $x + 0$  remains $x$  no matter how many times the addition is performed.
- Setting a value: Setting a variable to a specific value (e.g., $x = 5$) is idempotent because regardless of how many times you assign $x = 5$, the result is always the same:  $x$  becomes $5$.
