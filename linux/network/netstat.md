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
