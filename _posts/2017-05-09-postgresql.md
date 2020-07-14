---
layout:         post
title:          postgresql
subtitle:       postgresql
date:           2017-05-09 11:17:00
author:         nomadli
header-img:     ../img/bg-coffee.jpeg
catalog:        true
tags:
        - other
---

* content
{:toc} 

# 限制
- 最大单个数据库大小     不限	
- 最大数据单表大小      32 TB
- 单条记录最大         1.6 TB
- 单字段最大允许       1 GB
- 单表允许最大记录数    不限
- 单表最大字段数       250 - 1600 (取决于字段类型)
- 单表最大索引数       不限

# 基本命令
- pg_ctl start -D /xxx/xxx 启动数据库
- pg_ctl stop -D /xxx/xxx -m fast 停止数据库
- psql -h 127.0.0.1 -p 1980 -U postgres 链接数据库
- createuser -E -P -I -h ip -p port -U user newuser 新用户
- create extension pg_stat_statements; 创建统计扩展模块
- \q 断开数据库链接
- \l 列出数据库
- \c dbname 切换数据库
- \dnS 查看模式
- \du 查看用户
- \dt 列出表
- \d tablename 查看表结构
- \z tablename 查看表权限
- create schema app; 创建schema
- show search_path 当前默认schema
- set search_path to schemaname; 或 alter database dbname set search_path to schemaname;
- alter default privileges in schema schemaname grant select，... on tables to dbuser；
- select * from pg_namespace; schema权限
- select * from current_user; 当前用户
- drop database dbname 删除数据库
- drop role rolename 删除模式或用户
- 权限 r -- SELECT ("read")
      w -- UPDATE ("write")
      a -- INSERT ("append")
      d -- DELETE
      D -- TRUNCATE
      x -- REFERENCES
      t -- TRIGGER
      X -- EXECUTE
      U -- USAGE
      C -- CREATE
      c -- CONNECT
      T -- TEMPORARY
- revoke all on database dbname from public;
- revoke all on schema sch_name from public;   
- revoke all on language plpgsql from public; 
- revoke all on table ... from public;  
- revoke all on function ... from public;
- INFORMATION_SCHEMA.role_table_grants 权限表
- select current_database(); 当前数据库
- show server_encoding; 当前数据库编码
- CREATE DATABASE dbname OWNER usename TEMPLATE template1 ENCODING 'utf8' LC_COLLATE collate LC_CTYPE ctype TABLESPACE tablespacename CONNECTION LIMIT 10
- grant connect on database dbname to "dbuse";允许链接  
- grant usage on schema sch_name to "dbuse";赋予模式使用权限
- grant select,insert,... on table users to "dbuse";
- select setval(pg_get_serial_sequence('table', 'colname'), number); 设置自增变量当前值

# 权限
- 新建数据库 默认允许任何用户连接、不允许除超级用户和owner外任何人创建schema、自动创建public角色(schema)、允许任何人在public下创建、查询对象
- 新建schema 默认除超级用户和owner其他用户无法查看和修改对象

# C API
1.  PQconnectdb host、hostaddr、port、dbname、connect_timeout、user、password
2.  PQfinish 释放链接
3.  PQreset 重置链接
4.  PQdb 返回连接的数据库名称
5.  PQuser 返回连接的用户名称
6.  PQpass 返回连接的用户名密码
7.  PQhost 返回连接的服务器主机名
8.  PQport 返回连接的端口
9.  PQstatus 返回连接的状态
10. PQparameterStatus 查看当前连接的某个参数设置
11. PQerrorMessage 返回最近连接操作产生的错误信息
12. PQexec 执行sql
13. PQexecParams 执行单句sql
14. PQprepare、PQexecPrepared、DEALLOCATE 预编译sql语句

# postgresql.conf参数
- 参数的名称都是不区分大小写的。取值boolean、integer、floating point和string表示。
- 内存单位KB、MB、GB，默认单位是数据块的个数，每个数据块的大小是8KB。
- 时间单位:ms、s、min、h、d，默认单位是毫秒。
- 每一行只能指定一个参数，空格和空白行都会被忽略。“#”表示注释，可以出现在配置文件的任何地方。
- 如果参数的值不是简单的标识符和数字，用单引号引起来。参数的值中有单引号，应该写两个单引号，或者在单引号前面加一个反斜杠。
- 使用include包含其它配置文件
- pg_ctl reload来通知数据库重新读取配置文件。
- 用户可以在自己建立的会话中执行命令SET修改某些配置参数的值，有些参数只有数据库超级用户才能使用SET命令修改它们。
- show来查看所有的数据库参数的当前值。
- 连接与认证
	01. 连接设置 listen_addresses(string) 启动时被设置。默认* ，监听所有的IP地址，可以写成机器的名字，也可以写成IP地址，不同的值用逗号分开。设成localhost，只能接受本地的客户端连接。
	02. port (integer) 启动时设置。默认值是5432。
	03. max_connections (integer)启动时设置。同时建立的最大的客户端连接的数目。默认值是100
	04. superuser_reserved_connections(integer)启动时设置。预留给超级用户的数据库连接数目。小于max_connections默认3。
	05. unix_socket_group (string)启动时设置。Unix-domain socket所在的操作系统用户组。默认值空串，用启动数据库的操作系统用户所在的组作为Unix-domain socket的用户组。
	06. unix_socket_permissions(integer)启动时设置。设置Unix-domain socket的访问权限，默认0770任何操作系统用户都能访问Unix-domain socket。写权限才有意义，读和执行权限是没有意义的。
	07. tcp_keepalives_idle (integer)任何时候设置。默认0，使用操作系统的默认值。设置TCP_KEEPIDLE属性。对Unix-domain socket建立的数据库连接没有任何影响。
	08. tcp_keepalives_interval (integer)任何时候被设置。默认0，使用操作系统的默认值。设置TCP_KEEPINTVL属性。对于通过Unix-domain socket建立的数据库连接没有任何影响。
	09. tcp_keepalives_count (integer)任何时候被设置。默认0，使用操作系统的默认值。设置TCP_KEEPCNT属性。对于通过Unix-domain socket建立的数据库连接没有任何影响。
	10. authentication_timeout (integer)只在conf中设置，指定时间内完成客户端认证操作，否则拒绝请求。单位秒，默认60。
	11. ssl (boolean)启动时设置。决定数据库是否接受SSL连接。默认值是off。
	12. ssl_ciphers (string)指定可以使用的SSL加密算法。（执行命令openssl ciphers –v查看系统支持的加密方式）。
- 资源消耗
	01. shared_buffers(integer)启动时设置。数据缓冲区块的个数，块大小8KB。不能小于128KB。默认1024。共享缓存
	02. temp_buffers(integer)任何时候被设置。临时表数据缓冲区块的个数，块大小8KB。默认1024。每个数据库进程的私有内存中。
	03. max_prepared_transactions (integer)启动时设置。同时处于prepared状态的事务的最大数目，设为0关闭prepared事务特性。它的值通常应该和max_connections的值一样大。默认5。
	04. work_mem (integer)任何时候设置。排序、哈希表使用缓冲区大小。缓存耗尽使用磁盘文件完成操作，默认1024KB。
	05. maintenance_work_mem (integer)任何时候设置。维护操作缓存大小。在数据库进程的私有内存中，默认16384KB。
	6. max_stack_depth (integer)任何时候设置，需超级用户修改。设置进程运行时STACK的最大值。超过，终止当前事务。这个值应该比操作系统设置的进程STACK的大小的上限小1MB。ulimit –s查操作系统的进程STACK的最大值。单位KB，默认100KB。
- Free Space Map(FSM)结构记载数据文件中每个数据块的可用空间的大小。需要新的物理存储空间，会在FSM中查找，如果FSM中没有一个数据页有足够的可用空间，系统就会自动扩展数据文件。所以，FSM如果太小，会导致系统频繁地扩展数据文件，浪费物理存储空间。命令VACUUM VERBOSE在执行结束以后，会提示当前的FSM设置是否满足需要，如果FSM的参数值太小，它会提示增大参数。在数据库的共享内存中，FSM只跟踪一部分数据块的可用空间信息。
	01. max_fsm_relations (integer)启动时设置。默认1000。决定FSM跟踪的表和索引的个数的上限。表和索引在FSM中占7个字节。
	02. max_fsm_pages (integer)启动时设置。它决FSM中跟踪的数据块的个数的上限。initdb在创建数据库集群时会根据物理内存的大小决定它的值。每个数据块在fsm中占6个字节的存储空间。它的大小不能小于16 * max_fsm_relations。默认20000。
- 内核资源
	01. max_files_per_process (integer)启动时设置。每个数据库进程能够打开的文件的数目。默认1000。
	02. shared_preload_libraries (string)启动时设置。启动时要加载的操作系统共享库文件。多个库文件用逗号分开。如果数据库在启动时未找到shared_preload_libraries指定的某个库文件，数据库将无法启动。默认空串。
- 垃圾收集 执行VACUUM 和ANALYZE命令时，因为它们会消耗大量的CPU与IO资源，而且执行一次要花很长时间，这样会干扰系统执行应用程序发出的SQL命令。为了解决这个问题，VACUUM 和ANALYZE命令执行一段时间后，系统会暂时终止它们的运行，过一段时间后再继续执行这两个命令。这个特性在默认的情况下是关闭的。将参数vacuum_cost_delay设为一个非零的正整数就可以打开这个特性。
用户通常只需要设置参数vacuum_cost_delay和vacuum_cost_limit，其它的参数使用默认值即可。VACUUM 和ANALYZE命令在执行过程中，系统会计算它们执行消耗的资源，资源的数量用一个正整数表示，如果资源的数量超过vacuum_cost_limit，则执行命令的进程会进入睡眠状态，睡眠的时间长度是是vacuum_cost_delay。vacuum_cost_limit的值越大，VACUUM 和ANALYZE命令在执行的过程中，睡眠的次数就越少，反之，vacuum_cost_limit的值越小，VACUUM 和ANALYZE命令在执行的过程中，睡眠的次数就越多。
	01. vacuum_cost_delay (integer)任何时候设置。默认0。它决定执行VACUUM 和ANALYZE命令的进程的睡眠时间。单位是微秒。它的值最好是10的整数，如果不是10的整数，系统会自动将它设为比该值大的并且最接近该值的是10的倍数的整数。如果值是0，VACUUM 和ANALYZE命令在执行过程中不会主动进入睡眠状态，会一直执行下去直到结束。
	02. vacuum_cost_page_hit (integer)任何时候设置。默认值是1。
	03. vacuum_cost_page_miss (integer) 任何时候被设置。默认值是10。
	04. vacuum_cost_page_dirty (integer)在任何时候被设置。默认值是20。
	05. vacuum_cost_limit (integer)任何时候被设置。默认值是200。
- 后台写数据库进程 负责将数据缓冲区中的被修改的数据块写回到数据库物理文件中。
	01. bgwriter_delay (integer)只能在conf中设置。后台写数据库进程的睡眠时间, 值是10的倍数，如果用户设定的值不是10的倍数，数据库会自动将参数的值设为比用户指定的值大的最接近用户指定的值的同时是10的倍数的值。单位毫秒，默认200。
	02. bgwriter_lru_maxpages (integer)只能在conf中设置。默认100。写到外部文件中的脏数据块的个数不能超过bgwriter_lru_maxpages指定的值。例如，如果它的值是500，则后台写数据库进程每次写到物理文件的数据页的个数不能超过500，若超过，进程将进入睡眠状态，等下次醒来再执行写物理文件的任务。如果它的值被设为0, 后台写数据库进程将不会写任何物理文件（但还会执行检查点操作）。
	03. bgwriter_lru_multiplier (floating point)只能在conf中设置。默认2.0。每次写物理文件时，写到外部文件中的脏数据块的个数（不能超过bgwriter_lru_maxpages指定的值）。一般使用默认值即可，不需要修改这个参数。这个参数的值越大，后台写数据库进程每次写的脏数据块的个数就越多。
- 事务日志
	01. full_page_writes (boolean)只能在conf中被设置。默认on。开可以提高数据库的可靠性，减少数据丢失的概率，但产生过多的事务日志，降低数据库的性能。
	02. wal_buffers (integer)启动时设置。默认8。事务日志缓冲区块的个数，块大小8KB。事务日志缓冲区位于数据库的共享内存中。
	03. wal_writer_delay (integer)只能在conf中设置。事务日志进程的睡眠时间。单位毫秒，默认200。
	04. commit_delay (integer)任何时候设置。事务在发出提交命令以后的睡眠时间，只有在睡眠了commit_delay指定的时间以后，事务产生的事务日志才会被写到事务日志文件中，事务才能真正地提交。增大这个参数会增加用户的等待时间，但是可以让多个事务被同时提交，提高系统的性能。如果数据库中的负载比较高，而且大部分事务都是更新类型的事务，可以考虑增大这个参数的值。下面的参数commit_siblings会影响commit_delay是否生效。默认0，单位是微秒。
	05. commit_siblings (integer)任何时候设置。参数commit_delay是否生效。假设commit_siblings的值是5，如果一个事务发出一个提交请求，此时，如果数据库中正在执行的事务的个数大于或等于5，那么该事务将睡眠commit_delay指定的时间。如果数据库中正在执行的事务的个数小于5，这个事务将直接提交。默认值是5。
- 检查点
	01. checkpoint_segments (integer)只能在conf中设置。默认3。它影响系统何时启动一个检查点操作。如果上次检查点操作结束以后，系统产生的事务日志文件的个数超过checkpoint_segments的值，系统就会自动启动一个检查点操作。增大这个参数会增加数据库崩溃以后恢复操作需要的时间。
	02. checkpoint_timeout (integer)只在conf中设置。单位秒，默认300。它影响系统何时启动一个检查点操作。如果现在的时间减去上次检查点操作结束的时间超过了checkpoint_timeout的值，系统就会自动启动一个检查点操作。增大这个参数会增加数据库崩溃以后恢复操作需要的时间。
	03. checkpoint_completion_target (floating point)只在conf中设置. 控制检查点操作的执行时间。合法的取值在0到1之间，默认值是0.5。不要轻易地改变这个参数的值，使用默认值即可。
- 归档模式
	01. archive_mode (boolean)启动时设置。默认off。是否打开归档模式。
	02. archive_dir (string)启动时设置。默认空串。它设定存放归档事务日志文件的目录。
	03. archive_timeout (integer)只能在conf中设置。默认0。单位秒。如果archive_timeout的值不是0，而且当前时间减去数据库上次进行事务日志文件切换的时间大于archive_timeout的值，数据库将进行一次事务日志文件切换。一般情况下，数据库只有在一个事务日志文件写满以后，才会切换到下一个事务日志文件，设定这个参数可以让数据库在一个事务日志文件尚未写满的情况下切换到下一个事务日志文件。
- 优化器参数 存取方法参数,控制查询优化器是否使用特定的存取方法。除非对优化器特别了解，一般情况下，使用它们默认值即可。
	01. enable_bitmapscan (boolean) 打开或者关闭bitmap-scan 。默认值是 on。
	02. enable_hashagg (boolean) 打开或者关闭hashed aggregation。默认值是 on。
	03. enable_hashjoin (boolean) 打开或者关闭hash-join。默认值是 on。
	04. enable_indexscan (boolean) 打开或者关闭index-scan。默认值是 on。
	05. enable_mergejoin (boolean) 打开或者关闭merge-join。默认值是 on。
	06. enable_nestloop (boolean) 打开或者关闭nested-loop join。默认on。不可能完全不使用nested-loop join，关闭这个参数会让系统在有其它存取方法可用的情况下，不使用nested-loop join。
	07. enable_seqscan (boolean) 打开或者关闭sequential scan。默认值是 on。不可能完全不使用sequential scan，关闭这个参数会让系统在有其它存取方法可用的情况下，不使用sequential scan。
	08. enable_sort (boolean) 打开或者关闭explicit sort。默认值是 on。不可能完全不使用explicit sort，关闭这个参数会让系统在有其它存取方法可用的情况下，不使用explicit sort。
	09. enable_tidscan (boolean) 打开或者关闭TID scan。默认值是 on。
- 优化器成本常量 优化器用一个正的浮点数来表示不同的查询计划的执行成本，每个基本的数据库操作都会被赋给一个确定的成本常量，优化器根据每个基本操作的执行成本来计算每个查询计划的执行成本。不要轻易地改变下面的参数的值，使用它们的默认值即可。
	01. seq_page_cost (floating point) 设置从数据文件上顺序读取一个数据块的执行成本。默认值是1.0。
	02. random_page_cost (floating point) 设置从数据文件上随机读取一个数据块的执行成本。默认值是4.0。
	03. cpu_tuple_cost (floating point) 设置处理每一个数据行的执行成本。默认值是0.01。
	04. cpu_index_tuple_cost (floating point) 设置在扫描索引的过程中处理每一个索引项的执行成本。默认值是0.005。
	05. cpu_operator_cost (floating point) 设置处理每一个运算符或函数的执行成本。默认值是0.0025。
	06. effective_cache_size (integer) 设置单个查询可以使用的数据缓冲区的大小。默认值是128MB。
- Genetic Query Optimizer 控制优化器使用的遗传算法。除非对遗传算法特别了解，一般情况下，使用它们默认值即可。
	01. geqo (boolean) 打开或者关闭遗传优化器。默认值是on。
	02. geqo_threshold (integer) 确定使用遗传优化器的查询类型。默认值是12。如果FROM子句中引用的的表的数目超过geqo_threshold的值，就会使用遗传优化器。对于简单的查询使用穷举优化器。
	03. geqo_effort (integer) 控制遗传优化器在生成查询计划需要的时间和查询计划的有效性之间做一个折中。有效的取值范围是1到 10。默认值是5。值越大，优化器花在选择查询计划的上的时间越长，同时找到一个最优的查询计划的可能性就越大。系统通常不直接使用geqo_effort的值，而是使用它的值来计算参数geqo_pool_size和geqo_generations的默认。
	04. geqo_pool_size (integer) 控制遗传优化器的池(pool)大小。默认值是0。池大小是遗传群体中的个体数目。至少是2，典型的取值在10和1000之间。如果参数的值是0，系统会自动根据geqo_effort的值和查询中引用的表的个数选择一个默认值。
	05. geqo_generations (integer) 控制遗传优化器的代(generation)的大小。默认值是0。代是遗传算法的迭代次数。至少是1，典型的取值范围与池的取值范围相同。如果参数的值是0，系统会自动根据geqo_pool_size的值和选择一个默认值。
	06. geqo_selection_bias (floating point) 控制遗传优化器的代选择偏差(selection bias)的大小。默认值是2。取值范围在1.50到2.00之间。
- 其它优化器参数
	01. default_statistics_target (integer) 设置默认的收集优化器统计数据的目标值。它的值越大，ANALYZE操作的执行的时间越长，扫描的数据行的个数也就越多，得到的优化器统计数据就越准确。也可以使用命令ALTER TABLE ... ALTER COLUMN ... SET STATISTICS来为表的每个列设置一个单独的统计数据目标值，这个值的作用与参数default_statistics_target是一样，它只影响相关的列的统计数据收集过程。默认值是10。
	02. constraint_exclusion (boolean) 如果该参数的值是on，查询优化器将使用表上的约束条件来优化查询。如果它的值是off，查询优化器不会使用表上的约束条件来优化查询。默认值是off。
- 数据库运行日志配置参数
	01. log_directory (string) 这个参数只能在postgresql.conf文件中被设置。它决定存放数据库运行日志文件的目录。默认值是pg_log。可以是绝对路径，也可是相对路径（相对于数据库文件所在的路径）。
	02. log_filename (string) 它决定数据库运行日志文件的名称。默认值是postgresql-%Y-%m-%d_%H%M%S.log。它的值可以包含%Y、%m、%d、%H、%M和%S这样的字符串，分别表示年、月、日、小时、分和秒。 如果参数的值中没有指定时间信息（没有出现%Y、%m、%d、%H、%M和%S中的任何一个），系统会自动在log_filename值的末尾加上文件创建的时间作为文件名，例如，如果log_filename的值是 server_log，那么在Sun Aug 29 19:02:33 2004 MST被创建的日志文件的名称将是server_log.1093827753，1093827753是Sun Aug 29 19:02:33 2004 MST在数据库内部的表示形式。这个参数只能在postgresql.conf文件中被设置。 
	03. log_rotation_age (integer) 它决定何时创建一个新的数据库日志文件。单位是分钟。默认值是0。如果现在的时间减去上次创建一个数据库运行日志的时间超过了log_rotation_age的值，数据库将自动创建一个新的运行日志文件。如果它的值是0，该参数将不起任何作用。这个参数只能在postgresql.conf文件中被设置。
	04. log_rotation_size (integer) 这个参数只能在postgresql.conf文件中被设置。它决定何时创建一个新的数据库日志文件。单位是KB。默认值是10240。如果一个日志文件写入的数据量超过log_rotation_size的值，数据库将创建一个新的日志文件。如果它的值被设为0，该参数将不起任何作用。
	05. log_truncate_on_rotation (boolean) 系统在创建一个新的数据库运行日志文件时，如果发现存在一个同名的文件，当log_truncate_on_rotation的值是on时，系统覆盖这个同名文件。当log_truncate_on_rotation的值是off时，系统将重用这个同名文件，在它的末尾添加新的日志信息。另外要注意的是，只有在因为参数log_rotation_age起作用系统才创建新的日志文件的情况下，才会覆盖同名的日志文件。因为数据库重新启动或者因为参数log_rotation_size起作用而创建新的日志文件，不会覆盖同名的日志文件，而是在同名的日志文件末尾添加新的日志信息。这个参数只能在postgresql.conf文件中被设置。默认值是off。 例如，将这个参数设为on，将log_rotation_age设为60，将同时将log_filename设为postgresql-%H.log，系统中一共将只有24个日志文件，它们会被不断地重用，任何时刻，系统中最多只有最近24小时的日志信息。
	06. client_min_messages (string) 控制发送给客户端的消息级别。合法的取值是DEBUG5、DEBUG4、DEBUG3、DEBUG2、DEBUG1、LOG、NOTICE、WARNING、ERROR、FATAL和PANIC，每个级别都包含排在它后面的所有级别中的信息。级别越低，发送给客户端的消息就越少。 默认值是NOTICE。这个参数可以在任何时候被设置。
	07. log_min_messages (string) 控制写到数据库日志文件中的消息的级别。合法的取值是DEBUG5、DEBUG4、DEBUG3、DEBUG2、DEBUG1、INFO、NOTICE、WARNING、ERROR、 LOG、FATAL和PANIC，每个级别都包含排在它后面的所有级别中的信息。级别越低，数据库运行日志中记录的消息就越少。默认值是NOTICE。只有超级用户才能修改这个参数。只有超级用户才能设置这个参数。
	08. log_error_verbosity (string) 控制每条日志信息的详细程度。合法的取值是TERSE、DEFAULT和VERBOSE（每个取值都比它前面的取值提供更详细的信息）。只有超级用户才能修改这个参数。默认值是DEFAULT。
	09. log_min_error_statement (string) 是否记录导致数据库出现错误的SQL语句。合法的取值是DEBUG5、DEBUG4、DEBUG3、DEBUG2、DEBUG1、INFO、NOTICE、WARNING、ERROR、 LOG、FATAL和PANIC，每个级别都包含排在它后面的所有级别。默认值是ERROR。只有超级用户才能修改这个参数。
	10. log_checkpoints (boolean) 是否及记录检查点操作信息。默认off。只能在conf文件中设置。必须重启数据库才能生效。
	11. log_connections (boolean) 是否及记录客户端连接请求信息。默认off。只能在conf中设置。必须重启数据库才能生效。
	12. log_disconnections (boolean) 是否记录客户端结束连接信息。默认off。只能在conf文件中被设置。
	13. log_duration (boolean)是否记录每个完成的SQL语句的执行时间。只有超级用户才能修改这个参数。默认off。对于使用扩展协议与数据库通信的客户端，会记载Parse、Bind和Execute的执行时间。
	14. log_hostname (boolean)是否及记录客户端的主机名。默认值是off。如果设为on，可能会影响数据库的性能，因为解析主机名可能需要一定的时间。这个参数只能在postgresql.conf文件中被设置。只能在conf中设置。
	15. log_line_prefix (string)每条日志信息的前缀格式。默认值是空串。它的格式类似c语言中printf函数的format字符串。这个参数只能在postgresql.conf文件中被设置。%u 用户名、%d 数据库名、%r 客户端机器名或IP地址,还有客户端端口、%h 客户端机器名或IP地址、%p 进程ID、%t 带微秒的时间、%m 不带微秒的时间、%i 命令标签: 会话当前执行的命令类型、%c 会话ID、%l每个会话的日志编号，从1开始、%s 进程启动时间、%v 虚拟事务ID (backendID/localXID)、%x 事务ID (0表示没有分配事务ID)、%q 不产生任何输出。如果当前进程是backend进程，忽略这个转义序列，继续处理后面的转义序列。如果当前进程不是backend进程，忽略这个转义序列和它后面的所有转义序列。、%% 字符%
	16. log_lock_waits (boolean) 如果一个会话等待某个类型的锁的时间超过deadlock_timeout的值，该参数决定是否在数据库日志中记录这个信息。默认值是off。只有超级用户才能修改这个参数。
	17. log_statement (string) 控制记录哪种SQL语句的执行信息。有效的取值是none、ddl、mod和all。默认值是none。ddl包括所有数据定义语句，如CREATE、ALTER和DROP语句。mod包括所有ddl语句和更新数据的语句，例如INSERT、UPDATE、DELETE、TRUNCATE、 COPY FROM、PREPARE和 EXECUTE。All包括所有的语句。只有超级用户才能修改这个参数。
	18. log_temp_files (integer) 控制是否记录临时文件的删除信息。单位是KB。0表示记录所有临时文件的删除信息。正整数表示只记录大小比log_temp_files的值大的临时文件的删除信息。-1表示不记录任何临时文件删除信息。默认值是-1。这个参数可以在任何时候被设置。
	19. log_timezone (string) 设置数据库日志文件在写日志文件时使用的时区。默认值是unknown，意识是使用操作系统的时区。这个参数只能在postgresql.conf文件中被设置。
- 数据库运行统计数据相关参数 下面的参数控制是否搜集特定的数据库运行统计数据：
	01. track_activities (boolean) 是否收集每个会话的当前正在执行的命令的统计数据。默认on。只有超级用户才能修改这个参数。
	02. track_counts (boolean) 是否收集数据库活动的统计数据。默认值是on。只有超级用户才能修改这个参数。
	03. log_statement_stats (boolean) log_parser_stats (boolean) log_planner_stats (boolean) log_executor_stats (boolean) 这些参数决定是否在数据库的运行日志里记载每个SQL语句执行的统计数据。如果log_statement_stats的值是on，其它的三个参数的值必须是off。所有的这些参数的默认值都是off。log_statement_stats报告整个语句的统计数据，log_parser_stats记载数据库解析器的统计数据，log_planner_stats报告数据库查询优化器的统计数据，log_executor_stats报告数据库执行器的统计数据。只有超级用户才能修改这些参数。
- 自动垃圾收集相关参数 下面的参数控制自动垃圾收集的行为：
	01. autovacuum (boolean) 控制是够打开数据库的自动垃圾收集功能。默认值是on。如果autovacuum被设为on，参数track_counts（参考本章10.9）也要被设为on，自动垃圾收集才能正常工作。注意，即使这个参数被设为off，如果事务ID回绕即将发生，数据库会自动启动一个垃圾收集操作。这个参数只能在文件postgresql.conf中被设置。
	02. log_autovacuum_min_duration (integer) 单位毫秒。如果它的值为0，所有的垃圾搜集操作都会被记录在数据库运行日志中，如果它的值是-1，所有的垃圾收集操作都不会被记录在数据库运行日志中。如果把它的值设为250毫秒，只要自动垃圾搜集发出的VACUUM和ANALYZE命令的执行时间超过250毫秒，VACUUM和ANALYZE命令的相关信息就会被记录在数据库运行日志中。默认值是-1。这个参数只能在 postgresql.conf中被设置。
	03. autovacuum_max_workers (integer) 设置能同时运行的最大的自动垃圾收集工作进程的数目。默认值是3。这个参数只能在文件postgresql.conf中被设置。
	04. autovacuum_naptime (integer) 设置自动垃圾收集控制进程的睡眠时间。单位是秒，默认值是60。这个参数只能在文件postgresql.conf中被设置。
	05. autovacuum_vacuum_threshold (integer) 设置触发垃圾收集操作的阈值。默认值是50。这个参数只能在文件postgresql.conf中被设置。只有一个表上被删除或更新的记录的数目超过了autovacuum_vacuum_threshold的值，才会对这个表执行垃圾收集操作。
	06. autovacuum_analyze_threshold (integer) 设置触发ANALYZE操作的阈值。默认值是50。这个参数只能在文件postgresql.conf中被设置。只有一个表上被删除、插入或更新的记录的数目超过了autovacuum_analyze_threshold的值，才会对这个表执行ANALYZE操作。
	07. autovacuum_vacuum_scale_factor (floating point) 这个参数与何时对一个表进行垃圾收集操作相关。默认值是0.2。这个参数只能在文件postgresql.conf中被设置。
	08. autovacuum_analyze_scale_factor (floating point)这个参数与何时对一个表进行ANALYZE操作相关。默认值是0.1。这个参数只能在文件postgresql.conf中被设置。
- 锁管理参数
	01. deadlock_timeout(integer) 设置死锁超时检测时间。单位是微秒，默认值是1000。死锁检测是一个消耗许多 CPU资源的操作。这个参数的值不能太小。在数据库负载比较大的情况下，应当增大这个参数的值。
	02. max_locks_per_transaction(integer) 这个参数控制每个事务能够得到的平均的对象锁的个数。默认值是64。数据库在启动以后创建的共享锁表的最大可以保存max_locks_per_transaction * (max_connections + max_prepared_transactions)个对象锁。单个事务可以同时获得的对象锁的数目可以超过max_locks_per_transaction的值，只要共享锁表中还有剩余空间。
- 系统预设参数 这些参数系统启动后会自动设置，用户只能查询它们的值，不能用任何方式修改这些参数。
	01. block_size (integer) 报告数据库文件数据块的大小。
	02. integer_datetimes (boolean) on系统内部用64位整数存储时间/日期类型的数据。如果值是off，表示系统内部不是用64位整数存储时间/日期类型的数据。
	03. lc_collate (string) 报告字符排序设置。
	04. lc_ctype (string) 报告字符分类设置。
	05. max_function_args (integer) 报告函数参数的最大个数。
	06. max_identifier_length (integer) 报告标识符的最大长度。
	07. max_index_keys (integer) 报告索引的关键字中列的最大数目。
	08. server_encoding (string) 报告服务器的字符编码名称。
	09. server_version (string) 报告服务器的版本号。
	10. server_version_num (integer) 报告服务器的小版本号。
	11. data_directory (string) 数据文件所在的目录。
	12. config_file (string) 数据库参数配置文件名。
	13. hba_file (string) 数据库hba文件名。
	14. external_pid_file (string) 数据库PID文件名。
- 其它参数
	01. default_with_oids (boolean) 该参数控制CREATE TABLE 和CREATE TABLE AS命令是否给新建的表添加一个系统列OID。如果它的值是on，CREATE TABLE 和CREATE TABLE AS命令在没有指定WITH OIDS 和WITHOUT OIDS子句的情况下，新建的表将包含系统列OID，同时SELECT INTO命令也会将系统列OID添加到新建的表中去。默认值是off。
	02. search_path (string) 设置模式搜索路径。它的值由一个或多个模式名构成，不同的模式名用逗号隔开。默认值是'"$user", public'，$user表示与当前会话用户名同名的模式名，如果这样的模式不存在，$user将被忽略。系统表所在的模式pg_catalog，如果出现在search_path中，它会按指定的顺序被搜索。如果它没有出现在search_path中，则总是排在search_path中指定的所有模式前面被搜索。
当前会话如果存在存放临时表的模式，可以使用别名pg_tmp将它列在搜索路径中，例如，'"$user", public, pg_tmp'。如果存放临时表的模式没有出现在搜索路径中，则它会作为第一个搜索对象，排在pg_catalog和search_path中所有模式的前面。注意，系统只会在存放临时表的模式中搜索表、视图、序列对象和数据类型这样的数据库对象，不会在里面搜索函数或运算符这样的数据库对象。
	03. default_tablespace (string) 设置默认表空间。默认值是空串。
	04. temp_tablespaces (string) 设置默认的存放临时对象的表空间。默认值是空串。它的值由一个或多个表空间组成，不同的值用逗号隔开。
	05. check_function_bodies (boolean) 设置是否验证CREATE FUNCTION.命令中指定的函数体。默认值是on。关闭这个选项可以在从dump文件中恢复函数定义信息时避免系统报错，因为一个正在恢复的函数可能会引用尚未恢复的函数。
	06. default_transaction_isolation (string) 默认的事务隔离级别。合法的取值是"read uncommitted"、"read committed"、"repeatable read”和"serializable"。 默认值是read committed。
	07. default_transaction_read_only (boolean) 设置每个新创建的事务是否是只读的。默认值是off。只读事务只能修改临时表。详细信息参考SQL命令SET TRANSACTION。
	08. session_replication_role (string) 控制与复制有关的触发器和规则的行为。可能的取值是origin、replica和local，默认值是origin。
	09. statement_timeout(integer) 设置语句执行的超时值。如果一个语句执行的时间超过statement_timeout指定的值，该语句就会被强行终止。如果它的值为0，则关闭超时特性。单位是毫秒，默认值是0。最好不要在文件postgresql.conf中设置这个参数，因为它影响所有的会话。
	10. xmlbinary (string) 设置XML文档中二进制数据的编码类型。它的值可以是base64和hex。默认值是base64。、
	11. xmloption (string) 设置在字符串和XML数据之间进行类型转换时使用的XML文档类型。它的值是DOCUMENT 或者CONTENT，默认值是CONTENT。
	12. DateStyle (string) 设置时间和日期值的显示格式。initdb会将这个参数初始化成与lc_time一致的值。
	13. timezone (string) 设置显示和解释时间类型数值时使用的时区。
	14. timezone_abbreviations (string) 设置数据库接受的时区缩写值。默认值是Default，代表一些通用的时区缩写。
	15. extra_float_digits(integer) 控制浮点数显示时的有效的数值位的最大个数。float4类型的数的显示精度是6+ extra_float_digits, float8类型的数的显示精度是15+ extra_float_digits。有效的取值范围是-15到2。默认值是0。
	16. client_encoding (string) 设置客户端的字符编码类型。默认值是数据库的字符编码类型。
	17. lc_monetary (string) 设置货币值的显示格式。它影响to_char之类的函数的输出。
	18. lc_numeric (string) 设置数值的显示格式。它影响to_char之类的函数的输出。
	19. default_text_search_config (string) 设定全文检索的配置信息，默认值是pg_catalog.simple。
	20. explain_pretty_print (boolean) 设置EXPLAIN VERBOSE命令的是否打开idented输出格式。默认值是on。
	21. dynamic_library_path (string) 设置数据查找动态加载的共享库文件的路径。如果CREATE FUNCTION或LOAD命令没有指定库文件所在的路径，系统将在dynamic_library_path指定的目录中查找指定的库文件。它的值由一个或多个操作系统目录路径组成，不同的路径用冒号隔开。例如，dynamic_library_path = '/usr/local/lib/postgresql:/home/my_project/lib:$libdir'，其中的:$libdir表示PostgreSQL自己内置的共享库的路径，使用命令pg_config –pkglibdir可以找到这个路径。默认值是'$libdir'。如果它的值是空串，自动路径搜索将被关闭。超级用户可以在任何时候修改这个参数，推荐在文件postgresql.conf中配置这个参数。
	22. gin_fuzzy_search_limit (integer) 设置GIN索引返回的集合的大小的上限。
	23. backslash_quote 有三种取值，分别是on、off和safe_encoding。on表示可以在字符串中用\'表示但引号。off表示\只是一个普通字符，单个单引号必须用两个连续的单引号表示。safe_encoding与on的功能基本相同，但它比on更安全，推荐使用safe_encoding。默认值是safe_encoding。这个参数可以在任何时候被设置。
	24. regex_flavor(string) 设置系统接受的正则表达式的类型。取值可以是advanced、extended和basic。默认值是advanced。这个参数可以在任何时候被设置。
	25. standard_conforming_strings (boolean) 如果是on，系统会将sql命令中的字符串中的字符“\”作为普通的字符看待，而不是作为转义字符看待。如果是off，系统会将sql命令中的字符“\”作为转义字符看待。默认值是off。这个参数可以在任何时候被设置。
	26. synchronize_seqscans (boolean) 如果这个参数的值是是on，多个查询如果同时顺序扫描一个表中的同一个数据页，系统只会发出一个IO操作来读取这个数据页，读出来的数据被多个查询共享，减少不必要的 IO操作，提高系统执行的效率，但是不带 ORDER BY子句的查询产生的结果中数据行的顺序可能会发生改变。如果值为off，这个特性将被关闭。默认值是on。
	27. wal_level = minimal, replica, or logical

# pg_hba.conf 客户端认证配置
01. 第一列 local 本地连接认证、host 远程连接 tcp/ip连接 hostssl
02. 第二列 数据库 all 所有数据库，当其它规则没有匹配则匹配all
03. 第三列 数据库用户 all 同02
04. 第四列 地址  0.0.0.0/0 ::/0 表示所有地址
05. 第五列 认证方式
    01. ident local使用同名用户登录无需验证，如果没有同名用户则失败
    02. md5 md5密码
    03. password 明文密码
    04. trust 知道数据库用户名就可以登录
    05. reject 拒绝认证 

# HA
```sh
yum update -y

#postgresql
yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
yum -y install postgresql12-server postgresql12-contrib
/usr/pgsql-12/bin/postgresql-12-setup initdb
systemctl enable postgresql-12
systemctl start postgresql-12

#主
su - postgres
psql -c "ALTER SYSTEM SET listen_addresses TO '*';"
psql -c "create role HA_USER login replication encrypted password 'HA_USER_PASSWORD';"
exit

cat /usr/lib/systemd/system/postgresql-12.service
vi /var/lib/pgsql/12/data/postgresql.conf
	wal_level = hot_standby
    hot_standby = on

vi /var/lib/pgsql/12/data/pg_hba.conf
host     replication     HA_USER         slave_ip/32             md5
host     all             all             0.0.0.0/0               md5

systemctl restart postgresql-12

#从
systemctl stop postgresql-12

su - postgres
rm -rf /var/lib/pgsql/12/data/*
pg_basebackup -D /var/lib/pgsql/12/data -Fp -Xs -v -P -R -h master_ip -p 5432 -U HA_USER

systemctl start postgresql-12

#pgpool-II
ssh-keygen
ssh-copy-id -i .ssh/id_rsa.pub master_ip
ssh-copy-id -i .ssh/id_rsa.pub slave_ip

curl -O https://www.pgpool.net/yum/rpms/3.7/redhat/rhel-7-x86_64/pgpool-II-release-3.7-2.noarch.rpm
rpm -ivh pgpool-II-release-3.7-2.noarch.rpm
rm -rf pgpool-II-release-3.7-2.noarch.rpm

yum -y install pgpool-II-pg12 pgpool-II-pg12-extensions iproute 

chmod u+x /usr/sbin/ip
chmod u+s /usr/sbin/arping
chmod u+s /sbin/ip

chown postgres.postgres /var/run/pgpool
mkdir –p /var/log/pgpool/
touch /var/log/pgpool/pgpool_status
chown -R postgres.postgres /var/log/pgpool/


vi /etc/pgpool-II/pool_hba.conf
host    all         all        	0.0.0.0/0             md5
host    all         all         ::1/128               


pg_md5 postgres   #e8a48653851e28c69d0506508fb27fc5

vi /etc/pgpool-II/pcp.conf
postgres:e8a48653851e28c69d0506508fb27fc5

pg_md5 -p -m -u db_use_usr_name pool_passwd

vi /var/lib/pgsql/12/failover_stream.sh
		#! /bin/sh
		/usr/bin/ssh -T $1 "/usr/pgsql-12/bin/pg_ctl promote -D /var/lib/pgsql/12/data"
		exit 0;
chmod 777 /var/lib/pgsql/12/failover_stream.sh
chown postgres.postgres /var/lib/pgsql/12/failover_stream.sh


vi /etc/pgpool-II/pgpool.conf
		listen_addresses = '*'

		backend_hostname0 = 'master_ip'
		backend_port0 = 5432				#HA postgresql db server port
		backend_weight0 = 5
		backend_data_directory0 = '/var/lib/pgsql/12/data'
		backend_flag0 = 'ALLOW_TO_FAILOVER'

		backend_hostname1 = 'slave1_ip'
		backend_port1 = 5432
		backend_weight1 = 1
		backend_data_directory1 = '/var/lib/pgsql/12/data'
		backend_flag1 = 'ALLOW_TO_FAILOVER'

		....

		enable_pool_hba = on
		load_balance_mode = on
		master_slave_mode = on

		sr_check_period = 5
		sr_check_user = 'HA_USER'
		sr_check_password = 'HA_USER_PASSWORD'

		health_check_period = 10
		health_check_user = 'postgres'
		health_check_password = 'postgres'
		health_check_database = 'postgres'

		failover_command = '/var/lib/pgsql/12/failover_stream.sh %H'

		use_watchdog = on
		wd_hostname = 'master_ip'

		delegate_IP = 'nginx_ip or vs ip'
		if_up_cmd = 'ip addr add $_IP_$/24 dev eth0 label eth0:0'
		if_down_cmd = 'ip addr del $_IP_$/24 dev eth0:0'
		arping_cmd = 'arping -U $_IP_$ -w 1 -I eth0'

		heartbeat_destination0 = 'slave1_ip'
		heartbeat_destination_port0 = 9694		#other pgpool wd_heartbeat_port=?
		heartbeat_device0 = 'eth0'

		....

		other_pgpool_hostname0 = 'slave1_ip'
		other_pgpool_port0 = 9999				#other pgpool port=?
		other_wd_port0 = 9000					#other pgpool wd_port=?


		pcp_port = 9898		#client connect to this port for sql

#启动pgpool-II
su - postgres
pgpool -C –D

#主库下线重新上线
mv /var/lib/pgsql/12/data/recovery.done /var/lib/pgsql/12/data/recovery.conf
vi /var/lib/pgsql/12/data/postgresql.conf
	max_connections > 参数的值是大于主库的
systemctl restart postgresql-12
pcp_attach_node -d -U postgres -h vip_ip -p 9898 -n 0
```