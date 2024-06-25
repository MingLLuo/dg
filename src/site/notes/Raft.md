---
{"dg-publish":true,"permalink":"/raft/","created":"2024-06-24T12:49:36.476+08:00","updated":"2024-06-25T12:06:25.734+08:00"}
---

#Distributed_System #Raft
Raft is a consensus algorithm designed to manage a replicated log. It’s used to ensure that a cluster of machines can work together in a reliable and fault-tolerant manner, providing strong consistency even in the presence of failures.
### Roles of Raft
1. **Leader**: The central node that handles all client requests, replicates log entries to followers, and manages the cluster.
2. **Follower**: Passive nodes that respond to requests from the leader and apply log entries.
3. **Candidate**: Nodes that attempt to become leaders during an election.

### Key components
1. **Leader Election**: Ensures that one and only one leader is elected among the cluster nodes.
2. **Log Replication**: Ensures that logs are replicated consistently across all nodes.
3. **Safety**: Ensures that the system behaves correctly and consistently, even in the presence of failures.
4. **Membership Changes**: Manages changes to the cluster’s membership dynamically.
#### Leader Election
- **Election Timer**: Each node starts as a follower and waits for a heartbeat from the leader. If a follower does not receive a heartbeat within a certain timeout, it becomes a candidate and starts an election.
- **Voting**: Candidates request votes from other nodes. A node votes for the first candidate it receives a request from, resetting its election timer.
- **Winning the Election**: A candidate becomes a leader if it receives a majority of votes. The new leader then sends heartbeats to all followers to assert its leadership.
#### Log Replication
- **Append Entries**: The leader appends new log entries from clients to its log and sends these entries to its followers.
- **Commitment**: A log entry is committed once the leader has successfully replicated it to a majority of the cluster nodes.
- **Consistency**: Followers apply committed entries to their state machines in the order they were appended.

#### Safety
- **Term Number**: Each node maintains a term number that increases with each election. This term number helps maintain a consistent state across the cluster.
- **Log Matching**: Logs are replicated in the same order across all nodes, ensuring that the system remains consistent.
- **Leader Completeness**: The leader always contains all committed entries from previous terms.
#### Membership Changes
- **Configuration Changes**: Raft supports changes to the cluster configuration (adding or removing nodes) through a two-phase commit process to ensure safety and consistency during transitions.

### Algorithm in Detail
1. **Election Process**:
- *Followers* wait for a timeout period without receiving a heartbeat.
- The *follower* becomes a *candidate* and increments its term number.
- The *candidate* requests votes from other nodes.
- If the *candidate* receives a majority of votes, it becomes the *leader*.
- The *leader* sends heartbeats to *followers* to maintain authority.

2. **Log Replication Process**:
- The *leader* receives a command from the client and appends it to its log.
- The *leader* sends an AppendEntries RPC to *followers*.
- *Followers* append the entry to their logs and send an acknowledgment back.
- Once the *leader* receives a majority of acknowledgments, it commits the entry.
- The leader notifies *followers* of the committed entries.
- *Followers* apply the committed entries to their state machines.

3. **Safety and Consistency**:
- Logs are always ordered by term and index.
- Entries are only committed if they are replicated on a majority of nodes.
- *Followers* only accept entries from the current term or previous terms if they match their log’s previous entries.

### Reference
- [The Raft Consensus Algorithm](https://raft.github.io/)