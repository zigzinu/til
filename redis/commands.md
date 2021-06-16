# Commands

## 실행중인 Redis 확인

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

## Service restart/stop

`sudo service redis-server start` 로 실행한 서버는 default 값으로 `6379` 포트에서 실행된다. 이 서버는 `redis-cli shutdown` 명령어를 입력해도 다시 살아난다. 

```
fabric@cicd:~/redis-6.2.3/7000$ ps aux | grep redis-server
redis    31173  0.8  0.1  81388  8520 ?        Ssl  08:37   0:00 /usr/bin/redis-server 127.0.0.1:6379
fabric   31194  0.0  0.0  13144  1060 pts/5    S+   08:38   0:00 grep --color=auto redis-server
fabric@cicd:~/redis-6.2.3/7000$ redis-cli
127.0.0.1:6379> shutdown
not connected> exit
fabric@cicd:~/redis-6.2.3/7000$ ps aux | grep redis-server
redis    31214  1.5  0.1  81388  8848 ?        Ssl  08:38   0:00 /usr/bin/redis-server 127.0.0.1:6379
fabric   31237  0.0  0.0  13144  1080 pts/5    S+   08:38   0:00 grep --color=auto redis-server
```
- 새로운 `pid` 로 다시 프로세스가 떴음.

종료해주기 위해서는 `service` 명령어를 입력해야 한다.
```
fabric@cicd:~/redis-6.2.3/7000$ sudo service redis-server stop
fabric@cicd:~/redis-6.2.3/7000$ ps aux | grep redis-server
fabric   31294  0.0  0.0  13144  1120 pts/5    S+   08:38   0:00 grep --color=auto redis-server
```

```
$ sudo service redis-server restart
$ sudo service redis-server status
```

## Redis-cli

cli 접근

```
$ redis-cli -c -p 7000
```
- `-c`: cluster 노드에 접근할 때는 이 옵션을 사용해야 한다.
- `-p`: port
- `-h`: hostname (default 127.0.0.1)

레디스 서버 종료

```
> shutdown
```

```
> redis-cli -c -p 7000 shutdown
```

## Redis cluster 조회

```
$ redis-cli --cluster call 127.0.0.1:7002 keys '*'
```
- `--cluster call`: 클러스터에 포함된 노드 중 하나의 IP/Port를 입력하면, 클러스터의 모든 노드에 대한 명령(`keys '*'`)을 실시한다. 

```
$ redis-cli --cluster call 127.0.0.1:7002 keys '*'
[ERR] Unable to load info for node 127.0.0.1:7000
>>> Calling keys *
127.0.0.1:7002: key
spring:session:sessions:c337fbf2-ebe5-4e79-8ee5-cffb8da64976
127.0.0.1:7001: spring:session:expirations:1623809520000
```
- 암호가 설정된 노드에 대해선 `Authentication`이 실패한다.

### 암호 입력

```
$ redis-cli --cluster call 127.0.0.1:7002 keys '*' -a fabric
```
- `-a`: 암호를 입력하는 옵션
- 운영 환경에서 command 에 입력하는 것을 권장하지 않는다.

```
$ redis-cli --cluster call 127.0.0.1:7002 keys '*' -a fabric
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
Node 127.0.0.1:7002 replied with error:
ERR AUTH <password> called without any password configured for the default user. Are you sure your configuration is correct?
```
- 암호가 설정되어있지 않은 노드가 포함되어있으면, 명령어가 실패한다.

### Watch 모니터링

```
$ watch redis-cli --cluster call 127.0.0.1:7002 keys \'*\' -a fabric
```
- `\`: escape 문자로 따옴표/쌍따옴표를 입력해야 한다.

```
Every 2.0s: redis-cli --cluster call 127.0.0.1:7002 keys '*' -a fabric                                                                                                          cicd: Wed Jun 16 02:15:44 2021

Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
^[29;1m>>> Calling^[0m keys *
127.0.0.1:7002: key
spring:session:sessions:c337fbf2-ebe5-4e79-8ee5-cffb8da64976
127.0.0.1:7001: spring:session:sessions:2a9c0889-9582-402c-b338-0501f16de3c1
127.0.0.1:7000: spring:session:sessions:expires:2a9c0889-9582-402c-b338-0501f16de3c1
spring:session:expirations:1623811560000
key2
```
