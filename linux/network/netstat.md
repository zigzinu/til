### 사용중인 포트의 pid를 확인하기 위해 `netstat -tulpn` 를 입력했는데 pid가 없다.

```
$ netstat -tulpn
tcp        0      0 127.0.0.1:3306          0.0.0.0:*               LISTEN      -            
```

- sudo 로 실행하면 나온다.

```
$ sudo netstat -tulpn
tcp        0      0 127.0.0.1:3306          0.0.0.0:*               LISTEN      1188/mysqld     
```

- `ps -aux` 명령어로 대신하기

```
$ ps -aux | grep mysqld
mysql     8877  0.1  1.4 678328 75972 ?        Ssl  06:07   0:00 /usr/sbin/mysqld
```

- 프로세스 kill

```
$ kill -TERM 8877
```
