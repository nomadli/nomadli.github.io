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

# Consul 命令
- 端口占用
```
8300                    其他consul的传入请求。仅限TCP
8301                    LAN中的raft协议。TCP和UDP。
8302                    WAN中的raft协议。TCP和UDP。
8500                    HTTP API通信。仅限TCP。
8600                    解析DNS查询。TCP和UDP。
```
- 配置优先级              命令行>环境变量>配置文件
- consul agent          启动consul服务
```
consul agent -server -ui -bootstrap-expect=1 -data-dir=/tmp/consul -node=host_name -bind=172.20.20.10 -enable-script-checks=true -config-dir=/etc/consul.d
http://localhost:8500/ui
环境变量CONSUL_UI_LEGACY=true   启用旧版UI
basic_config.json              最先加载
extra_config.json              合并入当前配置
SIGHUP                         重新加载配置
INT                            正常退出服务
USR1                           查看运行日志,保留1分钟的日志
-data-dir                      指定数据文件夹
-node                          指定服务节点名称
-node-id                       十六进制36个字符,唯一id,保存在数据目录
-disable-host-node-id          默认true不使用主机名作为节点id,使用随机值
-node-meta                     最多64个键/值对,键1到128个字符,值字母数字-和_,不能以consul-前缀开头,0到512,默认RFC1464编码,rfc1035-开头在DNS TXT请求中逐字编码
-datacenter                    指定服务所在的机房、数据中心名称
-server                        指定以服务形式运行
-advertise                     对外服务地址,默认=bind地址,可设go-sockaddr模板
-advertise-wan                 跨机房地址,默认=advertise地址
-bootstrap                     第一台启动时将自己设置为领导者,多台这样启动会无法达到共识
-bootstrap-expect              指定预期服务器数量 不能与bootstrap一起使用,必须与server一起
-bind                          指定监听的ip,必须唯一,0.0.0.0检测到多个可用,会报错,go-sockaddr
-serf-wan-bind                 指定跨机房绑定地址,默认与bind相同,go-sockaddr
-serf-lan-bind                 指定机房内绑定地址,默认与bind相同,go-sockaddr
-client                        指定可以访问Http及DNS的ip列表或0.0.0.0,go-sockaddr
-config-file                   加载配置文件,可以多次指定配置,会合并配置
-config-dir                    指定加载配置文件的目录,按字母顺序加载文件并合并
-config-format                 配置文件的格式,通常从扩展名检测配置文件的格式。json或hcl
-dev                           启动开发者模式
-encrypt                       设置raft加密密钥,16字节Base64,consul keygen可以生成,默认保存
-disable-keyring-file          默认false保存密钥为文件,true时每次启动需要传入密钥
-dns-por                       默认8600
-domain                        指定服务注册中域名,默认consul
-enable-script-checks          默认false禁止在代理上执行脚本检测状态
-hcl                           允许在命令行设置配置
-http-port                     默认8500
-join                          启动时加入的代理,可多次指定添加多个,任意失败则启动失败
-retry-join                    启动时加入的代理,但链接失败时重试
-retry-interval                加入重试间隔默认30秒
-retry-max- -join              重试次数,默认1,0为无限重试
-join-wan                      启动时加入的跨网代理,可多次指定添加多个,任意失败则启动失败
-retry-join-wan                启动时加入跨网代理,但链接失败时重试
-retry-interval-wan- -join-wan 默认30秒
-retry-max-wan- -join-wan      默认1
-log-level                     默认info,trace、debug、info、warn、err
-syslog                        写入系统日志
-pid-file                      指定pid存储文件
-protocol                      指定协议版本,consul -v查看支持的协议版本,默认最新
-raft-protocol                 指定raft版本,默认为3
-raft-snapshot-threshold       指定修改多少条后做快照,默认为16384
-raft-snapshot-interval        指定保存快照的时间间隔,默认为30s
-recursor                      指定上游DNS服务,可多次指定
-rejoin                        重新加入上次退出的集群
-ui                            启用web界面及http路由
-segment                       企业版功能,设置网段名称
-non-voting-server             企业版功能,不参与raft仲裁,但接收数据并保存,用于只读扩展
```
- 权限配置
```
acl_datacenter                  指定权限控制的数据中心。所有服务器和数据中心必须与其达成一致
acl_default_policy              默认为allow。允许任何未明确禁止的操作。deny会被阻止
acl_down_policy                 默认extend-cache,无法联系ACL时使用缓存, allow允许所有操作,deny限制所有操作,async-cache在TTL过期后拒绝。
acl_agent_master_token          在acl_enforce_version_8设置为true时使用
acl_agent_token                 
acl_enforce_version_8           默认true使用新的ACL
acl_master_token                使用默认的令牌,用于ACL datacenter启动
acl_replication_token           在非ACL datacenter使用来同步ACL
acl_token                       代理在请求服务器时使用
acl_ttl                         过期时间默认30s
addresses                       绑定的ip,dns、http、https
advertise_addr                  等同命令行参数-advertise,对外服务地址
serf_wan                        等同命令行参数-serf-wan-bind,跨机房地址
serf_lan                        等同命令行参数-serf-lan-bind,局域网地址
advertise_addr_wan              等同命令行参数-advertise-wan,跨机房服务地址
autopilot                       自动化维护集群
    cleanup_dead_servers        自动清除失败的服务器、自动接入新服务器,默认true
    last_contact_threshold      允许服务器失联的最长时间,默认200ms
    max_trailing_logs           服务器可监听领导者最大日志条数,默认250条
    server_stabilization_time   服务器需要稳定运行多久才可以加入控制集群,默认10s
    redundancy_zone_tag         企业版功能,将局域网分成多个区
    disable_upgrade_migration   企业版功能,升级策略
bootstrap                       等同命令行参数-bootstrap,第一台启动时将自己设置为领导者
bootstrap_expect                等同命令行参数-bootstrap-expect,指定预期服务器数量
bind_addr                       等同命令行参数-bind,指定监听的ip,必须唯一
ca_file                         PEM编码的私钥证书
ca_path                         PEM编码的私钥证书的路径
key_file                        PEM编码私钥的文件路径
cert_file                       PEM编码的公钥证书的文件
check_update_interval           检测更新镜像层的时间间隔,默认5m,
client_addr                     等同命令行参数-client,指定可以访问Http及DNS的ip列表
connect                         是否允许链接
    enabled                     默认false,不提供服务
    ca_provider                 权限控制,只支持consul和vault
    ca_config                   根据ca设置不同的配置
        private_key             ca_provider="consul"时使用,用于CA的私钥的PEM内容。
        root_cert               ca_provider="consul"时使用,用于CA的根证书的PEM内容。
        address                 ca_provider="vault"时使用,要连接的Vault服务器的地址。
        token                   ca_provider="vault"时使用,要使用的Vault令牌。
        root_pki_path           ca_provider="vault"时使用,需root权限。
        intermediate_pki_path   ca_provider="vault"时使用,临时路径。
    proxy                       代理设置选项
        allow_managed_api_registration  允许通过代理配置服务,默认false
        allow_managed_root              允许以root方式允许,默认false
    proxy_defaults              默认代理,定义exec_mode、daemon_command、config默认值
datacenter                      等同命令行参数-datacenter,数据中心名称
data_dir                        等同命令行参数-data-dir,数据保存路径
disable_anonymous_signature     禁用匿名签名删除重复
disable_host_node_id            等同命令行参数-disable-host-node-id,禁止使用ID
disable_remote_exec             禁用远程执行,默认true
disable_update_check            禁用自动检查更新
discard_check_output            不保存健康检查数据。
discovery_max_stale             允许旧数据时间,默认0必须从master读取,   
                                X-Consul-Effective-Consistency响应标头，标记是否过时读取
dns_config                      DNS查询服务方式配置
    allow_stale                 启用陈旧查询,默认为true
    max_stale                   最大过时时间,默认10年(87600h),因为服务器通常只落后几毫秒
    node_ttl                    默认0,秒s,分钟m,指查询后过期时间
    service_ttl                 使用服务上设置的TTL,使用*通配符服务,默认0
    enable_truncate             true时UDP DNS查询返回超3条记录,设置截断标志,TCP获得完整记录
    only_passing                true时排除运行状况为警告或严重节点,默认false仅排除失败的节点
    recursor_timeout            递归查询上游DNS服务器时使用的超时
    disable_compression         默认false,压缩DNS响应
    udp_answer_limit            UDP DNS响应的最大记录数。不推荐使用此设置
    a_record_limit              限制A,AAAA,ANY DNS(TCP和UDP)记录数,默认无限制,不适用SRV记录
    enable_additional_node_meta_txt 默认true将节点元数据的TXT记录添加到DNS响应的附加部分中
domain                          等同命令行参数-domain,指定服务注册中域名,默认consul
enable_acl_replication          无需通过设置复制令牌即可启用ACL复制
acl_replication_token           通过设置复制令牌启用ACL复制
enable_agent_tls_for_checks     使用代理的TLS配置进行检查需要双向TLS的服务,默认false
enable_debug                    启用一些其他调试功能,仅用于设置运行时分析HTTP
enable_script_checks            等同命令行参数-enable-script-checks,在代理上执行脚本检测状态
enable_syslog                   等同命令行参数-syslog,写入系统日志
syslog_facility                 默认LOCAL0,系统日志的组标识
encrypt                         等同命令行参数-encrypt,设置加密密钥,16字节Base64
encrypt_verify_incoming         可选参数,传入的raft通信强制加密,默认true
encrypt_verify_outgoing         可选参数,传出的raft通信强制加密,默认为true
disable_keyring_file            等同命令行参数-disable-keyring-file,默认false保存密钥为文件
http_config                     设置HTTP API
    block_endpoints             默认空,前缀匹配后返回403,如设置/v1/acl,禁止所有acl
    response_headers            向HTTP API响应添加头,如{"x":{"x": {"x":"x"}}}
    leave_on_terminate          代理默认true,服务器默认false,代理收到TERM信号向群集其它发消息
    limits                      一个嵌套对象,配置代理强制执行的限制
        rpc_rate                单个令牌每秒最大请求速率,默认为无限
        rpc_max_burst           令牌最大值,默认为1000个
log_level                       等同命令行参数-log-level,默认info
node_id                         等同命令行参数-node-id,自定义节点ID
node_name                       等同命令行参数-node,指定节点名
node_meta                       将任意元数据键/值对与本地节点关联,如{"node_meta":{"x":"x"}}
performance                     一个嵌套对象，允许调整Consul中不同子系统的性能
    leave_drain_time            在选举期间外部请求停留的持续时间,以允许重试请求,默认5秒
    raft_multiplier             Raft检测领导者故障和执行领导者选举所需的时间,最大10秒,默认5秒
    rpc_hold_timeout            在领导选举期间重试内部请求的持续时间,默认为7s
ports                           这是一个嵌套对象,设置绑定端口
    dns                         DNS服务器,   -1禁用,默认8600
    http                        HTTP API,   -1禁用,默认8500
    https                       HTTPS API,  -1禁用,默认-1
    serf_lan                    Serf LAN端口,-1禁用,默认8301
    serf_wan                    Serf WAN端口,-1禁用,默认8302
    server                      服务器RPC端口,-1禁用,默认8300
    proxy_min_port              自动分配的托管代理的最小端口号,禁用使用指定端口,默认为20000
    proxy_max_port              自动分配的托管代理的最大端口号,默认为20255
protocol                        等同命令行参数-protocol,指定协议版本,consul -v查看
raft_protocol                   等同命令行参数-raft-protocol,raft协议版本
raft_snapshot_threshold         等同命令行参数-raft-snapshot-threshold,修改多少条后快照
raft_snapshot_interval          等同命令行参数-raft-snapshot-interval,快照时间间隔
reconnect_timeout               从群集中彻底删除故障节点的时间,默认72小时,s m h,必须>=8h
reconnect_timeout_wan           从WAN池中彻底删除控制故障服务器的时间,默认72小时,必须>=8h
recursors                       上游DNS服务器的地址、递归解析查询
rejoin_after_leave              等同命令行参数-rejoin,重新加入上次退出的集群
retry_join                      等同命令行参数-retry-join,启动时加入的代理,但链接失败时重试
retry_interval                  等同命令行参数-retry-interval,加入重试间隔默认30秒
retry_join_wan                  等同命令行参数-retry-join-wan,至少一个加入工作
retry_interval_wan              等同命令行参数-retry-interval-wan
segment                         等同命令行参数-segment,仅企业版
segments                        这是嵌套对象的列表,只能在服务器上设置,仅企业版
    name                        细分的名称,必须是长度在1到64个字符之间的字符串
    bind                        用于段的raft绑定地址,-bind如果未提供,则默认为该值
    port                        用于段的raft端口,必需
    advertise                   用于段的raft的广播地址,-advertise未提供,默认为该值
    rpc_listener                默认false,如果为true,在-bind的rpc端口上启动单独的RPC侦听器
server                          等同命令行参数-server
non_voting_server               等同命令行参数-non-voting-server,企业版功能,不参与raft仲裁
server_name                     覆盖node_name,确保证书名称与我们声明的主机名匹配
session_ttl_min                 最小会话TTL,默认10秒,心跳保持
skip_leave_on_interrupt         客户模式默认为false,服务器模式默认true,Ctrl-C客户从集群删除
start_join                      等同命令行参数-join,字符串
start_join_wan                  等同命令行参数-join-wan
telemetry                       这是一个嵌套对象,局域网位置预测
    circonus_api_token          创建/管理检查的有效API令牌,启用度量标准管理
    circonus_api_app            API令牌关联的有效应用名称,默认consul
    circonus_api_url            URL默认https://api.circonus.com/v2
    circonus_submission_interval指标提交时间间隔,默认10秒
    circonus_submission_urlcheck.config.submission_url  HTTPT RAP检查的Check API对象
    circonus_check_id           HTTPT RAP检查的检查ID
    circonus_check_force_metric_activation 强制激活已存在且当前不活动的度量标准,默认false
    circonus_check_instance_id  唯一标识来自此实例的ID,默认hostname:application_name
    circonus_check_search_tag   标记,与实例ID结合缩小搜索范围,默认service:application_name
    circonus_check_display_name 检查名称
    circonus_check_tags         逗号分隔的附加标记列表
    circonus_broker_id          创建新Broker时要使用的特定ID
    circonus_broker_select_tag  一个Broker标签,未指定ID时,默认为空
    disable_hostname            是否使用主机名预先添加运行时遥测,默认false。
    dogstatsd_addr              支持DogStatsD协议的地址host:port
    dogstatsd_tags              全局发送到DogStatsD的标签列表,每行xxx:xx
    filter_default              默认true未设置过滤器时允许所有指标,false无过滤器则不发送指标
    metrics_prefix              数据的前缀,默认consul
    prefix_filter               过滤规则列表,通过前缀允许/阻止度量标准
                                ["+consul.raft.apply",  //启用
                                "-consul.http",         //禁用
                                "+consul.http.GET"
                                ]规则重叠,详细优先,相同规则,禁用优先
    prometheus_retention_time   prometheus保持时间间隔,0为不保存
    statsd_address              支持statsd协议的地址host:port,仅支持UDP
    statsite_address            支持statsite协议的地址host:port,仅支持TCP流
tls_min_version                 TLS最低版本,tls10,tls11,tls12,默认tls12
tls_cipher_suites               逗号分隔的启用的加密算法列表
tls_prefer_server_cipher_suites 协商加密算法时优先使用服务器的设置
translate_wan_addrs             默认false,禁止为远程数据中心中的节点提供DNS和HTTP请求,
                                如果启用,返回头中含有X-Consul-Translate-Addresses:
                                /v1/catalog/nodes
                                /v1/catalog/node/<node>
                                /v1/catalog/service/<service>
                                /v1/health/service/<service>
                                /v1/query/<query or name>/execute
ui                              等同命令行参数-ui
unix_sockets                    设置Unix套接字文件权限
    user                        拥有套接字文件的用户名称或ID
    group                       套接字文件的组ID,仅支持数字ID
    mode                        套接字文件的权限
verify_incoming                 true所有传入连接使用TLS,默认false
verify_incoming_rpc             true所有RPC连接使用TLS,默认false
verify_incoming_https           true所有传入的HTTPS连接使用TLS,默认false
verify_outgoing                 true所有传出连接使用TLS,默认false
verify_server_hostname          true验证服务器提供的TLS证书与server.<datacenter>.<domain>
                                的主机名称,默认false
watches                         一个监视列表,允许数据修改时调用外部程序
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
# 服务配置
- 非注明必须都为可选
```
{
  "service": {
    "name": "redis",                        服务名,必须
    "id": "xxx",                            默认=服务名,防止服务重名设置
    "tags": ["primary"],                    可以用来区分服务版本、接口等,consul不关心
    "enable_tag_override": false,           是否允许修改tag,默认false
    "address": "",                          默认为代理地址
    "port": 8000,
    "meta": {                               最多64对k/v,k(A-Z a-z 0-9 _ -)
      "meta": "for my service"              k<=128,v<=256,token是acl保留的key
    },
    "checks": [                             健康检测名称为service:<service-id>[:num]
      {                                     检测类型为script,HTTP,TCP,TTL
        "args": ["/x/check_redis.py"],      script类型args,interval必须设置
        "interval": "10s",                  HTTP类型http,interval必须设置,method默认GET
        "timeout":"10s"                     TCP类型tcp,interval必须设置
      }                                     TTL类型ttl必须设置,由服务主动发心跳
    ],                                      timeout设置检测超时时间,
    "kind": "connect-proxy",                non-proxy则忽略下面字段
    "proxy_destination": "redis",           connect proxy描述
    "connect": {
      "native": false,                      true原生链接,无线proxy,不能有proxy项
      "proxy": {
        "command": [],
        "config": {}
      }
    }
  }
}
```
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