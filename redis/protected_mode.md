# Redis 접속 보안과 관련하여

## 로컬호스트 외 접근 제한

`P3X` 도구로 레디스 서버에 접속하려고 보니, 다음과 같은 경고문과 함께 실패했다.

나의 Redis 서버는 VM 인 `192.168.56.101`에 위치해있었다.

```
-DENIED Redis is running in protected mode because protected mode is enabled, no bind address was specified, no authentication password is requested to clients. In this mode connections are only accepted from the loopback interface. If you want to connect from external computers to Redis you may adopt one of the following solutions:
1) Just disable protected mode sending the command ‘CONFIG SET protected-mode no’ from the loopback interface by connecting to Redis from the same host the server is running, however MAKE SURE Redis is not publicly accessible from internet if you do so. Use CONFIG REWRITE to make this change permanent.
2) Alternatively you can just disable the protected mode by editing the Redis configuration file, and setting the protected mode option to ‘no’, and then restarting the server.
3) If you started the server manually just for testing, restart it with the ‘–protected-mode no’ option. 
4) Setup a bind address or an authentication password. 
NOTE: You only need to do one of the above things in order for the server to start accepting connections from the outside.
```

정리해보면 현재 Redis 서버는 `protected-mode=yes` 상태이기 때문에 `loopback` 주소 외에서는 접속이 막혀있다는 것이다.

`redis-cli` 명령어로 설정을 확인할 수 있다.

```
$ redis-cli config get protected*
1) "protected-mode"
2) "yes"
```

## 패스워드 설정

메세지에서 제시한 방법 중 하나를 적용하면 된다고 하니, 나의 레디스 서버에 `requirepass` 옵션을 추가하도록 하겠다.

`redis.conf`
```
port 7000
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 3000
appendonly yes
dir /home/fabric/redis-6.2.3/7000
logfile /home/fabric/redis-6.2.3/7000/redis.log
daemonize yes
requirepass fabric
```

재시작 

```
$ redis-cli -c -p 7000
> shutdown
$ redis-server redis.conf
> config get protected*
(error) NOAUTH Authentication required.
```

- `NOAUTH`: 패스워드 기능이 추가돼서 접근이 제한된 것을 확인.

권한을 얻기 위해선 `cli`에 접근 후 `AUTH` 명령어와 함께 패스워드를 입력하면 된다.

```
$ redis-cli -c -p 7000
127.0.0.1:7000> AUTH fabric
OK
127.0.0.1:7000> cluster nodes
9f876cbd724438705f7a5ee8bfda9c4d7999aaba 127.0.0.1:7000@17000 myself,master - 0 1623743464000 5 connected 0-5460
ef04033f13d83f3fa3c20c32ac959ba8f098c85c 127.0.0.1:7001@17001 master - 0 1623743464892 2 connected 5461-10922
9b3fb844b1329cd37b88fad82374e39fea38efb8 127.0.0.1:7002@17002 master - 0 1623743465902 3 connected 10923-16383
```

