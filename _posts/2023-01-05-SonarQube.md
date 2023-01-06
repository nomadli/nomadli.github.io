---
layout:         post
title:          SonarQube
subtitle:       SonarQube
date:           2023-01-05 11:03:00
author:         nomadli
header-img:     ../img/bg-coffee.jpeg
catalog:        true
tags:
        - other
---

* content
{:toc}

## postgresql
```shell
docker pull postgres
docker pull sonarqube
docker pull alpine
docker pull gitlab/gitlab-runner

docker network create sonarqube-net

mkdir -p /vdb/sonarqube/db/data
docker run -d --name sonarqube-db \
-v /vdb/sonarqube/db:/var/lib/postgresql \
-v /vdb/sonarqube/db/data:/var/lib/postgresql/data \
-e POSTGRES_USER=sonar \
-e POSTGRES_PASSWORD=xx \
-e POSTGRES_DB=sonar \
-e TZ=Asia/Shanghai \
--restart always --network sonarqube-net postgres:latest
sleep 10

mkdir -p /vdb/sonarqube/logs
mkdir -p /vdb/sonarqube/conf
mkdir -p /vdb/sonarqube/data
mkdir -p /vdb/sonarqube/extensions/plugins
vi /etc/sysctl.conf vm.max_map_count=524288 fs.file-max=131072
ulimit -n 131072
ulimit -u 8192
sysctl -p
docker run -d --name sonarqube -p 90:9000 \
-v /vdb/sonarqube/logs:/opt/sonarqube/logs \
-v /vdb/sonarqube/conf:/opt/sonarqube/conf \
-v /vdb/sonarqube/data:/opt/sonarqube/data \
-v /vdb/sonarqube/extensions:/opt/sonarqube/extensions \
-e SONAR_LOG_LEVEL=TRACE \
-e SONAR_JDBC_USERNAME=sonar \
-e SONAR_JDBC_PASSWORD=xxx \
-e SONAR_JDBC_URL=jdbc:postgresql://sonarqube-db:5432/sonar \
--restart always --network sonarqube-net sonarqube:latest
# default user pass admin admin
cp https://github.com/xuhuisheng/sonar-l10n-zh /vdb/sonarqube/extensions/plugins/

mkdir -p /vdb/sonarqube/runner/config
docker run -d --name sonarqube-runner \
-v /vdb/sonarqube/runner/config:/etc/gitlab-runner \
-v /var/run/docker.sock:/var/run/docker.sock \
--restart always gitlab/gitlab-runner:latest


docker exec -it sonarqube-runner bash
gitlab-runner register
    docker
    alpine:latest
```