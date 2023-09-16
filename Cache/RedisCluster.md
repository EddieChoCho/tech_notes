# Redis Cluster

## Redis cluster specification

* Information about the algorithms and design rationales of Redis Cluster.
* What is described in this document is implemented in Redis 3.0 or greater.

### Main properties and rationales of the design

#### Redis Cluster goals

* High performance and linear scalability up to 1000 nodes.
    * There are no proxies, asynchronous replication is used, and no merge operations are performed on values.

* Acceptable degree of write safety
    * The system tries (in a best-effort way) to retain all the writes originating from clients connected with the
      majority of the master nodes.
    * Usually there are small windows where acknowledged writes can be lost.
    * Windows to lose acknowledged writes are larger when clients are in a minority partition.

* Availability
    * Redis Cluster is able to survive partitions where the majority of the master nodes are reachable and there is at
      least one reachable replica for every master node that is no longer reachable.
    * Moreover using replicas migration, masters no longer replicated by any replica will receive one from a master
      which is covered by multiple replicas.

#### Implemented subset

* Redis Cluster implements all the single key commands available in the non-distributed version of Redis.
* Commands performing complex multi-key operations like set unions and intersections are implemented for cases where all
  of the keys involved in the operation hash to the same slot.

* Redis Cluster implements a concept called hash tags that can be used to force certain keys to be stored in the same
  hash slot.
* However, during manual resharding, multi-key operations may become unavailable for some time while single-key
  operations are always available.

### Client and Server roles in the Redis cluster protocol

* In Redis Cluster, nodes are responsible for holding the data, and taking the state of the cluster, including mapping
  keys to the right nodes.
* Cluster nodes are also able to auto-discover other nodes, detect non-working nodes, and promote replica nodes to
  master when needed in order to continue to operate when a failure occurs.

* Redis Cluster Bus
    * All the cluster nodes are connected using a TCP bus and a binary protocol, called the Redis Cluster Bus.
    * Every node is connected to every other node in the cluster using the cluster bus.
    * Nodes use a gossip protocol to
        * Propagate information about the cluster in order to discover new nodes.
        * Send ping packets to make sure all the other nodes are working properly.
        * Send cluster messages needed to signal specific conditions.
    * The cluster bus is also used in order to
        * Propagate Pub/Sub messages across the cluster.
        * Orchestrate manual failovers when requested by users (manual failovers are failovers which are not initiated
          by the Redis Cluster failure detector, but by the system administrator directly).

* Since cluster nodes are not able to proxy requests, clients may be redirected to other nodes using redirection errors
  -MOVED and -ASK.
* The client is in theory free to send requests to all the nodes in the cluster, getting redirected if needed, so the
  client is not required to hold the state of the cluster.
* However clients that are able to cache the map between keys and nodes can improve the performance in a sensible way.

#### Write safety

* Redis Cluster uses asynchronous replication between nodes, and `last failover wins` implicit merge function.
    * This means that the last elected master dataset eventually replaces all the other replicas.
    * There is always a window of time when it is possible to lose writes during partitions.
    * However these windows are very different in the case of a client that is connected to the majority of masters, and
      a client that is connected to the minority of masters.

* TBD...

## Scaling with Redis Cluster

* This topic will teach you how to set up, test, and operate Redis Cluster in production.
* You will learn about the availability and consistency characteristics of Redis Cluster from the end user's point of
  view.

### Redis Cluster 101

* Redis Cluster provides a way to run a Redis installation where data is automatically sharded across multiple Redis
  nodes.
* Redis Cluster also provides some degree of availability during partitions—in practical terms, the ability to continue
  operations when some nodes fail or are unable to communicate.
* However, the cluster will become unavailable in the event of larger failures (for example, when the majority of
  masters are unavailable).
* With Redis Cluster, you get the ability to:
    * Automatically split your dataset among multiple nodes.
    * Continue operations when a subset of the nodes are experiencing failures or are unable to communicate with the
      rest of the cluster.

#### Redis Cluster TCP ports

* Every Redis Cluster node requires two open TCP connections:
    * A Redis TCP port used to serve clients, e.g., 6379,
    * And second port known as the cluster bus port. By default, the cluster bus port is set by adding 10000 to the data
      port (e.g., 16379);
        * However, you can override this in the `cluster-port` configuration.

* Cluster bus is a node-to-node communication channel that uses a binary protocol, which is more suited to exchanging
  information between nodes due to little bandwidth and processing time.
* Nodes use the cluster bus for failure detection, configuration updates, failover authorization, and so forth.
* Clients should never try to communicate with the cluster bus port, but rather use the Redis command port.
* However, make sure you open both ports in your firewall, otherwise Redis cluster nodes won't be able to communicate.

#### Redis Cluster and Docker

* Redis Cluster does not support NATted environments and in general environments where IP addresses or TCP ports are
  remapped.
* Docker uses a technique called port mapping
    * Programs running inside Docker containers may be exposed with a different port compared to the one the program
      believes to be using.
    * This is useful for running multiple containers using the same ports, at the same time, in the same server.

* To make Docker compatible with Redis Cluster, you need to use Docker's host networking mode.(`--net=host` option in
  the Docker documentation).

#### Redis Cluster data sharding

* Redis Cluster does not use consistent hashing, but a different form of sharding where every key is conceptually part
  of what we call a `hash slot`.
* There are 16384 hash slots in Redis Cluster, and to compute the hash slot for a given key, we simply take the CRC16 of
  the key modulo 16384.
* For example, you may have a cluster with 3 nodes, where:
    * Node A contains hash slots from 0 to 5500.
    * Node B contains hash slots from 5501 to 11000.
    * Node C contains hash slots from 11001 to 16383.

* This makes it easy to add and remove cluster nodes.
    * If I want to add a new node D, I need to move some hash slots from nodes A, B, C to D.
    * If I want to remove node A from the cluster, I can just move the hash slots served by A to B and C. Once node A is
      empty, I can remove it from the cluster completely.

* Moving hash slots from a node to another does not require stopping any operations; therefore, adding and removing
  nodes, or changing the percentage of hash slots held by a node, requires no downtime.
* Redis Cluster supports multiple key operations as long as all of the keys involved in a single command execution (or
  whole transaction, or Lua script execution) belong to the same hash slot.
* Hash tags
    * User can use a feature, hash tags, to force multiple keys to be part of the same hash slot.
    * If there is a substring between {} brackets in a key, only what is inside the string is hashed.
    * For example, the keys user:{123}:profile and user:{123}:account are guaranteed to be in the same hash slot because
      they share the same hash tag.
    * As a result, you can operate on these two keys in the same multi-key operation.

#### Redis Cluster master-replica model

* To remain available when a subset of master nodes are failing or are not able to communicate with the majority of
  nodes, Redis Cluster uses a master-replica model where every hash slot has from 1 (the master itself) to N replicas (
  N-1 additional replica nodes).
* In our example cluster with nodes A, B, C, if node B fails the cluster is not able to continue, since we no longer
  have a way to serve hash slots in the range 5501-11000.
* However, when the cluster is created (or at a later time), we add a replica node to every master, so that the final
  cluster is composed of A, B, C that are master nodes, and A1, B1, C1 that are replica nodes. This way, the system can
  continue if node B fails.
* Node B1 replicates B, and B fails, the cluster will promote node B1 as the new master and will continue to operate
  correctly.
* However, note that if nodes B and B1 fail at the same time, Redis Cluster will not be able to continue to operate.

#### Redis Cluster consistency guarantees

* Redis Cluster does `not guarantee strong consistency`. It is possible that Redis Cluster will lose writes that were
  acknowledged by the system to the client.
* Losing writes because it uses asynchronous replication:
    * During writes the following happens
        * Your client writes to the master B.
        * The master B replies OK to your client.
        * The master B propagates the write to its replicas B1, B2 and B3.
    * B does not wait for an acknowledgement from B1, B2, B3 before replying to the client, since this would be a
      prohibitive latency penalty for Redis.
    * B acknowledges the write, but crashes before being able to send the write to its replicas, one of the replicas (
      that did not receive the write) can be promoted to master, losing the write forever.
        * Basically, there is a trade-off to be made between performance and consistency.
    * WAIT command
        * By using WAIT command, Redis Cluster has support for synchronous writes when absolutely needed.
        * However, note that Redis Cluster does not implement strong consistency even when synchronous replication is
          used.
            * It is always possible, under more complex failure scenarios, that a replica that was not able to receive
              the write will be elected as master.

* Losing writes during a network partition where a client is isolated with a minority of instances including at least a
  master:
    * Take as an example our 6 nodes cluster composed of A, B, C, A1, B1, C1, with 3 masters and 3 replicas. There is
      also a client, that we will call Z1.
    * After a partition occurs, it is possible that in one side of the partition we have A, C, A1, B1, C1, and in the
      other side we have B and Z1.
    * Z1 is still able to write to B, which will accept its writes. If the partition heals in a very short time, the
      cluster will continue normally.
    * However, if the partition lasts enough time for B1 to be promoted to master on the majority side of the partition,
      the writes that Z1 has sent to B in the meantime will be lost.
    * This amount of time is a very important configuration directive of Redis Cluster, and is called the `node timeout`
      .
        * After node timeout has elapsed, a master node is considered to be failing, and can be replaced by one of its
          replicas.
        * After node timeout has elapsed without a master node to be able to sense the majority of the other master
          nodes, it enters an error state and stops accepting writes.


* TBD...

## References

* [Redis cluster specification](https://redis.io/docs/reference/cluster-spec/)
* [Scaling with Redis Cluster](https://redis.io/docs/management/scaling/)