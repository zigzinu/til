### exit 된 컨테이너 삭제

```
docker rm $(docker ps -a -f status=exited -q)
```

### 쓰이지 않는 images 삭제

```
docker image prune
``` 

### RUN 명령어 중 error 무시

- `; exit 0` 코드를 끝에 추가한다

```
RUN unknown_command; exit 0
```

<br>

## Java 실행할 때 환경변수 전달하기

스프링부트 내에서 민감정보를 가리기 위해 환경변수로 Jasypt 암호를 전달할 때 활용되었다.

- 이 방법을 사용하면 `ENTRYPOINT`에서 `-Djasypt.encroptor.password=password` 같은 변수 java 실행에 전달하지 않아도 된다.
- `docker run` 할 떄 `-e "JASYPT_ENCRYPTOR_PASSWORD=password"` 처럼 옵션으로 전달하지 않아도 된다.

1. Dockerfile

```
FROM openjdk:11-jdk
EXPOSE 8080

ARG JAR_FILE=build/libs/*.jar
ARG JASYPT_ENCRYPTOR_PASSWORD
ENV JASYPT_ENCRYPTOR_PASSWORD=$JASYPT_ENCRYPTOR_PASSWORD

COPY ${JAR_FILE} /app/springboot.jar

ENTRYPOINT ["java", "-jar", "/app/springboot.jar"]
```
- `ARG` 를 통해 build 할 때 암호를 전달받는다.
- `ENV` 에서 컨테이너 내에서 java 실행 시 전달할 환경변수를 저장한다.

2. docker build

```
docker build --build-arg JASYPT_ENCRYPTOR_PASSWORD=password -t myProject .
```

3. docker 

```
docker run -p 8080:8080 myProject
```


출처: 
- https://blusky10.tistory.com/404
- https://stackoverflow.com/questions/19537645/get-environment-variable-value-in-dockerfile
