---
{"dg-publish":true,"permalink":"/distributed/wait-free-interface/","created":"2024-06-20T13:33:31.102+08:00","updated":"2024-06-25T12:05:37.960+08:00"}
---

#ZooKeeper 
The term **wait-free** indicates that the system guarantees a result within a bounded number of steps, avoiding indefinite delays.
A **wait-free interface** is one where operations can be completed in a finite number of steps, regardless of the actions of other threads or processes. 
Each client can complete its operations without waiting for other clients to finish their tasks. 
In the context of [[Distributed/ZooKeeper\|ZooKeeper]], this ensures that each client can perform its tasks independently, providing a high level of responsiveness and reliability. 