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

## Configuration parameters
- **cluster-enabled <yes/no>**
  - yes: Redis Cluster support in a specific Redis instance.
  - no: instance starts as a stand alone instance.
- **cluster-config-file <filename>**: file that Redis Cluster persists to re-read at startup.
  - contains nodes in the cluster, their state, persistent variables...
- **cluster-node-timeout <milliseconds>**
  - Every node that can't reach the majority of master nodes for specific amount of time, will stop accepting queries.
  - If a master node is not reachable for more than specific amount of time, it will be failed over by its slaves.
- **cluster-slave-validity-factor <factor>**
  - zero: slave remains always valid to failover a master
  - positive: slave losing connection to its master more than factor * node-timeout becomes invalid to failover a master.
- **cluster-migration-barrier <count>**: number of minimum slave counts for a master. Slave gets migrated from a master with sufficient slaves to a master that has lost slave below the barrier count. 
  - defualt: 1
- **cluster-require-full-coverage <yes/no>**
  - yes: slave 가 없는 master 중 하나라도 다운 되면 클러스터가 다운된다.
  - no: slave가 없는 master가 다운되어도 과만의 master가 살아있으면, 클러스터가 정상적으로 살아있다.
  - yes: 살아있는 노드의 데이터를 받아도 전체 데이터의 무결성이 깨질 때는 yes 선택.
- **cluster-allow-reads-when-down <yes/no>**
  - yes: allow read from a node during the `fail` state.
  
## Multikey operations

출처: http://redisgate.kr/redis/cluster/cluster_introduction.php
  
- 클러스터에서는 멀티 키 명령을 수행할 수 없다. 
  - `MSET key1 value1 key2 value2`
  - `SUNION key1 key2`
  - `SORT`
- **Hash tag**를 사용하면 사용할 수 있다.
  - `{user001}.following`와 `{user001}.followers`는 같은 슬롯에 저장된다.
- [**Enterprise 게이트 서버**](http://redisgate.kr/redisgate/ent/gate_intro.php)를 사용하면 멀티 키 명령을 사용할 수 있다.
