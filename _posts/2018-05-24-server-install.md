---
layout:         post
title:          centos 服务安装
subtitle:       centos 服务安装
date:           2018-05-24 09:07:00
author:         nomadli
header-img:     ../img/bg-coffee.jpeg
catalog:        true
tags:
        - other
---

# JDK
- tar -zxvf jdk-8u171-linux-x64.tar.gz
- /etc/profile

        JAVA_HOME=/lib64/jdk1.8.0_171
        JRE_HOME=$JAVA_HOME/jre
        CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
        PATH=$JAVA_HOME/bin:$JRE_HOME/bin:$PATH
        export JAVA_HOME JRE_HOME CLASSPATH

# zookeeper
- /etc/hostname

            172.168.30.11   zookeeper01
            ...
            
- /vdb/em_services/zookeeper-3.4.12/zoo.cfg

        tickTime=2000
        initLimit=10
        syncLimit=5
        dataDir=/vdb/zookeeper-data/data
        dataLogDir=/vdb/zookeeper-data/log
        clientPort=2181
        #maxClientCnxns=60
        #autopurge.snapRetainCount=80
        #autopurge.purgeInterval=8760
        server.1=zookeeper01:2888:3888
        server.2=zookeeper02:2888:3888
        server.3=zookeeper03:2888:3888

- /vdb/zookeeper-data/data/myid
- vi /etc/systemd/system/zookeeper.service

        [Unit]
        Description=zookeeper Server 01
        After=syslog.target network.target remote-fs.target nss-lookup.target

        [Service]
        User=root
        Type=forking
        Environment=ZOO_LOG_DIR=/vdb/zookeeper-data/log
        ExecStart=/vdb/em_services/zookeeper-3.4.12/bin/zkServer.sh start
        ExecStop=/vdb/em_services/zookeeper-3.4.12/bin/zkServer.sh stop
        Restart=always

        [Install]
        WantedBy=multi-user.target
- iptables

        iptables -L -n --line-numbers
        iptables -I INPUT 1 -p tcp --dport 2181 -j ACCEPT
        iptables -I INPUT 2 -p tcp --dport 2888 -j ACCEPT
        iptables -I INPUT 3 -p tcp --dport 3888 -j ACCEPT
        iptables-save > /etc/sysconfig/iptables
        systemctl start zookeeper
        em_services/zookeeper-3.4.12/bin/zkServer.sh status

# redis
- yum install -y http://rpms.famillecollet.com/enterprise/remi-release-7.rpm
- yum --enablerepo=remi install redis
- rpm -qa | grep redis   rpm -ql redis
- redis.conf

        daemonize   如果需要在后台运行，把该项改为yes
        pidfile     配置多个pid的地址 默认在/var/run/redis.pid
        bind        绑定ip，设置后只接受来自该ip的请求
        port        监听端口，默认是6379
        loglevel    分为4个等级：debug verbose notice warning
        logfile     用于配置log文件地址
        databases   设置数据库个数，默认使用的数据库为0
        save        设置redis进行数据库镜像的频率。
        rdbcompression 在进行镜像备份时，是否进行压缩
        dbfilename  镜像备份文件的文件名
        Dir         数据库镜像备份的文件放置路径
        Slaveof     设置数据库为其他数据库的从数据库
        Masterauth  主数据库连接需要的密码验证
        Requriepass 设置 登陆时需要使用密码
        Maxclients  限制同时使用的客户数量
        Maxmemory   设置redis能够使用的最大内存
        Appendonly  开启append only模式
        Appendfsync 设置对appendonly.aof文件同步的频率（对数据进行备份的第二种方式）
        vm-enabled  是否开启虚拟内存支持 （vm开头的参数都是配置虚拟内存的）
        vm-swap-file设置虚拟内存的交换文件路径
        vm-max-memory设置redis使用的最大物理内存大小
        vm-page-size 设置虚拟内存的页大小
        vm-pages 设置交换文件的总的page数量
        vm-max-threads 设置VM IO同时使用的线程数量
        Glueoutputbuf 把小的输出缓存存放在一起
        hash-max-zipmap-entries 设置hash的临界值
        Activerehashing 重新hash
        
- cp redis-4.0.9/redis.service  /etc/systemd/system/

        [Unit]
        Description=Redis Server
        After=syslog.target network.target remote-fs.target nss-lookup.target

        [Service]
        ExecStart=/vdb/em_services/redis-4.0.9/redis-server /vdb/em_services/redis-4.0.9/redis.conf --daemonize no
        ExecStop=/vdb/em_services/redis-4.0.9/src/redis-cli -p 6379 shutdown
        Restart=always

        [Install]
        WantedBy=multi-user.target

- systemtcl daemon-reload
- iptables -I INPUT -p tcp --dport 6379 -j ACCEPT
- iptables-save > /etc/sysconfig/iptables

# Tomcat
- /etc/profile

        CATALINA_HOME=/vdb/em_services/apache-tomcat-9.0.8
        CATALINA_BASE=/vdb/tomcat_data
        PATH=$PATH:$CATALINA_BASE/bin
        export CATALINA_BASE

- cp /vdb/em_services/apache-tomcat-9.0.8/conf/server.xml /vdb/tomcat_data/conf/

        800->80

- touch /vdb/em_services/apache-tomcat-9.0.8/bin/setenv.sh

        CATALINA_PID="$CATALINA_BASE/tomcat.pid"
        JAVA_OPTS="-server -XX:PermSize=256M -XX:MaxPermSize=1024M -Xms4096M -Xmx16384M -XX:MaxNewSize=256m"
        -server     一定要作为第一个参数，在多个CPU时性能佳
        -Xms 初始Heap堆大小，使用的最小内存,cpu性能高时此值应设的大一些
        -Xmx Java heap堆最大值，使用的最大内存
        -XX:PermSize 非堆内存初始大小、这里不会被垃圾回收,与堆得总和不能超过物理内存
        -XX:MaxPermSize         非堆内存最大值
        -XX:newSize             垃圾回收时新堆初始大小
        -XX:MaxNewSize          垃圾回收时新堆的最大值
        -Xss512k                线程所初始消耗的栈
        -XX:AggressiveHeap      忽略堆大小设置,尽量使用内存及交换空间
        -verbose:gc             现实垃圾收集信息
        -Xloggc:gc.log          指定垃圾收集日志文件
        -Xmn                    young generation的heap大小，一般设置为Xmx的3、4分之一
        -XX:+UseParNewGC        缩短minor收集的时间
        -XX:+UseConcMarkSweepGC 缩短major收集的时间

- touch  /usr/lib/systemd/system/tomcat.service

        [Unit]
        Description=Tomcat
        After=syslog.target network.target remote-fs.target nss-lookup.target

        [Service]
        Type=forking
        PIDFile=/vdb/tomcat_data/tomcat.pid
        ExecStart=/vdb/em_services/apache-tomcat-9.0.8/bin/catalina.sh start
        ExecReload=/vdb/em_services/apache-tomcat-9.0.8/bin/catalina.sh restart
        ExecStop=/vdb/em_services/apache-tomcat-9.0.8/bin/catalina.sh stop

        [Install]
        WantedBy=multi-user.target

- mkdir

        /vdb/tomcat_data//logs
        /vdb/tomcat_data/temp

- iptables -I INPUT -p tcp --dport 80 -j ACCEPT
- iptables-save > /etc/sysconfig/iptables

# jenkins
- /vdb/tomcat_data/webapp
- /etc/profile

        JENKINS_HOME=/vdb/jenkins_data
        export JENKINS_HOME

- touch /vdb/jenkins_data/log_parse
- 插件安装并全部更新

        Log Parser
        Mask Passwords

- 系统设置

        执行者数量              1           一个slave只能有一个在执行
        生成前等待时间          0           触发编译时等待时间,通过svn变化触发会导致多次触发，所以要等待
        Environment variables             LANG en_US.UTF-8
        Mask Passwords  File Parameter/Non-Stored Password Parameter/Password Parameter
        Console Output Parsing            /vdb/jenkins_data/log_parse

- Configure Global Security

        启用安全            Jenkins专有用户数据库
        项目矩阵授权策略

# journalctl 服务日志

    journalctl -xe                  查看服务错误原因
    journalctl --disk-usage         查看日志文件大小
    journalctl --vacuum-size=10M    指定日志文件大小

# maven
- SET MAVEN_OPTS=-Xmx1024m

# kong
- postgres数据库

        docker run -itd p 5432:5432 -v xx/db:/var/lib/postgresql/data \
        --restart=always --name kongdb -e POSTGRES_USER=kong -e POSTGRES_DB=kong

# docker
- yum install -y yum-utils device-mapper-persistent-data lvm2

        安装虚拟逻辑卷 lvm
        
- yum-config-manager --enable extras
- yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
- yum-config-manager --enable docker-ce-edge
- yum makecache fast
- yum list docker-ce.x86_64 --showduplicates | sort -r
- yum -y install docker-ce-18.03.1.ce
- rpm --import keyfile 导入公钥
- yum install -y https://zeroc.com/download/ice/3.7/el7/ice-repo-3.7.el7.noarch.rpm
- cur https://download.docker.com/linux/centos/7/x86_64/stable/Packages/containerd.io-1.4.4-3.1.el7.x86_64.rpm
- cur https://download.docker.com/linux/centos/7/x86_64/stable/Packages/docker-ce-cli-20.10.5-3.el7.x86_64.rpm
- cur https://download.docker.com/linux/centos/7/x86_64/stable/Packages/docker-ce-20.10.5-3.el7.x86_64.rpm
- cur https://download.docker.com/linux/centos/7/x86_64/stable/Packages/docker-ce-rootless-extras-20.10.5-3.el7.x86_64.rpm
- yum install -y docker-ce-*
- groupadd docker 添加docker组
- usermod -aG docker xxx 添加xxx到daocker组
- umount /xx/xx 卸载某分区
- pvscan 查看本机物理卷
- pvdisplay 查看物理卷详细信息
- pvcreate /dev/hda{2,3,..} 将逻辑分区创建为物理卷
- pvremove /dev/sdb2 删除物理卷
- pvchange 修改物理卷
- pvresize
- vgcreate vgname /dev/sdb2 物理卷3 创建物理卷组
- vgscan vgdisplay vgremove vgchange
- vgextend 添加新的物理卷
- vgreduce 删除物理卷
- vgmerge 合并两个物理卷组
- lvcreate 创建逻辑卷

        lvcreate -L 15G -n lvname vgname
        lvcreate -l 50%VG -n lvname vgname
        lvcreate -l 100%FREE -n lvname vgname
        lvcreate --wipesignatures y -L 15G -n lvname vgname 擦除签名

- lvscan lvdisplay lvremove lvchange lvconvert 
- lvextend 扩容量

        lvextend -L 10G -rf  /dev/vgname/lvname  强制resize 增加到10G
        lvextend -L +10G -rf  /dev/vgname/lvname  强制resize 增加10G
        
- lvreduce 缩容量

        lvreduce -L 10G -f -r /dev/vgname/lvname 强制resize 缩小到10G
        lvreduce -L -10G -f -r /dev/vgname/lvname 强制resize 缩小10G
        
- 将lv合并为thin pool

        lvconvert -y --zero n -c 512K --thinpool vgname/lvname --poolmetadata vgname/lvname
        需要在vgdisplay实际容量中占用8G,因此两个lv大小应该留足8G
         
- lsblk 查看磁盘结构
- df -hT 查看磁盘使用情况
- touch /etc/lvm/profile/vgname-lvname.profile

        activation {
            thin_pool_autoextend_threshold=80   容量使用到80开始扩容
            thin_pool_autoextend_percent=20     扩容20%
        }
        lvchange --metadataprofile vgname-lvname vgname/lvname
        lvs -o+seg_monitor 查看是否设置成功,监视磁盘使用
        dmsetup ls 查看精简卷
        
- mkfs.ext4 /dev/vgname/lvname 格式化
- vi /etc/fstab 开机挂载精简卷

        mkdir docker_disk
        /dev/vgname/lvname      /docker_disk        ext4    defaults    0 0
        
- /etc/docker/daemon.json

        {
            "storage-driver": "devicemapper",
            "storage-opts": [
                "dm.thinpooldev=/dev/mapper/vgname-lvname",
                "dm.use_deferred_removal=true",
                "dm.use_deferred_deletion=true"
            ],
            "registry-mirrors": [
                "https://hub-mirror.c.163.com"
            ],
            "insecure-registries":[
                "10.228.11.12:5000"
            ],
            "bip":"10.10.1.1/24",
            "fixed-cidr":"10.10.1.0/24"
        }
          
- systemctl enable docker
- systemctl start docker
- systemctl status docker
- /etc/sysctl.conf

        if [[ ! -d "/proc/sys/net/bridge/" ]]; then
	       echo "net.bridge.bridge-nf-call-ip6tables = 1" >> /etc/sysctl.conf
	       echo "net.bridge.bridge-nf-call-iptables = 1" >> /etc/sysctl.conf
	       echo "net.bridge.bridge-nf-call-arptables = 1" >> /etc/sysctl.conf
	       sysctl -p
        fi  
              
- docker info 查看配置
- crontab -e 定时任务

        0 1 * * * find /storage/app/package -mtime +30 -name "*.zip" -exec rm -rf {} \;
        8 1 * * * rm -rf /storage/app/build/Run.log.3
        9 1 * * * rm -rf /storage/app/build/run.log.3

## Vagrant 一个虚拟机开发测试环境自动搭建工具

## centos 精简
- yum remove -y firewalld iptables audit
- yum -y install yum-utils
- package-cleanup --quiet --leaves --exclude-bin | xargs yum remove -y
- package-cleanup --oldkernels --count=1
- rm -rf /root/.wp-cli/cache/* /root/.composer/cache /home/*/.wp-cli/cache/* /home/*/.composer/cache
- find -regex ".*/core\.[0-9]+$" -delete
- find /var -name "*.log" \( \( -size +50M -mtime +7 \) -o -mtime +30 \) -exec truncate {} --size 0 \;
- rm -rf /root/.npm /home/*/.npm /root/.node-gyp /home/*/.node-gyp /tmp/npm-*
- 删除无用的系统语言
```
localectl status
localedef --list-archive | grep -v en_US.utf8 | xargs localedef --delete-from-archive
mv /usr/lib/locale/locale-archive /usr/lib/locale/locale-archive.tmpl
build-locale-archive
cd /usr/share/locale/
ls | grep -v -i ^en | grep -v locale.alias | xargs rm -rf
rm -rf en@* en_AU en_CA en_GB en_NZ enm
cd /usr/share/i18n/locales/
ls | grep -v es_US | xargs rm -rf
```
- /etc/grub2.cfg
- rm -rf /var/cache/* /tmp/* /var/log/* && echo > /root/.bash_history && history -c

## selinux
- yum install policycoreutils-python 修改seliunx的配置
- semanage port -l |grep ssh 查看允许的端口
- semanage port -m -t ssh_port_t -p tcp 23 修改一个允许的端口
- semanage port -a -t ssh_port_t -p tcp 23 添加一个允许的端口
- semanage port -d -t ssh_port_t -p tcp 23 删除一个允许端口

## firewalld
- firewall-cmd --permanent --list-port
- firewall-cmd --permanent --zone=public --add-port=80/tcp 添加端口
- - firewall-cmd --permanent --zone=public --add-port=80-90/tcp 添加端口范围
- firewall-cmd --permanent --zone=public --remove-port=23/tcp 删除端口
- firewall-cmd --reload
