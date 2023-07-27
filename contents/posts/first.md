---
title: "🐳 도커로 DB 관리하기"
description: "도커로 어떻게 데이터베이스를 관리할까요?"
date: 2023-07-20
update: 2023-07-20
tags:
  - docker
  - database
series: "도커"
---

안녕하세요 이번에 바톤 프로젝트를 기초 세팅하는 시간을 가졌는데요. 그 중 docker로 DB를 관리하는 부분이 기억에 남아 글로 기록해보려합니다.

## docker로 DB를 관리하게 된 이유
### 1. 일관된 환경
도커 이미지를 사용하면 개발, 로컬, 운영 환경에서 모두 동일한 이미지를 배포할 수 있습니다.
즉, 모든 데이터베이스의 환경을 동일하게 유지할 수 있고 환경 변화가 있다면 동일하게 환경을 변경시킬 수 있습니다.

또한 도커 컴포즈를 이용하여 로컬에서도 동료들간 같은 환경을 구성하여 DB를 사용할 수 있습니다.

### 2. 환경 충돌 방지
우아한테크코스에서 DB를 위한 ec2 인스턴스는 1개만 제공되었습니다.
나중에 mysql 뿐만 아니라 다른 db를 추가하게 되는 경우에 인스턴스 내부에서 환경이 얽힐수도 있겠다는 생각을 했습니다.
만약 도커를 이용하여 컨테이너를 분리하게 된다면 환경이 얽힐 일이 없고, 더 안정적인 운영이 가능하다고 생각했습니다.

### 3. 손쉬운 스케일링
서비스가 커지게되면 트래픽이 늘어나고 부하가 발생할 수 있는데요.
이때 도커를 이용하면 컨테이너를 필요한 시점에 빠르게 확장하거나 축소할 수 있습니다.

</br>

## EC2에 도커 설치
```bash
sudo snap install docker
```
명령어를 이용하여 도커를 설치합니다.

처음에는 `apt update`와 `apt install` 명령어를 통해 도커를 설치하려 했는데, 잘 되지 않았습니다.
찾아보니 요즘은 `snap`을 사용하면 더 간단히 도커를 설치할 수 있기에 `snap` 사용을 권장한다고 하네요.
![](https://velog.velcdn.com/images/hooni_/post/25198967-049d-4723-9a59-c0b4ca7e5857/image.png)

위와 같이 설치를 완료했습니다.
![](https://velog.velcdn.com/images/hooni_/post/6dcd0c64-4f45-491f-b172-b850c0a0febb/image.png)

저는 매번 `sudo`를 이용하여 권한 인증하는 것이 귀찮아서 추가적인 설정을 통해 일반 사용자도 도커에 접근할 수 있도록 설정했습니다.
과정은 [해당 링크](https://snapcraft.io/docker)를 참고했습니다.

```bash
 sudo addgroup --system docker
 sudo adduser $USER docker
 newgrp docker
 sudo snap disable docker
 sudo snap enable docker
```

### before
![](https://velog.velcdn.com/images/hooni_/post/103f8395-0ee3-4eb8-935d-5fc165ae413c/image.png)
### after
![](https://velog.velcdn.com/images/hooni_/post/f1634b4b-120e-4808-943c-34a901dedf99/image.png)


</br>

## 도커에 mysql 컨테이너 설치

### 이미지 다운

도커에서 mysql 이미지를 다운받습니다.
```bash
docker pull mysql
```
도커에서 **이미지**란 하나의 환경파일(?)을 말합니다. nginx, mysql, node 등등 많은 이미지들이 있는데, 사용 목적에 맞게 이미지를 선택해서 컨테이너로 생성하면 해당 이미지에 맞는 환경이 설치된 상태로 컨테이너를 사용할 수 있습니다.
OOP에서 원하는 클래스의 인스턴스를 생성하는 것과 비슷한 느낌이라고 할 수 있겠습니다.

![](https://velog.velcdn.com/images/hooni_/post/9994784b-ed69-470b-8d09-e0323d42bc15/image.png)

mysql 이미지가 정상적으로 깔린 것을 확인할 수 있습니다.
`tag`는 버전을 뜻하는데, latest는 가장 최신버전이라는 뜻입니다. 만약 제가 `docker pull mysql:8.0.33` 명령어로 이미지를 풀 받게 되면 해당 버전의 mysql을 풀 받습니다.

![](https://velog.velcdn.com/images/hooni_/post/1dae895b-e8e4-48f4-88b1-19527ffb205e/image.png)

현재 가장 최근 버전의 mysql이 8.0.33 이어서 image id가 같은 것 같습니다.

### 컨테이너 생성 & 실행
```bash
docker run --name mysql-dev-container -e MYSQL_ROOT_PASSWORD={비밀번호} -d -p 3306:3306 mysql:latest
```

해석하자면, 
- `--name`: `mysql-dev-container`의 이름으로 컨테이너를 실행한다.
- `-e`: 컨테이너의 환경 변수중 `MYSQL_ROOT_PASSWORD`를 입력한 비밀번호로 설정한다.
- `-d`: 데몬 모드로 실행한다. (백그라운드 모드)
- `-p`: 호스트 포트:컨테이너 포트로 설정한다.
- 이미지는 `mysql:latest` 로 설정한다.

![](https://velog.velcdn.com/images/hooni_/post/f3d174f9-732d-49c7-befb-1bc570d1e138/image.png)

정상적으로 컨테이너가 생긴 것을 확인할 수 있습니다.

이제 컨테이너에 접속해보겠습니다.
```bash
docker exec -it {컨테이너 Id} bash
```

위 명령어를 통해 도커 컨테이너 bash에 접속했습니다.

### mysql에 db 생성
이제 컨테이너에 mysql database를 설치해보도록 하겠습니다.
여기부터는 일반 리눅스나 맥에서 database 생성하는 것과 똑같습니다.

먼저 mysql에 접속합니다.
```bash
mysql -u root -p
```
그리고, database를 생성합니다.
```bash
create database ditoo_dev;
```

intellij에서 생성한 db에 정상적으로 접근되는지 확인해보겠습니다!
![](https://velog.velcdn.com/images/hooni_/post/92c5d4df-6651-44da-aa40-dea62909261e/image.png)
정상적으로 접근되었습니다.

간단한 post, tag, post_tag 테이블을 만들어 table이 정상적으로 생성되는지 확인해보도록 하겠습니다.

![](https://velog.velcdn.com/images/hooni_/post/cb75119b-fa86-476b-9b26-bcf9001e992d/image.png)

스프링 어플리케이션에서는 정상적으로 테이블이 생성된 것을 확인했습니다.

![](https://velog.velcdn.com/images/hooni_/post/19117d25-f71e-4d01-b1fb-65e754bf582a/image.png)

db에도 정상적으로 table이 생긴 것을 확인할 수 있었습니다.

</br>

## 마치며

docker로 간단하게 db를 관리하는 것을 학습했습니다.
다음 시간에는 docker-compose를 이용하여 db, spring, react 모두 같이 관리하는 방법에 대해 학습해보려 합니다.