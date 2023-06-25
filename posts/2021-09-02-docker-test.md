---
aliases:
- /kubernetes/2021/09/02/docker-test
categories:
- kubernetes
date: '2021-09-02'
layout: post
title: Docker test
toc: false

---

[jaewonhimnae_git](https://github.com/jaewonhimnae/docker-codes/)
---

# 사전 테스트

## [docker-compose](https://engineer-mole.tistory.com/221)
여러 개의 컨테이너로부터 이루어진 서비스를 구축, 실행하는 순서를 자동으로 하여, 관리를 간단히하는 기능  

### docker-compose 설치  

```
# echo ${COMPOSE_VERSION} 
# curl -L https://github.com/docker/compose/releases/download/${COMPOSE_VERSION}/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose 
# chown root:docker /usr/local/bin/docker-compose 
# chmod ug+x /usr/local/bin/docker-compose 
# update-alternatives --install /usr/bin/docker-compose docker-compose /usr/local/bin/docker-compose 10
# docker-compose --version
```

### docker-compose 파일 위치
```
(Working dir) 
+- docker-compose.yml 
+- app-server/ 
  +- Dockerfile 
  +- src/ 
    +- app.js
```
### docker-compose.yml
```
version: "3"
services:
  redis-server:
    image: "redis"
  node-app:
    build: .
    ports:
      - "5000:8080"
```

## 나머지 구동 파일들 추가

### Dockerfile
```
FROM node:10
WORKDIR /usr/src/app
COPY package.json ./
RUN npm install
COPY ./ ./
CMD ["node","server.js"]
```

### package.json
npm init으로 생성
```
{
  "name": "docker-compose-app",
  "version": "1.0.0",
  "description": "",
  "main": "server.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "node server.js"
  },
  "dependencies": {
    "express": "4.17.1",
    "redis": "3.0.2"
  },
  "author": "",
  "license": "ISC"
}

```

### server.js
```
const express = require('express');
const redis = require('redis');

const client = redis.createClient({
    host: "redis-server",  // in docker-compose.yml
    port: 6379
})

//APP 
const app = express();

client.set("number", 0);

app.get('/', (req, res) => {
    client.get("number", (err, number) => {
        res.send("number conuting: " + number)
        client.set("number", parseInt(number)+1)
    })
});

app.listen(8080);
console.log(`Server is Running`);
```

## docker-compose 구동
```
# docker-compose up
```

---
<br><br>

# 1. Single Container
## 1.1. Development

## 1.2. Test

## 1.3. Deployment




---
<br><br>

# 2. Multi Container
## 2.1. Development

## 2.2. Test

## 2.3. Deployment




---
<br><br>



