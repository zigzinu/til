# 리눅스 저장공간

## 명령어 모음

### **Disk Free**: `df -h`

디스크 용량 부족에 대한 메세지가 나올 때 사용하는 명령어

![image](https://user-images.githubusercontent.com/83999058/123580006-e7cecc80-d813-11eb-9ba2-4402f56ed211.png)

- `Mounted On` 을 살펴보면 어떤 디렉토리에 마운트된 디스크에 공간이 부족한지 확인 가능하다.

<br>

### **Disk Usage**: `sudo du -sh * | sort -rh`

`ls -h` 명령어를 사용하면 디렉토리 사이즈만 보이며 실제 저장된 콘텐츠들의 사이즈가 확인이 불가능하다.

![image](https://user-images.githubusercontent.com/83999058/123580918-bbb44b00-d815-11eb-909a-176f7deadea0.png)

위 명령어를 사용하면 디렉토리 또는 파일 별로 실제로 차지하는 용량을 확인할 수 있다.

![image](https://user-images.githubusercontent.com/83999058/123581043-0cc43f00-d816-11eb-9077-5e172bbbd967.png)

**`du` 명령어**
- `-s` `--summarize`: 현 디렉토리에 있는 각 파일을 요약해라. `-d 0` 즉 depth 0 과 같은 역할
- `-h` `--human-readable`: 용량을 K, M, G과 같이 이해하기 쉬운 단위로 표현
- `*`: 현 디렉토리의 모들 파일에 대상
- `-c`: total 용량 정보도 표시

**`sort` 명령어**
- `-r`: 크기가 큰 순서로 정렬
- `-h`: `du` 명령어에서 `-h` 옵션처럼 사람이 읽기 쉬운 표현에 대한 정렬

출처: https://stackoverflow.com/questions/1019116/using-ls-to-list-directories-and-their-total-sizes


## ncdu

**ncdu** 프로그램을 설치하면 디렉토리를 순회하면서 용량을 탐색할 수 있다. 정확하게 어디서 많은 용량을 차지하는지 찾기 위해 아주 용이한 프로그램이다.

```
$ sudo apt-get update
$ sudo apt-get install ncdu
$ sudo ncdu -x /
```

![image](https://user-images.githubusercontent.com/83999058/123581843-ab04d480-d817-11eb-95d5-1ec0252542f0.png

용량이 큰 디렉토리 또는 파일로 정렬이 된 상태며 방향키와 엔터키로 순회할 수 있다.

출처: https://superuser.com/questions/1052403/100-usage-dev-dm-1
