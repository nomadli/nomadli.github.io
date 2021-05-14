---
layout: post
title: centos 服务安装
subtitle: centos 服务安装
date: 2018-05-24 09:07:00
author: nomadli
header-img: ../img/bg-coffee.jpeg
catalog: true
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

# rsyslog

- $PreserveFQDN on #用于正确的获取主机名，暂时应该没用到
- $FileOwner root #存储的文件属主
- $FileGroup root  #文件属主
- $FileCreateMode 0644 #生成的文件权限
- $DirCreateMode 0755 #生成的目录权限
- $Umask 0022
- $PrivDropToUser root #可以删除日志的用户
- $PrivDropToGroup root #可以删除日志的用户组
- $ActionFileEnableSync on 实时同步磁盘
- $OmitLocalLogging on 禁止所有socket日志, 所有通过imjournal
- $IMJournalStateFile imjournal.state
- $WorkDirectory /var/lib/rsyslog
- $AllowedSender tcp, 192.168.37.100/24
- $IncludeConfig /etc/rsyslog.d/*.conf 加载子配置
- template(name="xx" type="string|list" option.jsonf="on" string="%msg%\n") {...}
    - 如果主机跟tag都没有指定, 传出消息会从msg中查找tag
    - name                  名称,唯一引用模板的名称
    - type                  模板类型list,subtree,string,plugin
        - list              列表类型适合复杂定义,类似结构体,数据库的表,以key value成对出现
            - constant(outname="@version" value="1" format="jsonf")  固定输出 "@version":"1"
            - property(outname="msg:" name="msg" format="jsonf")     生成日志的内容
            - property(name="timereported" dataFormat="rfc3339" caseConversion="lower")
                - outname   在结构化输出(非直接文本)中的key如json "outname": "$name",
                - name      要引用的日志内容
                - format    输出格式csv,json(无段头json格),jsonf(完整的json),jsonr(无双反斜线),jsonfr(jsonf+jsonr)
                - dateformat日期格式
                - caseconversion lower、upper 字母转大小写
                - controlcharacters  控制字符转义: escape转义字符,` `替换为空格,drop删除
                - securepath         生成日志文件绝对路径名
                - position.from      从1开始的字符位置
                - position.to        从1开始的字符位置
                - position.relativetoend 到结尾的相对位置
                - field.number       数字字段
                - field.delimiter    十进制数字
                - regex.expression   正则
                - regex.type         ERE|BRE
                - regex.nomatchmode  未匹配的动作
                - regex.match        匹配的整体字符串
                - regex.submatch     匹配的子字符串
                - droplastLf         删除行尾换行
                - mandatory          on 字段没有值也会传输, 默认off
                - spifno1stsp        RFC3164 标准
                - datatype           只对jsonf有效,number string auto bool
                - onEmpty            只对jsonf有效,空
        - subtree   用于符合CEE标准的层级结构日志
            ```
            set $!usr!tpl2!msg = $msg;
            set $!usr!tpl2!dataflow = field($msg, 58, 2);
            template(name="tpl2" type="subtree" subtree="$!usr!tpl2")
            ```
        - string            字符串类型的输出, 包含一个必须的参数string指定模板
            - %             %号之间为变量及格式,其余为常量字符串
        - plug              template(name="tpl4" type="plugin" plugin="mystrgen") 必须先load
    - option 可选参数
        - jsonf             生成json格式
        - sql               生成sql语句
        - stdsql            生成sql语句中单引号会替换为两个单引号 与mysql的NO_BACKSLASH_ESCAPES=on有关
    - RSYSLOG_TraditionalFileFormat     时间精度低
    - RSYSLOG_FileFormat                高精度时间
    - RSYSLOG_TraditionalForwardFormat  时间精度低
    - RSYSLOG_ForwardFormat             高精度时间
    - RSYSLOG_SysklogdFileFormat        内核日志格式
    - RSYSLOG_SyslogProtocol23Format
    - RSYSLOG_DebugFormat               不要在生产环境试验
    - 变量 用于定义模板
        - msg	            日志的信息内容，message。
        - rawmsg	        不转义的日志内容。转义是默认开启的,所以它有可能与socket中接收到的内容不同。
        - rawmsg-after-pri	几乎与rawmsg相同，但是删除了syslog PRI。
        - hostname	        打印该日志的主机名。
        - source	        hostname属性的别名。
        - fromhost	        接收的信息来自于哪个节点。这里是DNS解析的名字。
        - fromhost-ip	    接收的信息来自于哪个节点，这里是IP，本地的是127.0.0.1。
        - syslogtag	        信息标签。大致形如 programed[14321] 。
        - programname	    tag的一部分，就是上面的programed那个位置。
        - pri	            消息的PRI部分-未解码(单值)
        - pri-text	        文本形式的消息的PRI部分，并在括号中添加数值PRI(例如“local0.err<133>”)
        - iut	            InfoUnitType 一款监视器软件，在与监视器后端通信的时候使用
        - syslogfacility	设备信息，数字形式表示
        - syslogfacility-text	设备信息，文本形式表示
        - syslogseverity	日志严重性等级，数字形式表示
        - syslogseverity-text	日志严重性等级，文本形式表示
        - syslogpriority	同 syslogseverity
        - syslogpriority-text	同 syslogseverity-text
        - timegenerated	    高精度时间戳,是日志产生的时间
        - timereported	    时间戳。精度取决于日志中提供的内容(在大多数情况下，为秒级)是接收处理前的时间
        - timestamp	        同 timereported    timestamp:8:15表示取8-15位
        - protocol-version	IETF draft draft-ietf-syslog-protocol 中的 PROTOCOL-VERSION 字段的内容
        - structured-data	IETF draft draft-ietf-syslog-protocol 中的 STRUCTURED-DATA 字段的内容
        - app-name	        IETF draft draft-ietf-syslog-protocol 中的 APP-NAME 字段的内容
        - procid	        IETF draft draft-ietf-syslog-protocol 中的 PROCID 字段的内容
        - msgid	            IETF draft draft-ietf-syslog-protocol 中的 MSGID 字段的内容
        - inputname	        生成日志的输入模块的名称(如“imuxsock”、“imudp”)
        - jsonmesg	        日志以json表示。可能出现数据重复，譬如syslogtag包含着programname。
        - bom               指定日志字符编码
        - 在处理是的时间
            - $now	        当前日期时间戳，格式为YYYY-MM-DD (2020-07-08)
            - $year	        当前年份， 四位数 (2020)
            - $month	    当前月份， 两位数 (07)
            - $day	        当前月份的日期，两位数 (08)
            - $wday	        当前天数周几 ：0=Sunday,...6=Saturday
            - $hour	        当前小时（24小时机制），两位数(16)
            - $hhour	    半小时机值，就是0-29分钟显示0，30-59分钟显示1。
            - $qhour	    一刻钟机值，通过0-3显示，每15分钟一截。
            - $minute	    当前分钟数，两位数(57)
            - now-unixtimestamp unix时间搓
        - 属性操作
            - %msg:1:2%     取前两个字符
            - %msg:10:$%    取10字符开始以后的字符
            - %msg:::lowercase% 小写
            - %msg:R:.*Sev:. \(.*\) \[.*–end% 正则
            - %msg:R,<regexp-type>,<submatch>,<nomatch>,<match-number>
            - [在线生成](https://www.rsyslog.com/regex/)
            - [属性操作](https://www.rsyslog.com/doc/v8-stable/configuration/property_replacer.html)
- rule
    - 每个rule由规则及操作列表组成,
    - 所有规则都会被评估,选择最优,除非碰到stop或discard
    - RSYSLOG_DefaultRuleset 默认日志处理规则
    - :property, [!]compare-operation, "value"
        - contains isequal startswith regex ereregex
- if "服务名称1,服务名称2[. .= .!]日志等级" then {...}
    - 服务名称
        - auth(LOG AUTH) 安全和认证相关消息  (不推荐使用 authpriv 替代)
        - authpriv(LOG_AUTHPRIV)	安全和认证相关消息(私有的)
        - cron (LOG_CRON)	系统定时任务cront和at产生的日志
        - daemon (LOG_DAEMON)	与各个守护进程相关的曰志
        - ftp (LOG_FTP)	ftp守护进程产生的曰志
        - kern(LOG_KERN)	内核产生的曰志(不是用户进程产生的)
        - mark	服务内部的信息，时间标识
        - local0-local7 (LOG_LOCAL0-7)	为本地使用预留的服务
        - lpr (LOG_LPR)	打印产生的日志
        - mail (LOG_MAIL)	邮件收发信息
        - news (LOG_NEWS)	与新闻服务器相关的日志
        - syslog (LOG_SYSLOG)	存syslogd服务产生的曰志信息
        - user (LOG_USER)	用户等级类别的日志信息
        - uucp (LOG_UUCP>	uucp子系统的日志信息，uucp是早期Linux系统进行数据传递的协议，后来 也常用在新闻组服务中
        - * 所有服务
    - 连接符
        - . 大于等于指定等级的日志
        - .= 只保存等于指定等级的
        - .! 只保存不等于指定等级的
    - 日志等级
        - debug (LOG_DEBUG)	    一般的调试信息说明
        - info (LOG_INFO)	    基本的通知信息
        - nolice (LOG_NOTICE)	普通信息，但是有一定的重要性
        - warning(LOG_WARNING)	警吿信息，但是还不会影响到服务或系统的运行
        - err(LOG_ERR)	        错误信息, 一般达到err等级的信息已经可以影响到服务成系统的运行了
        - crit (LOG_CRIT)	    临界状况信思，比err等级还要严®
        - alert (LOG_ALERT)	    状态信息，比crit等级还要严重，必须立即采取行动
        - emerg (LOG_EMERG)	    疼痛等级信息，系统已经无法使用了
        - *	代表所有日志等级
        - none                  忽略所有
    - 日志位置
        - 绝对路径               保存到文件
            - 路径前添加\- 符号表示使用内存缓存
        - 系统设备              /dev/lp0 第一台打印机 /dev/console 控制台
        - 网络 @ip:port UDP  @@(o,z9)ip:port TCP  o采用IETF新日志分隔标准,默认是\r\n, z9 zip压缩
        - 用户 *所有用户
        - ~ 丢弃日志
- module(load="imfile" xx=xx yy=yy) 加载模块插件
    - [插件](https://www.rsyslog.com/doc/v8-stable/configuration/modules/index.html)
    - imuxsock  unixsock 模块
    - imklog    内核日志printk
    - immark    日志标记模块
    - imfile    文件监控模块
        - 会记住上次位置、但如果服务停止后正好文件滚动,会导致内容丢失
        - module(load="imfile" xx=xx yy=yy)
            - Mode              inotify监听文件变化默认速度快|polling轮循  $ModLoad加载时默认是polling
            - readTimeout       默认0不超时, 单位秒
            - timeoutGranularity默认1超时检测间隔
            - sortFiles         文件排序处理配置
            - PollingInterval   轮循模式时,轮循间隔单位秒,默认10秒
            - statefile.directory 默认为全局的WorkDirectory配置,目录需要提前创建,用于存储模块状态
        - input(type="imfile" xx=xx yy=yy)
            - File              指定要监控的文件绝对路径,支持通配符
            - Tag               日志前加上Tag字符串
            - Facility          指定syslog设备 local0(default)...
            - Severity          指定syslog日志等级 notice(default)
            - PersistStateInterval 指定默认读多少行时将状态记录到状态文件,默认0只在日志文件切换后记录
            - startmsg.regex    多行日志时,指定日志开头的正则, 多行会合并为1行
            - endmsg.regex      多行日志结束正则
            - readTimeout       多行日志等待超时时间秒,0一直等到下个日志正则匹配才发生上一个
            - readMode          0每行都是新消息, 1 每个新行间有一个空行, 2 每个新行前没有空格或table
            - escapeLF          0不专业\r\n 1 转义\r\n
            - escapeLF.replacement 将\r\n替换为什么,当escapeLF打开但没设置是会替换为#012
            - MaxLinesAtOnce    在轮循模式下,对一个文件一次最多处理多少行,然后切换到另一个监控的文件,0处理完一个再一个
            - MaxSubmitAtOnce   一次处理的数据量 目前是1024, 对内存有影响
            - deleteStateOnFileDelete 当日志文件删除时删除对应的状态文件,默认on, 当如果要迁移日志可能会重传日志
            - Ruleset           指定使用的规则名称
            - addMetadata       是否添加metadata到日志中对象中,默认-1
                - filename
                - fileoffset
            - addCeeTag         是否添加@cee cookie到日志对象中 默认off
            - reopenOnTruncate  但日志文件滚动后,磁盘文件大小小于偏移量时,reopen日志,默认off
            - MaxLinesPerMinute 每分钟最大日志行,超出会丢弃,默认0 不丢弃
            - MaxBytesPerMinute 每分钟最大字节数,超出会丢弃,默认0 不丢弃
            - trimLineOverBytes 每行最大字节数,超出会丢弃,默认0 不丢弃,readMode为0或2才有效
            - freshStartTail    当第一次监控日志文件(无转态文件),丢弃旧日志
            - discardTruncatedMsg 当消息太长, 会被截断为下一个新消息,on时会直接丢弃,默认off
            - msgDiscardingError 截断消息时会显示错误消息, 默认on
            - needParse         默认off直接将消息转发至输出模块, on进入解析器
            - persistStateAfterSubmission 日志处理后写状态文件,默认off
    - imudp     udp模块
        - $SUDPServerRun        指定udp端口
    - imtcp     tcp模块
        - $InputTCPServerRun    指定TCP端口
        - $InputTCPMaxSessions  指定最大链接数
    - omkafka
        - module(load="omkafka")
        - action(type="omkafka" broker=["10.13.88.190:9092"] xx=xx yy=yy)
            - Broker            数组集群机器列表
            - Topic             使用的主题
            - DynaTopic         根据模板生成主题
            - DynaTopic.Cachesize动态主题缓冲个数
            - Partitions.Auto   默认off随机发生到partition,on自动选择一个partition
            - Partitions.number 指定主题分区个数, 用于负载均衡,默认不设置
            - Partitions.useFixed 使用指定的分区
            - Key               partitions.auto=on时用来选择partition
            - DynaKey           与Key相同,基于模板生成Key
            - errorFile         错误消息保存路径,json格式
            - statsFile         状态文件路径
            - ConfParam         kafa参数组成的数组
            - TopicConfParam    topic参数数组
            - Template          日志模板
            - closeTimeout      等待模块关闭的时间
            - resubmitOnFailure 默认off失败的消息不重发,on超过kafka的size限制的消息也不会重发
            - KeepFailedMessages 重启后继续发送失败的消息,需要resubmitOnFailure=on
            - failedMsgFile     失败消息保存文件
- action()
    - name                      自定义名字,不设置会自动生成一个1-n的值
    - type                      使用的插件名字
    - writeAllMarkMessages      是否开启心跳消息,默认on,此消息告知服务所有消息已经发生完成
    - execOnlyEveryNthTime      每间隔多少条发送一条消息其它丢弃,默认0
    - execOnlyEveryNthTimeout   当每隔几条丢弃时,设置秒杀来重新计数
    - errorfile                 指定错误信息保存文件,内容是json
    - execOnlyOnceEveryInterval 指定每隔多少秒执行一次, 对警告有用, 对日志就全丢了, 单位秒
    - execOnlyWhenPreviousIsSuspended   上一次提交确认再执行下次提交, 默认off
    - repeatedmsgcontainsoriginalmsg    估计是重复消息判断, 默认off
    - resumeRetryCount          失败重试次数, 默认0, -1一直重试
    - resumeInterval            重试时间间隔默认30秒,每失败10次间隔会*(重试次数/10+1)
    - resumeIntervalMax         最大重试间隔,默认1800秒
    - reportSuspension          当遇到暂停问题是报告
    - reportSuspensionContinuation 故障恢复报告
    - copyMsg                   默认off, 使用引用计数共享消息对象
- $ActionFileDefaultTemplate 模板名 设置默认模板
- :fromhost-ip, !isequal, "127.0.0.1" action(type="omfile" file="/var/log/xx")  条件匹配
- :rawmsg, contains, "xxx" action(type=”omfwd” target=”remote” protocol=”tcp” …)写到tcp服务器
- *.* ?DynFile;MyTemplate  由于DynFile是一个模板文件名,所以?代表文件名称会变化, -?表示带缓存动态文件名
- input(type="模块名", 参数=, ...)
```c
if prifilt("mail.info") then {
    action(type="omfile" file="/var/log/maillog")
    action(type=”omfwd” target=”remote” protocol=”udp” …)
    action(type=”omfwd” target=”remote” protocol=”tcp” …)
    action(type=”omusrmsg” users=”user” …)
    stop
}

local6.*  @xxx:aaa
main_queue(
    queue.workerthreads="10"        # threads to work on the queue
    queue.dequeueBatchSize="1000"   # max number of messages to process at once
    queue.size="50000"              # max queue size
)
module(load="imfile" PollingInterval="5") #加载文件监控模块,5秒刷新
module(load="omkafka")
$PreserveFQDN on                    #get the hostname
############################ php curl log #####################################
input(type="imfile"
      File="/data1/ms/log/php_common/php_slow_log"
      Tag="tag1"
      Facility="local5"
      freshStartTail="on"
      deleteStateOnFileDelete="off" 
      reopenOnTruncate="on"  ##日志切割Rsyslog不去读取文件，使用此选项
)
local5.* @xxx:bbb
########################## redis log  ########################################
$template redislog,"%$myhostname%`redislog`%msg%"
ruleset(name="redislog") {
    action(
        broker=["10.13.88.190:9092","10.13.88.191:9092","10.13.88.192:9092","10.13.88.193:9092"]
        type="omkafka"
        topic="redis-log"
        template="redislog"
        partitions.auto="on"
     )
  }
input(type="imfile"
      File="/data1/ms/log/front/redislog.log"
      Tag=""
      PersistStateInterval="1000"
      reopenOnTruncate="on"
      addMetadata="on"
      ReadMode="1"          #1代表多行日志之间以空格区分
      escapeLF="on"
      ruleset="redislog"
      freshStartTail="on"
      reopenOnTruncate="on" #日志文件有类似logrotate的分隔的话需要在分隔后重新打开
     )
...重复 $template ruleset input
template(name="xxx" type="string|list" string="%hostname% %msg%")
或
$InputFileName /var/log/http/access.log
$InputFileTag Apache1:
$InputFileStateFile state-Apache1

$InputRunFileMonitor 激活
```

# zookeeper

- znode     数据节点
    -  类型
        - 持久节点          创建|删除|子节点
        - 持久顺序节点      创建|删除|子节点|子节点序号
        - 临时节点          创建|链接断开消失|无子节点
        - 临时顺序节点      创建|链接断开消失|子节点|子节点序号
    - 状态
        - czxid             节点被创建时的事务ID
        - mzxid             节点最后一次被更新时的事务ID
        - ctime             节点被创建的时间
        - mtime             节点最后一次被修改的时间
        - version           节点的版本号
        - cversion          其子节点的版本号
        - aversion          节点的ACL版本号
            - CREATE        创建子节点的权限
            - READ          获取节点数据和子节点列表的权限
            - WRITE         更新节点数据的权限
            - DELETE        删除子节点的权限
            - ADMIN         设置节点 ACL 的权限
        - ephemeralOwner    创建该临时节点的会话的sessionID持久节点该值为0
        - dataLength        节点数据内容的长度
        - numChildre        该节点的子节点个数
        - pzxid             该节点子节点列表最后一次被修改时的事务ID,子节点内容修改不改变
- /etc/hostname

    172.168.30.11   zookeeper01
    ...


- /vdb/em_services/zookeeper-3.4.12/zoo.cfg

        tickTime=2000                           #毫秒,类似cpu tick,用于内部时钟
        initLimit=10                            #leader等待follower启动并完成数据同步的超时tick数
        syncLimit=5                             #leader与follower间心跳检测最大延时tick数
        dataDir=/vdb/zookeeper-data/data        #数据快照
        dataLogDir=/vdb/zookeeper-data/log      #事物日志
        clientPort=2181                         #服务端口
        clientPortAddress                       #多IP机器,不同IP不同端口时使用
        snapCount=100000                        #或环境变量zookeeper.snapCount 多少次事务后做一次快照
        preAllocSize=65536                      #或环境变量zookeeper.preAllocSize 事务日志文件预分配文件大小KB
                                                #preAllocSize = snapCount * 每次事务的日志量
        forceSync=yes                           #或环境变量zookeeper.forceSync 事物提交时强制刷盘事物日志
        autopurge.snapRetainCount=80            #保留多少快照和对应的事物日志,设置比3小自动设置为3
        autopurge.purgeInterval=8760            #单位小时, 定时清理间隔, 0不清理
        minSessionTimeout=2                     #如果客户端设置的会话超时时间小于x*tick, 设置为x*tick
        maxSessionTimeout=20                    #如果客户端设置的会话超时时间大于x*tick, 设置为x*tick
        maxClientCnxns=60                       #一个IP可以跟一台zookeeper建立多少链接, 0不限制
        jute.maxbuffer=1048575                  #必须是环境变量,字节,一个ZNode可以存储的最大数据,所有机器必须一致
        fsync.warningthresholdms=1000           #必须是环境变量,毫秒,日志同步磁盘时间的报警阈值
        zookeeper.globalOutstandingLimit=1000   #必须是环境变量,最大请求堆积
        zookeeper.leaderServes=yes              #必须是环境变量,客户端是否可以链接leader
        zookeeper.SkipAcl=no                    #必须是环境变量,是否跳过ACL权限检查
        zookeeper.cnxTimeout=5000               #必须是环境变量,毫秒,选举Leader过程中TCP链接创建超时时长
        server.1=zookeeper01:2888:3888          #1 zookeeper id #2888 消息端口  #3888 选举接口
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
- rpm -qa | grep redis rpm -ql redis
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


- cp redis-4.0.9/redis.service /etc/systemd/system/

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

# kafka
- 分为 broker、producter、consumer三部分配置
- broker->server.properties
    - broker.id                             #唯一ID, 改变IP,不改ID不影响consumers
    - listeners=PLAINTEXT://:p              #服务监听地址
    - advertised.listeners=PLAINTEXT://     #写入zookeeper的地址
    - num.network.threads=3                 #处理网络消息的最大线程数
    - num.io.threads=8                      #处理消息及磁盘IO的线程数,大于磁盘数
    - socket.send.buffer.bytes=102400       #发送缓冲区大小
    - socket.receive.buffer.bytes=102400    #接收缓冲区大小
    - socket.request.max.bytes=104857600    #最大请求大小
    - log.dirs                              #数据存放路径,多个地址用`,`分隔,对应不同磁盘
    - num.partitions=1                      #每个topic默认几个分区
    - num.recovery.threads.per.data.dir=1   #每个数据目录日志恢复或停机刷盘时的线程数
    - offsets.topic.replication.factor=1    #主题offsets的副本数,只在初始化时有用,一旦有数据修改无用
    - transaction.state.log.replication.factor=1    #主题数据状态副本数,只在初始化时有用,一旦有数据修改无用
    - transaction.state.log.min.isr=1       #
    - message.max.bytes                     #字节,消息体最大尺寸
    - background.threads                    #后台线程数,过期消息处理等
    - queued.max.requests                   #最大等待IO队列数,大于后会停止接收消息
    - host.name                             #主机地址,不设置监听所有IP

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

- touch /usr/lib/systemd/system/tomcat.service

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

- postgres 数据库

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
- yum install -y docker-ce-\*
- groupadd docker 添加 docker 组
- usermod -aG docker xxx 添加 xxx 到 daocker 组
- umount /xx/xx 卸载某分区
- pvscan 查看本机物理卷
- pvdisplay 查看物理卷详细信息
- pvcreate /dev/hda{2,3,..} 将逻辑分区创建为物理卷
- pvremove /dev/sdb2 删除物理卷
- pvchange 修改物理卷
- pvresize
- vgcreate vgname /dev/sdb2 物理卷 3 创建物理卷组
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


- 将 lv 合并为 thin pool

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
- rm -rf /root/.wp-cli/cache/_ /root/.composer/cache /home/_/.wp-cli/cache/_ /home/_/.composer/cache
- find -regex ".\*/core\.[0-9]+$" -delete
- find /var -name "\*.log" \( \( -size +50M -mtime +7 \) -o -mtime +30 \) -exec truncate {} --size 0 \;
- rm -rf /root/.npm /home/_/.npm /root/.node-gyp /home/_/.node-gyp /tmp/npm-\*
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
- rm -rf /var/cache/_ /tmp/_ /var/log/\* && echo > /root/.bash_history && history -c

## selinux

- yum install policycoreutils-python 修改 seliunx 的配置
- semanage port -l |grep ssh 查看允许的端口
- semanage port -m -t ssh_port_t -p tcp 23 修改一个允许的端口
- semanage port -a -t ssh_port_t -p tcp 23 添加一个允许的端口
- semanage port -d -t ssh_port_t -p tcp 23 删除一个允许端口

## firewalld

- firewall-cmd --permanent --list-port
- firewall-cmd --permanent --zone=public --add-port=80/tcp 添加端口
- firewall-cmd --permanent --zone=public --add-port=80-90/tcp 添加端口范围
- firewall-cmd --permanent --zone=public --remove-port=23/tcp 删除端口
- firewall-cmd --reload

## bind bind-chroot namedmanager
- https://github.com/conceptant/bind9-namedmanager/blob/master/Dockerfile