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

## install
```shell
docker pull postgres
docker pull sonarqube #9.0以上只支持java11 sonarqube:8.9-community 支持java8
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

##此时启动sonar容器先把conf extensions 拷贝出来
mkdir -p /vdb/sonarqube/logs
mv conf /vdb/sonarqube/
mkdir -p /vdb/sonarqube/data
mv extensions /vdb/sonarqube/
#汉化
cp https://github.com/xuhuisheng/sonar-l10n-zh /vdb/sonarqube/extensions/plugins/
#c/c++插件
cp https://github.com/SonarOpenCommunity/sonar-cxx /vdb/sonarqube/extensions/plugins/
#object-c/swift
cp https://github.com/Idean/sonar-swift /vdb/sonarqube/extensions/plugins/
#多分支插件
wget https://github.com/mc1arke/sonarqube-community-branch-plugin
chmod 775 branch-plugin.jar
cp branch-plugin.jar /vdb/sonarqube/extensions/plugins/
vi /vdb/sonarqube/conf/sonar.properties
sonar.web.javaAdditionalOpts=-javaagent:./extensions/plugins/branch-plugin.jar=web
sonar.ce.javaAdditionalOpts=-javaagent:./extensions/plugins/branch-plugin.jar=ce
#导出报告、配置等插件
https://github.com/cnescatlab/sonar-cnes-report

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

## sonar-scanner 地址
- [地址](https://docs.sonarqube.org/latest/analyzing-source-code/scanners/sonarscanner/)
- 4.6.2 最后一个支持java8
- sonar-scanner -Dsonar.projectKey=gitlabgroup1_gitlabgroup2_..._projectname_rangestr -Dsonar.sources=. -Dsonar.host.url=http://1.1.1.1:90 -Dsonar.login=token
- https://sonarcloud.io/static/cpp/build-wrapper-macosx-x86.zip
- https://sonarcloud.io/static/cpp/build-wrapper-win-x86.zip
- https://sonarcloud.io/static/cpp/build-wrapper-linux-x86.zip
- build-wrapper-macosx-x86 --out-dir ./x/logout xcodebuild clean build
- build-wrapper-win-x86-64.exe --out-dir ./x/logout MSBuild.exe /t:Rebuild /nodeReuse:False
- build-wrapper-linux-x86-64 --out-dir ./x/logout make clean all
- gem install xcpretty  #https://github.com/Backelite/xcpretty.git
- xcodebuild clean build | xcpretty -r json-compilation-database -o ./x/logout/logout.json
- chcp 65001 && set msbuild="x\Community\MSBuild\15.0\Bin\MsBuild.exe" && rmdir /S/Q build && mkdir build && cd build && CMake -G "Visual Studio xx" -T v141 .. && %Msbuilder% xx.sln /p:PlatformToolset=v141 /m:1 /t:build /p:Configuration=Release;WarningLevel=3 /fileLogger /fileLoggerParameters:WarningsOnly;LogFile=./x/logout/vcout.log;Verbosity=detailed;Encoding=UTF-8;/maxcpucount:4 && cd .. && sonar-scanner
- vi sonar-project.properties
```ini
sonar.cfamily.build-wrapper-output=./x/logout
sonar.sources=./
sonar.projectKey=xx
sonar.projectName=xx
sonar.language=objc #c++|cxx|ObjC|Swift
sonar.objectivec.workspace=xx.xcworkspace
sonar.objectivec.project=xx.xcodeproj
sonar.objectivec.appScheme=xx
sonar.sourceEncoding=UTF-8
sonar.junit.reportsPath=sonar-reports/
sonar.objectivec.oclint.report=sonar-reports/oclint.xml
sonar.swift.infer.report=infer-out/report.json
sonar.objectivec.excludedPathsFromCoverage=./x/logout/logout.json
sonar.cxx.vc.reportPaths=./x/logout/vcout.log
sonar.cxx.vc.encoding=UTF-8
//vc 2010-2015 正则在配置文件中需要\\作为\, 下面用\表示, 真正使用需要\\
sonar.cxx.vc.regex=(.*>)?(?<file>.*)\((?<line>\d+)\)\x20:\x20warning\x20(?<id>C\d+):(?<message>.*)
//vc 2017 https://github.com/SonarOpenCommunity/sonar-cxx/wiki/sonar.cxx.vc.regex
sonar.cxx.vc.regex=(.*>)?(?<file>.*)\((?<line>\d+)\):\x20warning\x20(?<id>C\d+):\x20(?<message>.*)
//vc 2019 微软命令需要在shell=cmd中执行
sonar.cxx.vc.regex=(?>[^>]*+>)?(?<file>(?>[^\\]{1,260}\\)*[^\\]{1,260})\((?<line>\d{1,5})\)\x20?:\x20warning\x20(?<id>C\d{4,5}):\x20?(?<message>.*)
```