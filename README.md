
<img src="https://capsule-render.vercel.app/api?type=waving&color=ADEDFF&height=300&section=header&text=🐳Docker-Compose🐳&fontSize=55&fontColor=FFFFFF&animation=fadeIn&width=1200" width="1200" />


<br>

## 📍 Outline
- [1️⃣ Contributors](#1%EF%B8%8F⃣-contributors)
- [2️⃣ Purpose](#2%EF%B8%8F⃣-purpose)
- [3️⃣ Contents](#3%EF%B8%8F⃣-contents)
- [4️⃣ Performance Optimization](#4%EF%B8%8F⃣-performance-optimization)
- [5️⃣ Conclusion](#5%EF%B8%8F⃣-Conclusion)
- 
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

- **Docker Compose + 실행 스크립트**`docker-compose.yml`, `Dockerfile`, 그리고 실행 스크립트(`sh` 파일)를 작성하여 한 번에 실행
- **MySQL 컨테이너 기반의 주기적인 백업 및 자동화**를 `mysqldump`를 이용하여 자동화

<br>


## 3️⃣ Contents

<br>

✅ **폴더 구조**
```
Project
├── app1
│   ├── app.jar
│   └── Dockerfile
├── docker-compose.yml
├── backups
└── scripts
    ├── backup1.sh
    └── start.sh
```

<br>

✅ **docker-compose.yml**

```
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

✅ **start.sh**

```
echo "Docker Compose 실행 중..."
docker-compose up -d --build

echo "모든 컨테이너 실행 완료!"
docker ps
```

<br>

✅ **backup1.sh**

```
BACKUP_DIR=/home/admin1/10.Project/backups
mkdir -p $BACKUP_DIR

# 현재 날짜 및 시간
TIMESTAMP=$(date +'%Y%m%d_%H%M%S')

# MySQL 컨테이너에서 fisa 데이터베이스 백업 실행
docker exec mysqldb mysqldump --default-character-set=utf8mb4 -u user01 -puser01 fisa > $BACKUP_DIR/backup_$TIMESTAMP.sql

echo "백업 완료: $BACKUP_DIR/backup_$TIMESTAMP.sql"

# testdb로 복원하기
docker exec -i mysqldb mysql -u root -proot testdb < $BACKUP_DIR/backup_$TIMESTAMP.sql

echo "testdb로 데이터 복원이 완료되었습니다."admin1@myserver1:~/10.Project/scripts$ cat backup1.sh
BACKUP_DIR=/home/admin1/10.Project/backups
mkdir -p $BACKUP_DIR

# 현재 날짜 및 시간
TIMESTAMP=$(date +'%Y%m%d_%H%M%S')

# MySQL 컨테이너에서 fisa 데이터베이스 백업 실행
docker exec mysqldb mysqldump --default-character-set=utf8mb4 -u user01 -puser01 fisa > $BACKUP_DIR/backup_$TIMESTAMP.sql

echo "백업 완료: $BACKUP_DIR/backup_$TIMESTAMP.sql"

# testdb로 복원하기
docker exec -i mysqldb mysql -u root -proot testdb < $BACKUP_DIR/backup_$TIMESTAMP.sql

echo "testdb로 데이터 복원이 완료되었습니다."
```
<br>

✅ **Dockerfile**

```
FROM openjdk:17
COPY app.jar /app.jar
ENTRYPOINT ["java", "-jar", "/app.jar"]
```

<br>

✅ **최종 실행 방법**

1. 컨테이너 실행
./scripts/start.sh

2. 자동 백업 실행

![image](https://github.com/user-attachments/assets/e1018f7f-d04e-49fa-86ef-88fb8a1decd4)

```
crontab -e
* * * * * /bin/bash /home/admin1/10.Project/scripts/backup1.sh
```
<br>

![image](https://github.com/user-attachments/assets/5cd0bae9-d305-4bba-a938-20ceea9425ea)

- `backup1.sh`와 `crontab`을 이용해 **DB 백업 자동화** 진행
- **Docker Compose를 통해 전체 시스템 구성 및 실행**`docker-compose.yml`을 통해 컨테이너 실행

<br>


## 4️⃣ Performance Optimization

<br>

<br>



## 5️⃣ Conclusion

sql문으로 자동 백업을 통해서 'testdb'데이터베이스로 데이터 백업 가능
<br>
<br>



<img src="https://capsule-render.vercel.app/api?type=waving&color=ADEDFF&height=150&section=footer" width="1000" />
