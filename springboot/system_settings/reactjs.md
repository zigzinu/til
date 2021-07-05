# 스프링부트 에서 리액트 코드 패키징

- gradle 이용시 리액트 프론트엔드 코드를 서버와 함께 띄우기 위해 아래 소스를 추가한다.

**build.gradle**
```
// 여기서 node 관련
plugins {
	id 'org.springframework.boot' version '2.4.2'
	id 'io.spring.dependency-management' version '1.0.11.RELEASE'
	id 'java'
    id 'com.github.node-gradle.node' version '2.2.0'
}

// 이건 해야하는 건지 모르겠음
configurations {
	compileOnly {
		extendsFrom annotationProcessor
	}
}

// Package react into .jar
node {
    version = '14.16.0'
    npmVersion = '6.14.11'
    download = true
}

def webappDir = "$projectDir/frontend"

task appNpmInstall(type: NpmTask) {
    workingDir = file("${webappDir}")
    args = ["install"]
}

task appNpmBuild(type: NpmTask) {
    workingDir = file("${webappDir}")
    args = ["run", "build"]
}

task copyWebApp(type: Copy) {
    from 'frontend/build'
    into 'build/resources/main/static'
}

appNpmBuild.dependsOn(appNpmInstall)
copyWebApp.dependsOn(appNpmBuild)
compileJava.dependsOn(copyWebApp)
```

- `gradlew bootrun` 명령어 실행시 프론트앤드를 포함시키지 않기 위해서는 아래 처럼 실행한다.

```
$ ./gradlew bootrun -x copyWebApp
```
