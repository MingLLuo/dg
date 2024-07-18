---
{"dg-publish":true,"permalink":"/distributed/zoo-keeper/","noteIcon":"","created":"2024-06-20T13:21:22.152+08:00","updated":"2024-06-26T13:19:22.866+08:00"}
---

#Distributed_System #ZooKeeper 
Zookeeper is a *coordination kernel* which can build new primitives(like distributed locks or leader election).
### Overview
ZooKeeper exposes a [[Distributed/Wait-free interface\|Wait-free interface]] and an [[Distributed/Event-driven mechanism\|Event-driven mechanism]] to provide a simple and high-performance kernel. 
Since it doesn’t use blocking primitives, slow clients do not negatively impact faster clients. 
- **Why Slow Clients Do Not Negatively Impact Faster Clients**
	1. **Non-blocking Operations**: Each client’s request is handled independently, ensuring that a slow client’s operation does not prevent a fast client from completing its own operations.
	2. **Local Read Processing**: Read operations in ZooKeeper are handled locally by each server without requiring coordination with other servers. This means that a client can quickly read data from its connected server without being affected by the read or write operations of other clients.
	3. **Efficient Write Propagation**: Write operations are coordinated by the leader node, which ensures that all state changes are linearizable. Once a write request is acknowledged by the leader, it is quickly propagated to other servers. This ensures that even if some clients are slower, the overall system remains responsive because write operations are promptly handled and synchronized across the system.
	4. **Event-driven Notifications**: Clients that need to be aware of changes set watches and receive notifications when changes occur. This event-driven approach allows clients to react to changes immediately without polling for updates continuously. Consequently, the system can efficiently handle a large number of clients with varying speeds, as slow clients simply receive notifications when they are ready to process them, without impacting the overall system performance.
The ZooKeeper API allows clients to manipulate wait-free data objects, called _znodes_, organized hierarchically as in a file system.

There are two types of znodes that a client can create:
- _Regular_ : Clients are responsible for creating and deleting regular znodes explicitly
- _Ephemeral_ : Clients create ephemeral znodes and may delete them explicitly. These znodes are also deleted by the system automatically when the session that creates them terminates.

Clients can additionally set a _sequential_ flag when creating a znode. Nodes created with the sequential flag set have the value of a monotonically increasing counter appended to its name.

ZooKeeper allows clients to keep _watches_ on znodes to receive notifications of changes without requiring polling. Watches are unregistered once triggered or once the session closes.

ZooKeeper guarantees *FIFO client ordering* of all operations and *linearizable writes*. These two guarantees along with the wait-free nature of the service enables an efficient implementation while being sufficient to implement coordination.

ZooKeeper uses a leader-based atomic broadcast protocol called _Zab_ to guarantee that update operations satisfy linearizability. Read operations are fulfilled locally, that is ZooKeeper does not use Zab to totally order them. This is favorable to scale read throughput and benefits applications whose workloads are dominated by reads.
### ZooKeeper Guarantees
#### Linearizable Writes
ZooKeeper guarantees that all requests that update the state are serializable and are delivered in order.
#### FIFO Client Order
> Guaranteeing FIFO client order enables clients to submit operations asynchronously. With asynchronous operations, a client is able to have multiple outstanding operations at a time
#### Client API
The ZooKeeper API resembles that of a file system. Some important functions in the API are:
- `create(path, data, flags)`  
    Creates a znode with path name `path`, stores `data[]` in it and returns name of the new znode. `flags` can be used to specify the type of znode (regular or ephemeral) and set the sequential flag.
- `delete(path, version)`  
    Deletes znode at `path` if the znode is at the expected version.
- `exists(path, watch)`  
    Returns true if the znode at path `path` exists. `watch` allows clients to set a watch on the znode to receive notification if the znode state changes. ZooKeeper sets the watch even if the znode does not exist allowing clients to be notified of znode creation.
- `getData(path, watch)`  
    Returns the data and meta-data associated with the znode. Unlike `exists()`, ZooKeeper does not set the watch if the znode does not exist.
- `setData(path, data, version)`  
    Writes `data[]` to znode `path` if the znode is at the expected version.
- `getChildren(path, watch)`  
    Returns names of the children of a znode
- `sync(path)`  
    Waits for all updates pending to propagate to the server that the client is connected to. `path` is ignored.

### Building coordination primitives using ZooKeeper
ZooKeeper can be used to build both blocking and non-blocking primitives efficiently by taking advantage of its ordering guarantees.
> ZooKeeper’s ordering guarantees allow efficient reasoning about system state, and watches allow for efficient waiting.
#### Configuration Management
> In its simplest form configuration is stored in a znode, zc. Processes start up with the full pathname of zc. Starting processes obtain their configuration by reading zc with the watch flag set to true. If the configuration in zc is ever updated, the processes are notified and read the new configuration, again setting the watch flag to true
#### Group Membership
This is relatively simple if _ephemeral_ znodes are used. A znode _zg_ represents the group. A process that is member of the group creates an ephemeral child znode under _zg_. After the creation, the process does not need to do anything else. If the process fails or terminates, the child znode representing it is automatically removed.

To obtain group information or watch for group membership changes, processes can simply list all children of _zg_ optionally setting the `watch` flag.
#### Simple Locks
Represent the lock by a designatd znode. To acquire a lock, a client tries to create the designated znode with the _ephemeral_ flag. The client holds the lock if the creation is successful. Otherwise, the client can `watch` the znode to be notified of a lock release. A client releases the lock when it dies or explicitly deletes the znode.
#### Simple Locks without Herd Effect
The above simple locking protocol works but suffers from _herd effect_. All clients waiting to acquire a lock vie for the lock when the lock is released, even though only one client can acquire the lock.

To get rid of this _herd effect_, we can order the client’s attempt to acquire the lock with respect to all other attempts using the _sequential_ flag when creating the lock znode. If the client’s znode has the lowest sequence number, the client holds the lock. Otherwise, the client watches the znode that precedes it. By only watching the znode that precedes the client’s znode, we avoid the herd effect by only waking up one process when a lock is released or a lock request is abandoned.
```java
public class SimpleLockWithoutHerdEffect {
    private ZooKeeper zk;
    private String lockPath;
    private String lockZnode;

    public SimpleLockWithoutHerdEffect(ZooKeeper zk, String lockPath) {
        this.zk = zk;
        this.lockPath = lockPath;
    }

    public boolean acquireLock() throws KeeperException, InterruptedException {
        // Step 1: Create a sequential, ephemeral znode
        lockZnode = zk.create(lockPath + "/lock-", new byte[0],                               ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL_SEQUENTIAL);
        
        // Step 2: Get the list of znodes under the lock path
        List<String> children = zk.getChildren(lockPath, false);
        Collections.sort(children);

        // Step 3: Find our position in the list
        String sequenceNode = lockZnode.substring(lockPath.length() + 1);
        int index = children.indexOf(sequenceNode);

        // Step 4: Check if we are the first node
        if (index == 0) {
            return true; // Acquired the lock
        }

        // Step 5: Watch the node just before us
        String previousNode = children.get(index - 1);
        Stat stat = zk.exists(lockPath + "/" + previousNode, event -> {
            if (event.getType() == EventType.NodeDeleted) {
                synchronized (this) {
                    notifyAll(); // Notify this thread to re-check lock condition
                }
            }
        });

        if (stat != null) {
            synchronized (this) {
                wait(); // Wait for notification of the previous node's deletion
            }
        }

        return true; // Acquired the lock after previous node deletion
    }

    public void releaseLock() throws KeeperException, InterruptedException {
        // Step 6: Delete our znode to release the lock
        if (lockZnode != null) {
            zk.delete(lockZnode, -1);
            lockZnode = null;
        }
    }
}
```

#### Read/Write Locks
To implement read/write locks, we differentiate between readers and writers. Writers must wait for all readers and other writers to release the lock, while readers can acquire the lock concurrently with other readers but not with writers.
```java
public class ReadWriteLock {
    private ZooKeeper zk;
    private String lockPath;
    private String readLockPath;
    private String writeLockPath;
    private String lockZnode;

    public ReadWriteLock(ZooKeeper zk, String lockPath) {
        this.zk = zk;
        this.lockPath = lockPath;
        this.readLockPath = lockPath + "/readers";
        this.writeLockPath = lockPath + "/writers";
    }

    public boolean acquireReadLock() throws KeeperException, InterruptedException {
        // Create a read lock node
        lockZnode = zk.create(readLockPath + "/read-", new byte[0], ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL_SEQUENTIAL);
        
        // Check if there are any writer nodes
        List<String> writers = zk.getChildren(writeLockPath, false);
        return writers.isEmpty(); // Acquired read lock if no writers
    }

    public boolean acquireWriteLock() throws KeeperException, InterruptedException {
        // Create a write lock node
        lockZnode = zk.create(writeLockPath + "/write-", new byte[0], ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL_SEQUENTIAL);
        
        // Check if there are any reader or other writer nodes
        List<String> readers = zk.getChildren(readLockPath, false);
        List<String> writers = zk.getChildren(writeLockPath, false);

        return readers.isEmpty() && writers.size() == 1; // Acquired write lock if no readers and only this writer
    }

    public void releaseLock() throws KeeperException, InterruptedException {
        // Delete our znode to release the lock
        if (lockZnode != null) {
            zk.delete(lockZnode, -1);
            lockZnode = null;
        }
    }
}
```

### Architecture
Zookeeper maintains an [[Distributed/In-memory database\|In-memory database]] containing the entire data tree. Updates are efficiently logged to disk before being applied to the in-memory database.

Every Zookeeper server serves clients. Servers first prepare requests for execution in the Request Processor. Read requests are serviced from the local replica of each server database while write requests are processed using an agreement protocol (atomic broadcast).

By processing reads locally, ZooKeeper obtains excellent read performance because it is just an in-memory operation on the local server, and there is no disk activity or agreement protocol to run.

Write requests are forwarded to a single server (leader) and the rest of the servers (followers) receive message proposals for state changes from the leader and agree upon state changes.

#### Request Processor
*Request processor* transforms a client request, that changes state, into a transaction. 
Unlike client requests, transactions are **[[Distributed/Idempotent Operations\|Idempotent Operations]]**. On receiving a write request, the leader calculates what the state of the system will be when the write is applied and transforms it into a transaction that captures this new state.

Zookeeper guarantees that the local replicas never diverge, however, it is possible that at any point in time some replicas may have applied more transactions than others.
#### Atomic Broadcast
ZooKeeper uses *Zab*, an atomic broadcast protocol. Requests that update state are forwarded to the leader which then executes the request and broadcasts the change to followers through Zab. Response to the client request is sent when the corresponding state change is delivered by the server that received the request.

Since Zab uses simple majority quorums to decide on a proposal, ZooKeeper can only work if a majority of servers are healthy.

Zab guarantees that changes broadcast by a leader are delivered in the order that they are sent and all changes from previous leaders are delivered to an established leader before it broadcasts its own changes.

Message order is handled by **TCP**, simplifying the Zab implementation. 
Zab may redeliver a message during recovery and ZooKeeper relies on Zab to redeliver at least all messages that were delivered after the start of the last snapshot. 
Note that multiple deliveries are acceptable as long as they are delivered in order because of the idempotent nature of the transactions.
#### Snapshots and Recovery
ZooKeeper saves periodic snapshots of the state and thus, only requires redelivery of messages since the start of the snapshot. This helps a recovering ZooKeeper server to recover its internal state relatively quickly since it *doesn’t have to replay all delivered messages* since the beginning.

ZooKeeper snapshots are fuzzy since the ZooKeeper state is not locked to take the snapshot. Every znode is read atomically and written to the disk as part of the snapshot. The resulting snapshot may have applied some subset of the state changes delivered during the generation of the snapshot. *This snapshot may not correspond to the state of ZooKeeper at any point in time.*

This is not a problem because state changes are idempotent. Applying state changes more than once is fine as long as the state changes are applied in order. A server that recovers with a snapshot that does not correspond to a valid ZooKeeper state can still reconstruct the state because Zab redelivers the state changes to the server.

###  Reference
- [ZooKeeper: Wait-free coordination for Internet-scale systems](https://www.siddharthjain.dev/posts/2020/zookeeper-wait-free-coordination-for-internet-scale-systems/)
- [USENIX Paper on ZooKeeper: Wait-free Coordination for Internet-scale Systems](https://www.usenix.org/conference/usenix-atc-10/zookeeper-wait-free-coordination-internet-scale-systems)
- [ZooKeeper Documentation](https://zookeeper.apache.org/doc/current/)