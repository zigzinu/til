# Redis Cluster

- 출처: https://redis.io/topics/cluster-tutorial

**Redis Cluster**
- Redis Cluster is a mix between *query routing* and *client side partitioning*
- Redis Cluster provides a way to run a Redis installation where data is automatically sharded across multiple Redis nodes.
  - You get the ability to automatically split your dataset among multiple nodes.
- Redis Cluster provides some degree of availability during partitions. That is the ability to continue the operations when some nodes fail or are not able to communicate.

**TCP ports**
- Normal port (**6379**)
  - open to all the clients
  - plus all other cluster nodes for key migrations
- Data port (**normal port + 10000**): used for **Cluster bus**
  - node-to-node communication channel using a binary protocol.
  - used by nodes for failure detection, configuration update, failover authorization...

## Download

출처: https://redis.io/download

```
$ sudo add-apt-repository ppa:redislabs/redis
$ sudo apt-get update
$ sudo apt-get install redis
```

## Docker

- Redis Cluster doesn't support NATted environments where IP addresses or TCP ports are remapped.
- To make Docker compatible with Redis Cluster you need to use **host networking mode** (`--net=host`).

## Data sharding

- There are 16384 hash slots in Redis Cluster.
  - take CRC16 of the key mod 16384
- Every node is responsible for a subset of the hash slots.
  - Node A: 0 ~ 5000 / Node B: 5501 ~ 11000 / Node C: 11001 ~ 16383
- Moving hash slots from a node to another doesn't require to stop operations.
- Multiple keys can be part of same hash slot using **hash tags**.
  - `this{foo}key` and `another{foo}key` are guaranteed to be in the same hash slot.
  - Redis Cluster supports multiple key operations as long as all the keys involved into a single command execution all belong to the same hash slot.

## Master-slave model

- Cluster remain available when a subset of master nodes are failing or not able to communicate with the majority of nodes.
- Every hash slot has from 1 (master node) to N replicas (N-1 slave nodes).
- Node B1 replicates B, and B fails, the cluster will promote node B1 as the new master and continue to operate correctly.

## Consistency guarantees

- There is a trade-off to be made between performance and consistency.
- Redis cLuster can lose writes when
  1. client writes something
  2. B acknowledges the write
  3. B crashes before being able to send the write to its slaves
  4. one of the slaves that did not receive the write gets promoted to master
- Redis Cluster has support for synchronous writes via `WAIT` command.
  - Still Redis Cluster doesn't implement strong consistency.
- **Node timeout**
  - After node timeout, a master node considered to be failing can be replaced by one of its replicas.
  - After node timeout, a master node that cannot sense majority of the other master nodes enters an error state and stops accepting writes.
