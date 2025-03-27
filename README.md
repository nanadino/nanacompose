
<img src="https://capsule-render.vercel.app/api?type=waving&color=ADEDFF&height=300&section=header&text=🐳Docker-Compose🐳&fontSize=55&fontColor=FFFFFF&animation=fadeIn&width=1200" width="1200" />


<br>

## 📍 Outline
- [1️⃣ Contributors](#1%EF%B8%8F⃣-contributors)
- [2️⃣ Purpose](#2%EF%B8%8F⃣-purpose)
- [3️⃣ Contents](#3%EF%B8%8F⃣-contents)
- [4️⃣ Conclusion](#4%EF%B8%8F⃣-Conclusion)
  
<br>
<br>

## 1️⃣ Contributors
<br>

| <img src="https://avatars.githubusercontent.com/u/87555330?v=4" width="200px"> | <img src="https://avatars.githubusercontent.com/u/82265395?v=4" width="200px"> | <img src="https://github.com/JaeHee-devSpace.png" width="200px"> | <img src="https://avatars.githubusercontent.com/u/115103394?v=4" width="200px"> |
| :---: | :---: | :---: | :---: |
| [김민성](https://github.com/minsung159357) | [구민지](https://github.com/minjee83) | [박재희](https://github.com/JaeHee-devSpace) | [이현정](https://github.com/nanahj) |


<br>

## 2️⃣ Purpose

<br>

# 📌 프로젝트 개요

이 프로젝트는 **Docker Compose**를 활용하여 **MySQL 컨테이너 기반의 주기적인 백업 및 자동화**를 구현하는 것을 목표로 합니다. `docker-compose.yml`, `Dockerfile`, 그리고 실행 스크립트(`.sh` 파일)를 작성하여 한 번의 명령어로 컨테이너를 실행하고, `mysqldump`를 이용한 자동 백업을 설정하였습니다.

<br>


## 3️⃣ Contents

<br>

## 📂 폴더 구조

```
Project
├── app1
│   ├── app.jar
│   └── Dockerfile
├── docker-compose.yml
├── backups
└── scripts
    ├── backup1.sh
    └── start.sh
```

<br>

## ⚙️ 구성 요소

### 1️⃣ **docker-compose.yml**

```yaml
version: '3.8'

services:
  mysql:
    build: ./mysql
    container_name: mysqldb
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: fisa
      MYSQL_USER: user01
      MYSQL_PASSWORD: user01
    networks:
      - docker-compose
    ports:
      - "3306:3306"
    volumes:
      - mysql_data:/var/lib/mysql

  app1:
    build: ./app1
    container_name: app1_container
    restart: always
    depends_on:
      - mysql
    environment:
      SPRING_DATASOURCE_URL: jdbc:mysql://mysqldb:3306/fisa
      SPRING_DATASOURCE_USERNAME: user01
      SPRING_DATASOURCE_PASSWORD: user01
    ports:
      - "8081:8080"
    healthcheck:
      test: ["CMD", "curl", "-f", "http://192.168.1.149:8080/actuator/health"]
      interval: 30s
      retries: 3
      start_period: 10s
      timeout: 5s
    networks:
      - docker-compose

volumes:
  mysql_data:

networks:
  docker-compose:
    driver: bridge
```

<br>

### 2️⃣ **Dockerfile** (Spring Boot 애플리케이션 컨테이너)

```dockerfile
FROM openjdk:17
COPY app.jar /app.jar
ENTRYPOINT ["java", "-jar", "/app.jar"]
```

<br>

### 3️⃣ **start.sh** (Docker Compose 실행 스크립트)

```sh
echo "Docker Compose 실행 중..."
docker-compose up -d --build

echo "모든 컨테이너 실행 완료!"
docker ps
```

<br>

### 4️⃣ **backup1.sh** (MySQL 자동 백업 및 복원 스크립트)

```sh
BACKUP_DIR=/home/admin1/10.Project/backups
mkdir -p $BACKUP_DIR

# 현재 날짜 및 시간
TIMESTAMP=$(date +'%Y%m%d_%H%M%S')

# MySQL 컨테이너에서 fisa 데이터베이스 백업 실행
docker exec mysqldb mysqldump --default-character-set=utf8mb4 -u user01 -puser01 fisa > $BACKUP_DIR/backup_$TIMESTAMP.sql

echo "백업 완료: $BACKUP_DIR/backup_$TIMESTAMP.sql"

# 백업 파일을 testdb 데이터베이스로 복원
docker exec -i mysqldb mysql -u root -proot testdb < $BACKUP_DIR/backup_$TIMESTAMP.sql

echo "testdb로 데이터 복원이 완료되었습니다."
```

<br>

## 🚀 실행 방법

### 1️⃣ 컨테이너 실행

```sh
./scripts/start.sh
```

### 2️⃣ 자동 백업 설정 (`crontab` 활용)

백업 스크립트를 일정 주기(1분)로 실행하도록 `crontab`을 설정합니다.

```sh
crontab -e
```

![image](https://github.com/user-attachments/assets/31bd2d1f-4047-46a2-883d-839761b48bf6)


추가할 스케줄 예시 (매일 자정마다 실행):

```sh
0 0 * * * /bin/bash /home/admin1/10.Project/scripts/backup1.sh
```
> ⏰ 위 설정은 매일 00:00시에 자동으로 `backup1.sh`를 실행하여 백업 수행
<br>

![image](https://github.com/user-attachments/assets/5cd0bae9-d305-4bba-a938-20ceea9425ea)


<br>


## 4️⃣ Conclusion

이 프로젝트는 **Docker Compose를 활용한 애플리케이션 및 데이터베이스 컨테이너 자동화**를 목표로 하며, 다음과 같은 주요 효과를 기대할 수 있다.

- **손쉬운 배포 및 실행**: `docker-compose.yml`을 통해 간단한 명령어로 애플리케이션과 MySQL 컨테이너를 실행할 수 있다.
- **자동화된 데이터 백업**: `mysqldump`와 `crontab`을 활용하여 MySQL 데이터베이스의 주기적인 백업을 자동화하여 데이터 유실 위험을 최소화한다.
- **빠른 복구 가능**: 백업된 `.sql` 파일을 활용하여 특정 시점의 데이터베이스 상태로 신속하게 복구할 수 있다.
- **확장성 및 유지보수 용이**: 컨테이너 기반의 구조를 채택함으로써, 개발 및 운영 환경에서 동일한 환경을 유지할 수 있으며, 확장 및 유지보수가 용이하다.

이 프로젝트는 DevOps 및 CI/CD 환경에서 **컨테이너 기반 데이터 관리 및 자동화**를 학습 및 적용하고, 활용할 수 있는 구조를 제공한다.

<br>

<br>




<img src="https://capsule-render.vercel.app/api?type=waving&color=ADEDFF&height=150&section=footer" width="1000" />
