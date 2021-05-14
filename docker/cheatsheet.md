### exit 된 컨테이너 삭제

```
docker rm $(docker ps -a -f status=exited -q)
```

### 쓰이지 않는 images 삭제

```
docker image prune
``` 
