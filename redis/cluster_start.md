# Cluster Start

출처: http://redisgate.kr/redis/cluster/cluster_start.php

## 레디스 서버 시작하기

### 1. 최소 3개의 노드를 위한 디렉토리를 생성한다.
  - `7000`, `7001`, `7002` 포트 사용.
  - 각각의 conf 파일을 저장하기 위한 디렉토리 생성.
  - Redis 버젼으로 상위 디렉토리 생성.
  
```
$ redis-server -v
Redis server v=6.2.3 sha=00000000:0 malloc=jemalloc-5.1.0 bits=64 build=a0976c4a01bcd8ee
```
  
```
mkdir -p ~/redis-6.2.3
cd ~/redis-6.2.3
mkdir 7000 7001 7002
```

### 2. 각 폴더에 **redis.conf** 파일을 생성해준다.

```
port 7000
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 3000
appendonly yes
dir /home/fabric/redis-6.2.3/7000
logfile /home/fabric/redis-6.2.3/7000/redis.log
daemonize yes
```
- appendonly: 다운되었던 마스터도느 재 시작시 appendonly 파일에 가장 최근 데이터가 있다. 클러스터 운영시 yes 권장.
- `port`와 `dir`를 수정한다.
- daemonize: `redis-server` 명령어로 입력 시 데몬으로 실행

### 3. Redis 시작
  
```
$ redis-server 7000/redis.conf

6653:C 27 May 2021 04:44:38.997 # Redis version=6.2.3, bits=64, commit=00000000, modified=0, pid=6653, just started
6653:C 27 May 2021 04:44:38.997 # Configuration loaded
6653:M 27 May 2021 04:44:38.999 * Increased maximum number of open files to 10032 (it was originally set to 1024).
6653:M 27 May 2021 04:44:38.999 * monotonic clock: POSIX clock_gettime
6653:M 27 May 2021 04:44:39.001 * Node configuration loaded, I'm 9f876cbd724438705f7a5ee8bfda9c4d7999aaba
6653:M 27 May 2021 04:44:39.001 * Running mode=cluster, port=7000.
6653:M 27 May 2021 04:44:39.001 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
6653:M 27 May 2021 04:44:39.001 # Server initialized
6653:M 27 May 2021 04:44:39.001 # WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
6653:M 27 May 2021 04:44:39.002 * Ready to accept connections

$ redis-server 7001/redis.conf

6708:C 27 May 2021 04:47:24.165 # Redis version=6.2.3, bits=64, commit=00000000, modified=0, pid=6708, just started
6708:C 27 May 2021 04:47:24.166 # Configuration loaded
6708:M 27 May 2021 04:47:24.167 * Increased maximum number of open files to 10032 (it was originally set to 1024).
6708:M 27 May 2021 04:47:24.168 * monotonic clock: POSIX clock_gettime
6708:M 27 May 2021 04:47:24.171 * No cluster configuration found, I'm ef04033f13d83f3fa3c20c32ac959ba8f098c85c
6708:M 27 May 2021 04:47:24.175 * Running mode=cluster, port=7001.
6708:M 27 May 2021 04:47:24.175 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
6708:M 27 May 2021 04:47:24.175 # Server initialized
6708:M 27 May 2021 04:47:24.175 # WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
6708:M 27 May 2021 04:47:24.175 * Ready to accept connections

$ redis-server 7002/redis.conf
```
- `No cluster configuration found, I'm ef04033f13d83f3fa3c20c32ac959ba8f098c85c`: node.conf 파일이 없을 때
  - 한번 실행하면 node.conf 파일이 생성된다.
- `Node configuration loaded, I'm 9f876cbd724438705f7a5ee8bfda9c4d7999aaba`: node.conf 파일이 있을 때
- Running mode=cluster

```
$ redis-server -c -p 7000
> set key value
(error) CLUSTERDOWN Hash slot not served
```

레디스는 시작했지만, 클러스터가 시작된 것은 아니다.

## 레디스 클러스터 시작하기

### 1. 구동

```
$ redis-cli --cluster create 127.0.0.1:7000 127.0.0.1:7001 127.0.0.1:7002
>>> Performing hash slots allocation on 3 nodes...
Master[0] -> Slots 0 - 5460
Master[1] -> Slots 5461 - 10922
Master[2] -> Slots 10923 - 16383
M: 9f876cbd724438705f7a5ee8bfda9c4d7999aaba 127.0.0.1:7000
   slots:[0-5460] (5461 slots) master
M: ef04033f13d83f3fa3c20c32ac959ba8f098c85c 127.0.0.1:7001
   slots:[5461-10922] (5462 slots) master
M: 9b3fb844b1329cd37b88fad82374e39fea38efb8 127.0.0.1:7002
   slots:[10923-16383] (5461 slots) master
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join
..
>>> Performing Cluster Check (using node 127.0.0.1:7000)
M: 9f876cbd724438705f7a5ee8bfda9c4d7999aaba 127.0.0.1:7000
   slots:[0-5460] (5461 slots) master
M: ef04033f13d83f3fa3c20c32ac959ba8f098c85c 127.0.0.1:7001
   slots:[5461-10922] (5462 slots) master
M: 9b3fb844b1329cd37b88fad82374e39fea38efb8 127.0.0.1:7002
   slots:[10923-16383] (5461 slots) master
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```
- redis-server 버젼 `4.x` 전까지는 `src/redis-trib.rb create 127.0.0.1:7000 127.0.0.1:7001 127.0.0.1:7002` 명령어 사용

### 2. 클라이언트 사용

```
$ redis-cli -c -p 7000
127.0.0.1:7000> cluster info
cluster_state:ok
cluster_slots_assigned:16384
cluster_slots_ok:16384
cluster_slots_pfail:0
cluster_slots_fail:0
cluster_known_nodes:3
cluster_size:3
cluster_current_epoch:3
cluster_my_epoch:1
cluster_stats_messages_ping_sent:394
cluster_stats_messages_pong_sent:407
cluster_stats_messages_sent:801
cluster_stats_messages_ping_received:405
cluster_stats_messages_pong_received:394
cluster_stats_messages_meet_received:2
cluster_stats_messages_received:801

127.0.0.1:7000> cluster nodes
ef04033f13d83f3fa3c20c32ac959ba8f098c85c 127.0.0.1:7001@17001 master - 0 1622092214158 2 connected 5461-10922
9b3fb844b1329cd37b88fad82374e39fea38efb8 127.0.0.1:7002@17002 master - 0 1622092214662 3 connected 10923-16383
9f876cbd724438705f7a5ee8bfda9c4d7999aaba 127.0.0.1:7000@17000 myself,master - 0 1622092213000 1 connected 0-5460
```
- 위에서 실행한 각 노드가 master 인 것을 확인할 수 있다.
- `cluster_state:ok`: 정상 작동중
- `cluster_slots_assigned:16384`: 사용할 수 있는 DB slot 개수

```
127.0.0.1:7000> set key Hello
-> Redirected to slot [12539] located at 127.0.0.1:7002
OK
127.0.0.1:7002> set key2 World
-> Redirected to slot [4998] located at 127.0.0.1:7000
OK
127.0.0.1:7000> get key
-> Redirected to slot [12539] located at 127.0.0.1:7002
"Hello"
127.0.0.1:7002> get key2
-> Redirected to slot [4998] located at 127.0.0.1:7000
"World"
```

- 클러스터 모드로 접속하면 IP 다음에 DB 번호가 아닌 포트로 표시된다.
- 키 입력 시 레디스 클러스터와 동일한 알고리즘으로 slot을 구한 다음 해당 slot을 보유하고있는 레디스 서버에 접속해서 저장/조회를 한다.

### 3. 노드 다운

```
$ redis-cli -c -p 7002
127.0.0.1:7002> debug segfault
Could not connect to Redis at 127.0.0.1:7002: Connection refused
(1.53s)
not connected>
```
- `debug segfault`: 디버그 용도로 서버를 crash 하는 명령어

7000/7001 로그

```
6708:M 27 May 2021 05:17:41.276 * Marking node 9b3fb844b1329cd37b88fad82374e39fea38efb8 as failing (quorum reached).
6708:M 27 May 2021 05:17:41.276 # Cluster state changed: fail
```

```
$ redis-cli -c -p 7000
127.0.0.1:7000> set key Hello
(error) CLUSTERDOWN The cluster is down

127.0.0.1:7000> cluster info
cluster_state:fail
cluster_slots_assigned:16384
cluster_slots_ok:10923
cluster_slots_pfail:0
cluster_slots_fail:5461
cluster_known_nodes:3
cluster_size:3
cluster_current_epoch:3
cluster_my_epoch:1
cluster_stats_messages_ping_sent:1235
cluster_stats_messages_pong_sent:1275
cluster_stats_messages_fail_sent:1
cluster_stats_messages_sent:2511
cluster_stats_messages_ping_received:1273
cluster_stats_messages_pong_received:1234
cluster_stats_messages_meet_received:2
cluster_stats_messages_fail_received:1
cluster_stats_messages_received:2510

127.0.0.1:7000> cluster nodes
ef04033f13d83f3fa3c20c32ac959ba8f098c85c 127.0.0.1:7001@17001 master - 0 1622092831531 2 connected 5461-10922
9b3fb844b1329cd37b88fad82374e39fea38efb8 127.0.0.1:7002@17002 master,fail - 1622092657507 1622092656000 3 disconnected 10923-16383
9f876cbd724438705f7a5ee8bfda9c4d7999aaba 127.0.0.1:7000@17000 myself,master - 0 1622092831000 1 connected 0-5460
```
- `7002`: 상태가 fail/disconnected
- `cluster_slots_ok:10923`: 정상적으로 할당된 슬롯 수

### 4. 노드 복구

```
$ redis-server 7002/redis.conf
```

7000 서버 로그

```
6653:M 27 May 2021 05:22:49.869 * Clear FAIL state for node 9b3fb844b1329cd37b88fad82374e39fea38efb8: is reachable again and nobody is serving its slots after some time.
6653:M 27 May 2021 05:22:49.870 # Cluster state changed: ok
```

## 노드 추가하기

### 1. 7003 서버 준비

```
$ mkdir 7003
$ vi 7003/redis.conf
port 7003
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 3000
appendonly yes
dir /home/fabric/redis-6.2.3/7003
logfile /home/fabric/redis-6.2.3/7003/redis.log
daemonize yes
$ redis-server 7003/redis.conf
```

### 2. 노드 추가 (add-node)

```
$ redis-cli --cluster add-node 127.0.0.1:7003 127.0.0.1:7000
>>> Adding node 127.0.0.1:7003 to cluster 127.0.0.1:7000
>>> Performing Cluster Check (using node 127.0.0.1:7000)
M: 9f876cbd724438705f7a5ee8bfda9c4d7999aaba 127.0.0.1:7000
   slots:[0-5460] (5461 slots) master
M: ef04033f13d83f3fa3c20c32ac959ba8f098c85c 127.0.0.1:7001
   slots:[5461-10922] (5462 slots) master
M: 9b3fb844b1329cd37b88fad82374e39fea38efb8 127.0.0.1:7002
   slots:[10923-16383] (5461 slots) master
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
>>> Send CLUSTER MEET to node 127.0.0.1:7003 to make it join the cluster.
[OK] New node added correctly.
```

- `redis-cli --cluster add-node 127.0.0.1:7003 127.0.0.1:7000`
  - 첫 번째 IP:PORT = 추가랑 노드
  - 두 번째 IP:PORT = 작업할 노드

```
$ redis-cli -c -p 7000
127.0.0.1:7000> cluster nodes
ef04033f13d83f3fa3c20c32ac959ba8f098c85c 127.0.0.1:7001@17001 master - 0 1622093463538 2 connected 5461-10922
9b3fb844b1329cd37b88fad82374e39fea38efb8 127.0.0.1:7002@17002 master - 0 1622093464548 3 connected 10923-16383
548e9ebb259255e2d7e76ec9fa92c25803ebb172 127.0.0.1:7003@17003 master - 0 1622093464951 0 connected
9f876cbd724438705f7a5ee8bfda9c4d7999aaba 127.0.0.1:7000@17000 myself,master - 0 1622093464000 1 connected 0-5460
```

- 아직 7003 노드에는 슬롯이 할당되지 않았음.

### 3. Slot 할당 (resharding)

```
$ redis-cli --cluster reshard 127.0.0.1:7000
>>> Performing Cluster Check (using node 127.0.0.1:7000)
M: 9f876cbd724438705f7a5ee8bfda9c4d7999aaba 127.0.0.1:7000
   slots:[0-5460] (5461 slots) master
M: ef04033f13d83f3fa3c20c32ac959ba8f098c85c 127.0.0.1:7001
   slots:[5461-10922] (5462 slots) master
M: 9b3fb844b1329cd37b88fad82374e39fea38efb8 127.0.0.1:7002
   slots:[10923-16383] (5461 slots) master
M: 548e9ebb259255e2d7e76ec9fa92c25803ebb172 127.0.0.1:7003
   slots: (0 slots) master
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
How many slots do you want to move (from 1 to 16384)? 10
What is the receiving node ID? 548e9ebb259255e2d7e76ec9fa92c25803ebb172
Please enter all the source node IDs.
  Type 'all' to use all the nodes as source nodes for the hash slots.
  Type 'done' once you entered all the source nodes IDs.
Source node #1: 9f876cbd724438705f7a5ee8bfda9c4d7999aaba
Source node #2: done

Ready to move 10 slots.
  Source nodes:
    M: 9f876cbd724438705f7a5ee8bfda9c4d7999aaba 127.0.0.1:7000
       slots:[0-5460] (5461 slots) master
  Destination node:
    M: 548e9ebb259255e2d7e76ec9fa92c25803ebb172 127.0.0.1:7003
       slots: (0 slots) master
  Resharding plan:
    Moving slot 0 from 9f876cbd724438705f7a5ee8bfda9c4d7999aaba
    Moving slot 1 from 9f876cbd724438705f7a5ee8bfda9c4d7999aaba
    Moving slot 2 from 9f876cbd724438705f7a5ee8bfda9c4d7999aaba
    Moving slot 3 from 9f876cbd724438705f7a5ee8bfda9c4d7999aaba
    Moving slot 4 from 9f876cbd724438705f7a5ee8bfda9c4d7999aaba
    Moving slot 5 from 9f876cbd724438705f7a5ee8bfda9c4d7999aaba
    Moving slot 6 from 9f876cbd724438705f7a5ee8bfda9c4d7999aaba
    Moving slot 7 from 9f876cbd724438705f7a5ee8bfda9c4d7999aaba
    Moving slot 8 from 9f876cbd724438705f7a5ee8bfda9c4d7999aaba
    Moving slot 9 from 9f876cbd724438705f7a5ee8bfda9c4d7999aaba
Do you want to proceed with the proposed reshard plan (yes/no)? yes
Moving slot 0 from 127.0.0.1:7000 to 127.0.0.1:7003:
Moving slot 1 from 127.0.0.1:7000 to 127.0.0.1:7003:
Moving slot 2 from 127.0.0.1:7000 to 127.0.0.1:7003:
Moving slot 3 from 127.0.0.1:7000 to 127.0.0.1:7003:
Moving slot 4 from 127.0.0.1:7000 to 127.0.0.1:7003:
Moving slot 5 from 127.0.0.1:7000 to 127.0.0.1:7003:
Moving slot 6 from 127.0.0.1:7000 to 127.0.0.1:7003:
Moving slot 7 from 127.0.0.1:7000 to 127.0.0.1:7003:
Moving slot 8 from 127.0.0.1:7000 to 127.0.0.1:7003:
Moving slot 9 from 127.0.0.1:7000 to 127.0.0.1:7003:
```
- `redis-cli --cluster reshard 127.0.0.1:7000`: 작업할 노드를 입력한다
- `How many slots do you want to move (from 1 to 16384)? 10`: 몇 개의 slot을 reshard 할지 입력한다.
- `What is the receiving node ID? 548e9ebb259255e2d7e76ec9fa92c25803ebb172`: 받을 노드 ID를 입력한다.
- `Source node #1: 9f876cbd724438705f7a5ee8bfda9c4d7999aaba`: 할당 해줄 소스 노드 ID를 입력한다.
- `Source node #2: done`: 할당 해줄 소스 노드 입력이 끝나면 `done`을 입력한다.
- `Do you want to proceed with the proposed reshard plan (yes/no)? yes`: 확인

```
$ redis-cli -c -p 7000
127.0.0.1:7000> cluster info
cluster_state:ok
cluster_slots_assigned:16384
cluster_slots_ok:16384
cluster_slots_pfail:0
cluster_slots_fail:0
cluster_known_nodes:4
cluster_size:4
cluster_current_epoch:4
cluster_my_epoch:1
cluster_stats_messages_ping_sent:3154
cluster_stats_messages_pong_sent:3214
cluster_stats_messages_fail_sent:1
cluster_stats_messages_sent:6369
cluster_stats_messages_ping_received:3211
cluster_stats_messages_pong_received:3163
cluster_stats_messages_meet_received:3
cluster_stats_messages_fail_received:1
cluster_stats_messages_received:6378
```
- `cluster_size:4`: resharding 후 4로 변경되었다.

```
127.0.0.1:7000> cluster nodes
ef04033f13d83f3fa3c20c32ac959ba8f098c85c 127.0.0.1:7001@17001 master - 0 1622094009940 2 connected 5461-10922
9b3fb844b1329cd37b88fad82374e39fea38efb8 127.0.0.1:7002@17002 master - 0 1622094009534 3 connected 10923-16383
548e9ebb259255e2d7e76ec9fa92c25803ebb172 127.0.0.1:7003@17003 master - 0 1622094009000 4 connected 0-9
9f876cbd724438705f7a5ee8bfda9c4d7999aaba 127.0.0.1:7000@17000 myself,master - 0 1622094009000 1 connected 10-5460
```

## 노드 제거하기

### 1. Resharding

```
$ redis-cli --cluster reshard 127.0.0.1:7000
>>> Performing Cluster Check (using node 127.0.0.1:7000)
M: 9f876cbd724438705f7a5ee8bfda9c4d7999aaba 127.0.0.1:7000
   slots:[0-9],[20-5460] (5451 slots) master
M: ef04033f13d83f3fa3c20c32ac959ba8f098c85c 127.0.0.1:7001
   slots:[5461-10922] (5462 slots) master
M: 9b3fb844b1329cd37b88fad82374e39fea38efb8 127.0.0.1:7002
   slots:[10923-16383] (5461 slots) master
M: 548e9ebb259255e2d7e76ec9fa92c25803ebb172 127.0.0.1:7003
   slots:[10-19] (10 slots) master
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
How many slots do you want to move (from 1 to 16384)? 10
What is the receiving node ID? 9f876cbd724438705f7a5ee8bfda9c4d7999aaba
Please enter all the source node IDs.
  Type 'all' to use all the nodes as source nodes for the hash slots.
  Type 'done' once you entered all the source nodes IDs.
Source node #1: 548e9ebb259255e2d7e76ec9fa92c25803ebb172
Source node #2: done

Ready to move 10 slots.
  Source nodes:
    M: 548e9ebb259255e2d7e76ec9fa92c25803ebb172 127.0.0.1:7003
       slots:[10-19] (10 slots) master
  Destination node:
    M: 9f876cbd724438705f7a5ee8bfda9c4d7999aaba 127.0.0.1:7000
       slots:[0-9],[20-5460] (5451 slots) master
  Resharding plan:
    Moving slot 10 from 548e9ebb259255e2d7e76ec9fa92c25803ebb172
    Moving slot 11 from 548e9ebb259255e2d7e76ec9fa92c25803ebb172
    Moving slot 12 from 548e9ebb259255e2d7e76ec9fa92c25803ebb172
    Moving slot 13 from 548e9ebb259255e2d7e76ec9fa92c25803ebb172
    Moving slot 14 from 548e9ebb259255e2d7e76ec9fa92c25803ebb172
    Moving slot 15 from 548e9ebb259255e2d7e76ec9fa92c25803ebb172
    Moving slot 16 from 548e9ebb259255e2d7e76ec9fa92c25803ebb172
    Moving slot 17 from 548e9ebb259255e2d7e76ec9fa92c25803ebb172
    Moving slot 18 from 548e9ebb259255e2d7e76ec9fa92c25803ebb172
    Moving slot 19 from 548e9ebb259255e2d7e76ec9fa92c25803ebb172
Do you want to proceed with the proposed reshard plan (yes/no)? yes
Moving slot 10 from 127.0.0.1:7003 to 127.0.0.1:7000:
Moving slot 11 from 127.0.0.1:7003 to 127.0.0.1:7000:
Moving slot 12 from 127.0.0.1:7003 to 127.0.0.1:7000:
Moving slot 13 from 127.0.0.1:7003 to 127.0.0.1:7000:
Moving slot 14 from 127.0.0.1:7003 to 127.0.0.1:7000:
Moving slot 15 from 127.0.0.1:7003 to 127.0.0.1:7000:
Moving slot 16 from 127.0.0.1:7003 to 127.0.0.1:7000:
Moving slot 17 from 127.0.0.1:7003 to 127.0.0.1:7000:
Moving slot 18 from 127.0.0.1:7003 to 127.0.0.1:7000:
Moving slot 19 from 127.0.0.1:7003 to 127.0.0.1:7000:
```

- 노드 추가할 때 했던 resharding 을 역방향으로 실행

```
fabric@cicd:~/redis-6.2.3/7001$ redis-cli -c -p 7000
127.0.0.1:7000> cluster info
cluster_state:ok
cluster_slots_assigned:16384
cluster_slots_ok:16384
cluster_slots_pfail:0
cluster_slots_fail:0
cluster_known_nodes:4
cluster_size:3
```

- `cluster_known_nodes` 개수는 4인데 `cluster_size`는 3으로 변경되었다.

### 2. 노드 제거 (del-node)

```
$ redis-cli --cluster del-node 127.0.0.1:7000 548e9ebb259255e2d7e76ec9fa92c25803ebb172
>>> Removing node 548e9ebb259255e2d7e76ec9fa92c25803ebb172 from cluster 127.0.0.1:7000
>>> Sending CLUSTER FORGET messages to the cluster...
>>> Sending CLUSTER RESET SOFT to the deleted node.
```

- 접속할 노드의 IP:PORT 와 삭제할 노드의 ID를 입력한다.
- 7003 노드가 `shutdown`되지는 않는다.

