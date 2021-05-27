# Cluster Start

출처: http://redisgate.kr/redis/cluster/cluster_start.php

## Redis 설정

1. 최소 3개의 노드를 위한 디렉토리를 생성한다.
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

2. 각 폴더에 **redis.conf** 파일을 생성해준다.

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

3. Redis 시작
  
```
$ redis-server 7000/redis.conf
$ redis-server 7001/redis.conf
$ redis-server 7002/redis.conf
```

## Commands

### 실행중인 Redis 확인

```
$ ps aux | grep redis-server
fabric    4634  0.0  0.0  13144  1120 pts/3    S+   02:22   0:00 grep --color=auto redis-server
redis    19863  0.1  0.1  81388  8644 ?        Ssl  May26   2:09 /usr/bin/redis-server 127.0.0.1:6379
fabric   19984  0.1  0.1  84460  5832 ?        Ssl  May26   1:48 redis-server *:7000 [cluster]

$ netstat -tulpn | grep 7000
tcp        0      0 0.0.0.0:17000           0.0.0.0:*               LISTEN      19984/redis-server
tcp        0      0 0.0.0.0:7000            0.0.0.0:*               LISTEN      19984/redis-server
tcp6       0      0 :::17000                :::*                    LISTEN      19984/redis-server
tcp6       0      0 :::7000                 :::*                    LISTEN      19984/redis-server
```

### Service restart

```
$ sudo service redis-server restart
$ sudo service redis-server status
```

### Redis-cli

```
$ redis-cli -c -p 7000
```
- `-c`: cluster mode
- `-p`: port
- `-h`: hostname (default localhost)
