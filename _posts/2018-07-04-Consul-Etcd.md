---
layout: post
title:  "服务发现与配置中心"
date:   2018-07-04 19:35:00
categories: S
excerpt: 服务发现与配置中心
---

- [Consul](https://github.com/hashicorp/consul)
- [Etcd](https://github.com/coreos/etcd)

# 选择Consul
- 全部服务自建,无需第三方服务
- 服务发现及配置中心为一体
- 跨机房、跨云
- 自带web界面

# Consul 基本信息
- 端口占用
```
8300                    其他consul的传入请求。仅限TCP
8301                    LAN中的raft协议。TCP和UDP。
8302                    WAN中的raft协议。TCP和UDP。
8500                    HTTP API通信。仅限TCP。
8600                    解析DNS查询。TCP和UDP。
```
- 配置优先级              命令行>环境变量>配置文件
- 环境变量CONSUL_UI_LEGACY是否启用旧版UI,true启用
- SIGHUP                重新加载配置
- INT                   正常退出服务
- USR1                  查看运行日志,保留1分钟的日志
- -config-file          加载配置文件,可以多次指定配置,会合并配置
- -config-dir           指定加载配置文件的目录,按字母顺序加载文件并合并
- -config-format        配置文件的格式,不指定从扩展名检测,json或hcl
- -hcl                  命令行设置HCL格式的配置片段,可以多次指定
- -dev                  启动开发者模式
- -pid-file             指定pid存储文件
- 基本配置
```
{
  "server": true,                                   服务器
  "node_name": "consul-01",                         等同-node,名称
  "server_name": "consul-01",                       覆盖node_name,确保证书名称与我们声明的主机名匹配
  "disable_host_node_id": true,                     禁止使用主机名称作为node_id,默认true
  "node_id": "consul-01-201808021807",              可自动生成
  "bind_addr": "192.168.8.11",                      等同-bind,指定监听的ip,必须唯一,0.0.0.0检测到多个可用,会报错,go-sockaddr
  "advertise_addr": "192.168.8.11",                 等同-advertise,对外服务地址,默认=bind地址,go-sockaddr
  "advertise_addr_wan": "192.168.8.11",             等同-advertise-wan,跨机房服务地址,默认=advertise地址,go-sockaddr
  "serf_wan": "192.168.8.11",                       等同-serf-wan-bind,指定跨机房gossip绑定地址,默认与bind相同,go-sockaddr
  "serf_lan": "192.168.8.11",                       等同-serf-lan-bind,指定机房内gossip绑定地址,默认与bind相同,go-sockaddr
  "client_addr": "192.168.8.11",                    等同-client,指定RPC、DNS、HTTP服务地址,go-sockaddr
  "bootstrap": false,                               第一台启动时将自己设置为领导者,多台这样启动会无法达到共识,必须与server一起
  "bootstrap_expect": 3,                            指定预期服务器数量 不能与bootstrap一起使用,必须与server一起
  "rejoin_after_leave": false,                      等同-rejoin,重新加入上次退出的集群
  "start_join": [                                   等同-join(可多次),启动时加入的本机房代理,任意失败则启动失败
    "192.168.8.12"        
  ],        
  "retry_join": [                                   启动时加入的本机房代理,但链接失败时重试
    "192.168.8.12"        
  ],        
  "retry_interval": "30s",                          加入本机房代理失败,重试间隔,默认30
  "retry_max": 0,                                   加入本机房代理失败,重试次数,默认0无限重试
  "start_join_wan": [                               等同-join_wan(可多次),启动时加入的其它机房代理,任意失败则启动失败
    "192.168.8.12"        
  ],        
  "retry_join_wan": [                               启动时加入的其它机房代理,但链接失败时重试
    "192.168.8.12"        
  ],        
  "retry_interval_wan": "30s",                      加入其它机房代理失败,重试间隔,默认30
  "retry_max_wan": 0,                               加入其它机房代理失败,重试次数,默认0无限重试 
  "node_meta": {                                    附加信息,最多64对,key 1-128字符;value 0-512字符,字母数字-和,不能以consul-开头,
    "node-info":"this is vm consul node"            _value 默认RFC1464编码,rfc1035-开头在DNS TXT请求中逐字编码
  },        
  "datacenter": "nomadli-vm",                       机房名称
  "domain": "nomad",                                指定服务注册中域名,默认consul
  "data_dir": "/root/consul-data",                  数据保存位置
  "log_level": "INFO",                              日志级别,默认info,trace、debug、info、warn、err
  "enable_syslog": false,                           等同-syslog,写入系统日志
  "syslog_facility": "nomadli-vm",                  默认LOCAL0,系统日志的组标识
  "verify_incoming": false,                         传入连接使用TLS,默认false
  "verify_incoming_rpc": false,                     RPC连接使用TLS,默认false
  "verify_incoming_https": false,                   传入的HTTPS连接使用TLS,默认false
  "verify_outgoing": false,                         传出连接使用TLS,默认false
  "verify_server_hostname": false,                  验证服务器证书server.<datacenter>.<domain>主机名称,默认false
  "key_file": "xx/xx.key",                          consul服务与consul代理之间私钥
  "cert_file": "xx/xx.crt",                         consul服务与consul代理之间公钥
  "ca_path": "xx/xx",                               PEM编码的证书保存路径
  "ca_file": "xx/xx.crt",                           PEM编码的私钥证书,verify_incoming,verify_outgoing
  "tls_min_version": "tls12",                       TLS最低版本,tls10,tls11,tls12,默认tls12
  "tls_cipher_suites": [                            启用的加密算法列表
    "AES-256"
  ],
  "tls_prefer_server_cipher_suites": true,          协商加密算法时优先使用服务器的设置
  "encrypt": "bm9tYWRsaUAxNTE3NTI3OQ==",            设置raft加密密钥,16字节的Base64,consul keygen可以生成,默认保存
  "encrypt_verify_incoming": true,                  可选参数,传入的raft通信强制加密,默认true
  "encrypt_verify_outgoing": true,                  可选参数,传出的raft通信强制加密,默认为true
  "disable_keyring_file": false,                    默认false保存密钥为文件,true时每次启动需要传入密钥
  "acl_datacenter": "nomadli-vm",                   指定权限控制的数据中心。所有服务器和数据中心必须与其达成一致
  "acl_master_token": "123456",                     设置令牌,用于ACL启动,可忽略,通过/v1/acl/bootstrap获取自动创建的
  "acl_enforce_version_8": true,                    使用新版ACL,默认true
  "acl_agent_master_token": "123456",               在ACL服务器失效或服务器失效时使用,可操作链接新的服务
  "acl_agent_token": "123456",                      在ACL服务器失效或服务器失效时使用,仅某台agent可以使用
  "enable_acl_replication": true,                   启用ACL缓存复制
  "acl_replication_token": "123456",                在非ACL机房使用,复制缓存ACL的token
  "acl_token": "123456",                            普通服务器使用的token,当这些服务没有实现获取ACL
  "acl_ttl": "30s",                                 Token过期时间默认30s
  "acl_default_policy": "deny",                     默认为allow。允许任何未明确禁止的操作,deny阻止
  "acl_down_policy": "extend-cache",                默认extend-cache,ACL异常使用缓存, allow允许,deny禁止,async-cacheTTL过期后拒绝
  "disable_anonymous_signature": true,              禁用匿名签名
  "disable_remote_exec": true,                      禁用远程执行调用,默认true
  "ui": true,                                       启用WEB-UIUI
  "enable_agent_tls_for_checks": false,             健康检查需要双向TLS的服务,默认false
  "enable_script_checks": false,                    默认false禁止在代理上使用脚本做状态检测
  "protocol": 3,                                    指定协议版本,consul -v查看支持的协议版本,默认最新
  "raft_protocol": 3,                               指定raft版本,默认为3
  "raft_snapshot_threshold": 16384,                 指定修改多少条后做快照,默认为16384
  "raft_snapshot_interval": "30s",                  指定保存快照的时间间隔,默认为30s
  "check_update_interval": "5m"                     检测更新镜像层的时间间隔,默认5m,
  "discard_check_output": false,                    不保存健康检查数据
  "discovery_max_stale": 0                          数据过期时间,默认0每次从master读取,响应头X-Consul-Effective-Consistency标记是否缓存数据
  "autopilot": true,                                自动化维护集群
  "cleanup_dead_servers": true,                     自动清除异常服务器,默认true
  "last_contact_threshold": "200ms",                允许服务器失联的最长时间,默认200ms
  "max_trailing_logs": 250,                         服务器可监听领导者最大日志条数,默认250条
  "server_stabilization_time": "10s",               服务器需要稳定运行多久才可以加入控制集群,默认10s
  "session_ttl_min": "10s",                         最小会话TTL,默认10秒,心跳保持
  "skip_leave_on_interrupt": true,                  客户模式默认为false,服务器模式默认true,Ctrl-C客户从集群删除
  "reconnect_timeout": "72h",                       从群集中彻底删除故障节点的时间,默认72小时,s m h,必须>=8h
  "reconnect_timeout_wan": "72h",                   从WAN池中彻底删除控制故障服务器的时间,默认72小时,必须>=8h
  "performance": {                                  raft服务配置
    "leave_drain_time": "5s",                       在选举期间外部请求停留的保持时间,以允许重试请求,默认5秒
    "raft_multiplier": "5s",                        Raft检测领导者故障和执行领导者选举所需的时间,最大10秒,默认5秒
    "rpc_hold_timeout": "7s"                        在领导选举期间重试内部请求的保持时间,默认为7s
  },
  "translate_wan_addrs": false,                     默认false,解析其它机房地址,返回头含X-Consul-Translate-Addresses为远程
  "dns_config": {                                   DNS相关设置
    "enable_truncate": true,                        UDP查询返回超3条记录,设置截断标志,TCP获得完整记录
    "only_passing": false,                          排除运行状况为警告或严重节点,默认false仅排除失败的节点
    "recursor_timeout": "5s",                       递归查询上游DNS服务器超时时间
    "allow_stale": true,                            允许缓存查询数据,默认为true
    "max_stale": "87600h",                          最大缓存时间,默认10年,因为服务器通常只落后几毫秒
    "service_ttl": {                                服务解析后用户端缓存TTL,可用*通配符服务,默认0
      "*": "0s",                                    没有单独列出的都0秒过期
      "web": "30s"      
    },      
    "node_ttl": "0s",                               node缓存时间,默认0,秒s,分钟m
    "disable_compression": false,                   压缩DNS响应,默认false
    "udp_answer_limit": 3,                          UDP DNS响应的最大记录数。不推荐使用此设置
    "a_record_limit": 3,                            限制A,AAAA,ANY DNS(TCP和UDP)记录数,默认无限制,不适用SRV记录
    "enable_additional_node_meta_txt": true         将节点元数据添加到DNS响应附加部分中,默认true
  },      
  "recursors": [                                    等同-recursor(多次),指定上游DNS服务,可多次指定
    "8.8.8.8"       
  ],      
  "http_config": {                                  RPC配置
    "response_headers": {                           向HTTP API响应添加头
      "Access-Control-Allow-Origin": "*"      
    },      
    "block_endpoints": [                            禁止部分API,前缀匹配后返回403,默认空,
      "/v1/acl"                                     禁止所有acl的RPC调用
    ],      
    "leave_on_terminate": false,                    收到TERM信号向群集发下线消息,代理默认true,服务器默认false
    "limits": {                                     防刷配置
        "rpc_rate": 0,                              单个令牌每秒最大请求速率,默认为无限
        "rpc_max_burst": 1000                       令牌最大值,默认为1000个
    }     
  },
  "unix_sockets": {                                 设置Unix套接字文件权限,unix://file
    "user": "root",                                 拥有套接字文件的用户名称或ID
    "group": 0,                                     套接字文件的组ID,仅支持数字ID
    "mode": 777                                     套接字文件的权限
  },
  "connect": {                                      启用代理功能,为没有改造、不支持证书的服务提供代理
    "enabled": false,                               默认false,不提供服务
    "ca_provider": "consul",                        权限控制方式,consul、vault
    "ca_config": {                                  根据ca_provider设置不同的配置
      "private_key": "BEGIN RSA PRIVATE",           ca_provider="consul"时使用,用于CA的私钥的PEM内容。
      "root_cert": "BEGIN CERTIFICATE",             ca_provider="consul"时使用,用于CA的根证书的PEM内容。
      "address": "x.x.x.x",                         ca_provider="vault"时使用,要连接的Vault服务器的地址。
      "token": "xxxxxxx",                           ca_provider="vault"时使用,要使用的Vault令牌。
      "root_pki_path": "x/x",                       ca_provider="vault"时使用,需root权限。
      "intermediate_pki_path": "x/x"                ca_provider="vault"时使用,临时路径。
    },      
    "proxy": {                                      代理设置选项
      "allow_managed_api_registration": false,      允许通过代理配置服务,默认false
      "allow_managed_root": false,                  允许以root系统权限,默认false
      "proxy_defaults": {                           设置代理默认参数,定义exec_mode、daemon_command、config默认值
      }     
    }     
  },      
  "telemetry": {                                    局域网位置预测
    "circonus_api_token": "xxx",                    创建/管理检查的有效API令牌,启用度量标准管理
    "circonus_api_app": "xxx",                      API令牌关联的有效应用名称,默认consul
    "circonus_api_url": "xxx",                      URL默认https://api.circonus.com/v2
    "circonus_submission_interval": "10s",          指标提交时间间隔,默认10秒
    "circonus_submission_url": "xxx",               HTTPT RAP检查的Check API对象
    "circonus_check_id": "xx",                      HTTPT RAP检查的检查ID
    "circonus_check_force_metric_activation":false, 强制激活已存在且当前不活动的度量标准,默认false
    "circonus_check_instance_id": "xxx",            唯一标识来自此实例的ID,默认hostname:application_name
    "circonus_check_search_tag": "xxx",             标记,与实例ID结合缩小搜索范围,默认service:application_name
    "circonus_check_display_name": "xxx",           显示名称
    "circonus_check_tags": [                        逗号分隔的附加标记列表
      "xx"
    ],
    "circonus_broker_id": "xxx",                    创建新Broker时要使用的特定ID
    "circonus_broker_select_tag": "xxx",            一个Broker标签,未指定ID时,默认为空
    "disable_hostname": false,                      是否使用主机名预先添加运行时遥测,默认false。
    "dogstatsd_addr": "x.x.x.x:78",                 支持DogStatsD协议的地址host:port
    "dogstatsd_tags": [                             全局发送到DogStatsD的标签列表
      "xxx:xx"
    ],
    "filter_default": true,                         默认true未设置过滤器时允许所有指标,false无过滤器则不发送指标
    "metrics_prefix": "nomad",                      数据的前缀,默认consul
    "prefix_filter":[                               过滤规则列表,通过前缀允许/阻止度量标准
      "+consul.raft.apply",                         //启用
      "-consul.http",                               //禁用
    ],                                              规则重叠,详细优先,相同规则,禁用优先
    "prometheus_retention_time": "0s",              prometheus保持时间间隔,0为不保存
    "statsd_address": "x.x.x.x:78",                 支持statsd协议的地址host:port,仅支持UDP
    "statsite_address": "x.x.x.x:78",               支持statsite协议的地址host:port,仅支持TCP流
  },
  "addresses": {                                    设置bind地址,默认client_addr
    "dns": "192.168.8.11",
    "http": "192.168.8.11",
    "https": "192.168.8.11"
  },
  "ports": {                                        等同于-xxx-port
    "dns": 8600,                                    默认8600
    "http": 8500,                                   默认8500
    "https": -1,                                    默认-1,禁止使用
    "serf_lan": 8301,                               默认8301
    "serf_wan": 8302,                               默认8302
    "server": 8300,                                 默认8300,RPC接口
    "proxy_min_port": 20000,                        默认20000,代理服务随机端口起始,0-0表示必须指定
    "proxy_max_port": 20255,                        默认20255,代理服务随机端口结束,0-0表示必须指定
  },
  "watches": [                                      一个监视列表,允许数据修改时调用外部程序
    {
      "type": "key",
      "key": "foo/bar/baz",
      "handler_type": "http",
      "http_handler_config": {
        "path":"https://localhost:8000/watch",
        "method": "POST",
        "header": {"x-foo":["bar", "baz"]},
        "timeout": "10s",
        "tls_skip_verify": false
      }
    }
  ],
  "enable_debug": false,                            启用一些其他调试功能,仅用于设置运行时分析HTTP
  "disable_update_check": true,                     禁用自动检查更新,企业版无法设置
  "upgrade_version_tag": "",        
  "segment": "nomadli-vm-01",                       企业版功能,设置同机房不同网段名称
  "segments": [                                     企业版功能,只能在服务器上设置,网段列表
    {       
      "name": "xx",                                 网段名称名称,必须是长度在1到64个字符之间的字符串
      "bind": "x.x.x.x",                            用于段的gossip绑定地址,默认为-bind的地址
      "port": 8303,                                 用于段的gossip端口
      "advertise": "x.x.x.x",                       用于段的raft的广播地址,默认为-advertise的地址
      "rpc_listener": false                         在-bind的rpc端口上启动单独的RPC侦听器
    }       
  ],
  "non_voting_server": false,                       企业版功能,不参与raft仲裁,但接收数据并保存,用于只读扩展
  "redundancy_zone_tag": "01",                      企业版功能,将局域网分成多个区
  "disable_upgrade_migration": false,               企业版功能,自动升级策略
}
```
- CLI consul 其它命令
```
CONSUL_HTTP_ADDR                地址 127.0.0.1:8500 或 unix://xx/xx.sock
CONSUL_HTTP_TOKEN               API访问令牌 aba7cbe5-879b-999a-07cc-2efd9ac0ffe
CONSUL_HTTP_AUTH                用户名:密码对 operations:JPIMCmhDHzTukgO6
CONSUL_HTTP_SSL                 默认为false不启用HTTPS true启用HTTPS
CONSUL_HTTP_SSL_VERIFY          默认为true指定SSL证书验证; false不验证
CONSUL_CACERT                   TLS的CA文件 ca.crt
CONSUL_CAPATH                   TLS的CA证书目录的路径
CONSUL_CLIENT_KEY               TLS的客户机密钥文件
CONSUL_CLIENT_CERT              TLS的客户端证书文件的路径
CONSUL_TLS_SERVER_NAME          TLS连接时用作SNI主机的服务器名称 consulserver.domain
xxx -h                          帮助
xxx -help                       帮助
-autocomplete-install           安装自动补全
--version                       显示版本
agent                           启动代理
catalog                         查询数据
    catalog datacenters         返回数据中心列表
    catalog nodes               返回节点列表 -service=redis指定服务的节点
    catalog services            返回服务列表 -node=worker-01指定节点的服务
connect
    ca                          与证书服务器交互
        get-config              获取CA服务器配置
        set-config              设置CA服务器配置
            -ca-file            TLS的CA文件的路径
            -ca-path            TLS的CA证书目录
            -client-cert        verify_incoming启用时用于TLS的客户端证书文件路径
            -client-key         verify_incoming启用时用于TLS的客户机密钥文件路径
            -http-addr          代理地址(IP或DNS),须包含端口,默认http://127.0.0.1:8500
            -tls-server-name    通过TLS连接时用作SNI主机的服务器名称
            -token              请求使用的ACL令牌,未指定则查询HTTP地址处的代理的令牌
            -datacenter         要查询的数据中心的名称,未指定则查询HTTP地址处的代理的数据中心
            -stale              允许任何服务器(非领导者)响应此请求,默认false
            -config-file        指定新的配置文件
```
- [太多都转为UI点击](https://www.consul.io/docs/commands/index.html)

# consul 健康检测
- 退出代码 0 通过 1 警告 其他 失败
- "status": "passing" 指定健康检测的初始状态
- "service_id": "web-app" 指定健康检测涉及的服务
- script 必须enable_script_checks设置为true
```
{
  "check": {
    "id": "mem-util",
    "name": "Memory utilization",
    "args": ["/usr/local/bin/check_mem.py", "-limit", "256MB"],
    "interval": "10s",
    "timeout": "1s"
  }
}
```
- ttl 服务主动put状态pass,warn,fail,update
```
{
  "check": {
    "id": "web-app",
    "name": "Web App Status",
    "notes": "Web app does a curl internally every 10 seconds",
    "ttl": "30s"
  }
}
```
- TCP 链接端口,域名解析为ipv4和ipv6时任意一个可以链接算健康
```
{
  "check": {
    "id": "ssh",
    "name": "SSH TCP on port 22",
    "tcp": "localhost:22",
    "interval": "10s",
    "timeout": "1s"
  }
}
```
- HTTP 允许字段header、method默认GET
```
{
  "check": {
    "id": "api",
    "name": "HTTP API on port 5000",
    "http": "https://localhost:5000/health",
    "tls_skip_verify": false,
    "method": "POST",
    "header": {"x-foo":["bar", "baz"]},
    "interval": "10s",
    "timeout": "1s"
  }
}
```
- Docker API 需要设置Docker Http端口,环境变量\$DOCKER_HOST,enable_script_checks为true
```
{
  "check": {
    "id": "mem-util",
    "name": "Memory utilization",
    "docker_container_id": "f972c95ebf0e",
    "shell": "/bin/bash",
    "args": ["/usr/local/bin/check_mem.py"],
    "interval": "10s"
  }
}
```
- gRPC检测需要符合gRPC check标准,grpc_use_tls设置使用STL
```
{
  "check": {
    "id": "mem-util",
    "name": "Service health status",
    "grpc": "127.0.0.1:12345",
    "grpc_use_tls": true,
    "interval": "10s"
  }
}
```
- 别名检测
```
{
  "check": {
    "id": "web-alias",
    "alias_service": "web"
  }
}
```
# consul 监听状态
- type 监听类型
    - key           k/v
    - keyprefix     k/v目录前缀     "prefix": "foo/",
    - services      服务列表变化    无参数
    - nodes         节点列表变化    无参数
    - service       监听指定服务    "service": "redis",
    - checks        监听监控状态    "service" 或 "state"
    - event         监听用户事件    "name": "web-deploy"
- 脚本回调
```
{
  "type": "key",                监听类型,此处为k/v
  "key": "foo/bar/baz",         监听的具体key
  "handler_type": "script",     执行类型脚本
  "args": ["/usr/bin/my-service-handler.sh", "-redis"],
  "datacenter": "cd1",          数据中心,可选
  "token": "xx",                ACL令牌,可选
  "handler": "?"
}
```
- http回调
```
{
  "type": "key",
  "key": "foo/bar/baz",
  "handler_type": "http",       执行类型http
  "http_handler_config": {      header中包含X-Consul-Index字段
    "path":"https://localhost:8000/watch",
    "method": "POST",
    "header": {"x-foo":["bar", "baz"]},
    "timeout": "10s",
    "tls_skip_verify": false
  }
}
```
# consul 使用
- DNS解析, 需要将外网DNS转发出去
```
node_name.node[.datacenter].<domain>                    节点查询
[tag.]service_name.service[.datacenter].<domain>        服务查询
_<service>._<protocol>[.service][.datacenter][.domain]  RFC 2782查询格式
<query or name>.query[.datacenter].<domain>             模板匹配查询
<service>.connect.<domain>                              connect服务
```
# consul connect
- 用来为各服务提供TLS加密代理,使用服务名来验证而无需ip绑定
- 启用connect需要在consul服务配置中添加connect {enabled = true}同时启用CA服务
```
PrivateKey      指定PEM格式connect验证私钥
RootCert        指定PEM根证书
"native": true  无线代理,服务本身支持通过consul获取证书验证等
```
- connect在服务定义配置中定义
```
{
  "service": {
    "name": "web",
    "port": 8080,
    "connect": {                                    所有字段可选
      "proxy": {
        "config": {
          "bind_address": "0.0.0.0",                TLS监听地址,默认agent监听地址
          "bind_port": 20000,                       TLS端口,不指定会从配置的随机端口中选
          "tcp_check_address": "192.168.0.1",       被健康检测的地址,端口使用bind_port
          "disable_tcp_check": false,               禁用tcp健康检测
          "local_service_address": "127.0.0.1:8080",被代理的地址:端口,默认local:服务定义端口
          "local_connect_timeout_ms": 1000,         链接被代理服务的超时时间
          "handshake_timeout_ms": 10000,            TLS握手链接超时时间
          "upstreams": [                            被代理服务需要访问的服务定义
            {
              "destination_type": "service",        被链接的类型,默认service
              "destination_name": "redis",          被链接的名称,在consul中查询使用(必须)
              "destination_datacenter": "dc1",      数据中心
              "local_bind_address": "127.0.0.1",    反向代理绑定地址,默认localhost
              "local_bind_port": 1234,              反向代理端口,被代理服务链接这个端口(必须)
              "connect_timeout_ms": 10000           链接超时,默认10000
            },
          ]
        }
      }
    }
  }
}
```
- 自定义代理
```
{
  "service": {
    "name": "web",
    "port": 8080,
    "connect": {
      "proxy": {
        "exec_mode": "daemon",                                  daemon唯一选项
        "command":   ["/usr/bin/my-proxy", "-flag-example"]
      }
    }
  }
}
```
- 修改默认代理
```
connect {
  proxy_defaults {
    command = ["/usr/bin/my-proxy"]
  }
}
```
- 手动代理,不由consul集群管理
```
{
  "Name": "redis-proxy",
  "Kind": "connect-proxy",
  "ProxyDestination": "redis",
  "Port": 8181
}
```
- 管理模式需设置CONSUL_PROXY_TOKEN CONSUL_PROXY_ID
- 设置规则 intention,拒绝优先允许,精确优先模糊,来源模糊优先目标模糊
```
consul intention create -deny web db            禁止web链接db,已经链接的不断开
consul intention create -allow web *            允许web到任何服务的链接
    -meta description='Hello there'             添加meta,key=description

service "web" {
  policy = "read"                               可以读取
  intention = "deny"                            不可以修改规则
}
```
# consul 权限管理系统ACL
- 角色
    - agent [Agent API](https://www.consul.io/api/agent.html),不可以执行检测及服务注册
    - event	[Event API](https://www.consul.io/api/event.html)
    - key [KV Store API](https://www.consul.io/api/kv.html)
    - keyring [Keyring API](https://www.consul.io/api/operator/keyring.html)
    - node
        - [Catalog API](https://www.consul.io/api/catalog.html)
        - [Health API](https://www.consul.io/api/health.html)
        - [Prepared Query API](https://www.consul.io/api/query.html), 
        - [Network Coordinate API](https://www.consul.io/api/coordinate.html)
        - [Agent API](https://www.consul.io/api/agent.html),只有节点、目录权限
    - operator [Operator API](https://www.consul.io/api/operator.html)除Keyring API
    - query	[Query API](https://www.consul.io/api/query.html)中qurey操作
    - service 以下API中的服务目录操作部分
        - [Catalog API](https://www.consul.io/api/catalog.html)
        - [Health API](https://www.consul.io/api/health.html),
        - [Prepared Query API](https://www.consul.io/api/query.html)
        - [Agent API](https://www.consul.io/api/agent.html)
    - session [Session API](https://www.consul.io/api/session.html)
- 任何角色无法修改,只能从启动脚本配置
    - [Status API](https://www.consul.io/api/status.html)
    - [Catalog API](https://www.consul.io/api/catalog.html)中数据中心的部分
- ACL只允许一个数据中心控制,其它数据中心从这里同步
- policy 可选值 read write deny
- [ACL配置方法](https://www.consul.io/docs/guides/acl.html)