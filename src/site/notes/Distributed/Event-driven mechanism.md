---
{"dg-publish":true,"permalink":"/distributed/event-driven-mechanism/","created":"2024-06-20T13:35:32.749+08:00","updated":"2024-06-25T12:05:21.783+08:00"}
---

#ZooKeeper
An **event-driven mechanism** involves executing actions in response to specific events or changes in the state of the system. 

In [[Distributed/ZooKeeper\|ZooKeeper]], this is achieved through the use of “watches” and notifications. 
For example, clients can set watches on znodes (ZooKeeper nodes) to be notified of changes, such as data updates or deletions. When the state of a znode changes, ZooKeeper sends notifications to the clients that have set watches on that znode, allowing them to react accordingly. 

This mechanism is similar to the event-driven approaches used in GUI programming, where user actions (like clicks) trigger specific functions.