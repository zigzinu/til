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
dir ~/redis-6.2.3/7000
```
- appendonly: 다운되었던 마스터도느 재 시작시 appendonly 파일에 가장 최근 데이터가 있다. 클러스터 운영시 yes 권장.
- `port`와 `dir`를 수정한다.

3. Redis 시작
  
```
$ redis-server 7000/redis.conf
$ redis-server 7001/redis.conf
$ redis-server 7002/redis.conf
```
