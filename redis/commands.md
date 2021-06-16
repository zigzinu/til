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