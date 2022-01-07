---
layout:         post
title:          ios相关开源
subtitle:       ios相关开源
date:           2015-12-04 09:08:00
author:         nomadli
header-img:     ../img/bg-coffee.jpeg
catalog:        true
tags:
        - IOS
---

* content
{:toc}

## apple open source
- https://developer.apple.com/opensource/
- https://opensource.apple.com/
- https://github.com/macports
- https://github.com/macosforge
- https://github.com/MacRuby
- git://git.webkit.org/WebKit-https.git https://git.webkit.org/git/WebKit-https.git
- https://github.com/apple
- https://github.com/scaponapple
- https://github.com/dcerpc
- https://github.com/smartcardservices
- https://github.com/XQuartz
- https://github.com/researchkit
- https://github.com/carekit-apple

## React JavaScript MVC 框架
- Facebook开发的前后端通吃的Web App JavaScript解决方案.  [![github][1]](https://github.com/facebook/react)  
- React Native 用写Web App的方式去写Native App.  [![github][1]](https://github.com/facebook/react-native) 

## C函数运行时
- c函数调用 [![github][1]](https://github.com/libffi/libffi) 

## sqlite3
01. [代码版本管理工具](https://www.fossil-scm.org/home/uv/fossil-macosx-2.13-preview-20201009.tar.gz)
02. 克隆原始 fossil clone https://www.sqlite.org/src sqlite.fossil
03. 修改密码fossil sql -R sqlite.fossil   UPDATE user SET pw='xxx' WHERE login='nomadli';
04. 退出sql .exit
05. 将密码做SHA fossil test-hash-passwords sqlite.fossil
06. cheout 原始代码 ../fossil open ../sqlite.fossil
07. 将github 缺失文件复制到代码目录 manifest.uuid manifest
08. 编辑manifest文件添加自定义源文件   F src/encry.c F src/aes256cbc.c ...
09. 新建代码合并目录，在合并目录执行 sqlite_path/configure --enable-tempstore ...
10. 在合并目录执行 make sqlite3.c
11. 找到最终文件 sqlite3.c sqlite3.h config.h 添加编译宏 _HAVE_SQLITE_CONFIG_H等
12. src目录的manager.c manager.h 简单封装sqlite3更方便些
13. 在sqlite3.c和sqlite3.h 添加编译选择参数
14. 开启网站服务 fossil ui sqlite.fossil
```C
////1串行模式(多个线程可以使用一个句柄) 0单线程模 2多线程模式(每个线程一个句柄)
#define THREADSAFE                      (1)
#define SQLITE_HAS_CODEC                (1) //启用sqlite加密回调
#define SQLITE_SECURE_DELETE            (1) //删除数据时使用0填充
#define SQLITE_SOUNDEX                  (1) //启用soundex函数可以返回字符串的读音
#define SQLITE_ENABLE_COLUMN_METADATA   (1) //启用列信息相关函数,如列名称
#define SQLITE_OMIT_WAL                 (1) //启用WAL模式
//非日志临时文件 0磁盘忽略pragma指令、 1磁盘pragma可覆盖、2内存pragma可覆盖、3内存忽略pragma指令
#define SQLITE_TEMP_STORE               (2)
```
11. 将文件添加到需要数据库的工程
12. sqlite 文件头的原大小是100修改为160存储密码数据
13. ENCRY_KEY_POST = 72 密钥数据开始位置
14. 密钥长度 16
15. ENCRY_RESERVE = 64 预留字节数,密算法需要的额外空间用于对齐或其它,如果有加密后会按倍变长=失败
16. DB_PAGE_SIZE = 1024 数据库页面尺寸 最小512 最大 65536
17. src PRAGMA journal_mode 获取是否是 wal 模式 内存数据库不需要
   1. src 是wal模式则获取"数据库名-wal"文件的尺寸
   2. src wal 文件有内容 则 PRAGMA page_size 获取页面大小
   3. src 计算wal是否超过了一定的页面 如果2000
   4. src 如果wal超过了一定页面 PRAGMA cache_size=xxxx页面数 根据设备内存设置页面缓存，小心内存溢出
18. src PRAGMA busy_timeout=10000 设置超时时间
19. src PRAGMA temp_store=MEMORY 设置临时存储使用内存
20. des PRAGMA synchronous=OFF 如果备份目标是空 设置关闭同步加快速度
21. des PRAGMA journal_mode=TRUNCATE 设置日志模式使数据直接写入备份数据库
22. des PRAGMA temp_store=MEMORY
23. des PRAGMA cache_size=1 备份数据库不会有其他访问，cache使用后就可以丢弃因此设置为缓存1页
24. sqlite3_backupInit sqlite3_backuppagecount sqlite3_backup_remaining sqlite3_backupstep sqlite3_backupfinish
25. des PRAGMA journal_mode= 设置目标数据库与源码数据库相同 WAL｜DELETE｜TRUNCATE｜PERSIST｜MEMORY
26. sqlite3 编译参数
001. _HAVE_SQLITE_CONFIG_H \#include "config.h" 包含其他配置选项，如由autoconf脚本生成的HAVE_ INTERFACE类型选项。
002. HAVE_FDATASYNC = true，VFS使用fdatasync()而不是fsync()。=false始终使用fsync()。
003. HAVE_GMTIME_R = true && SQLITE_OMIT_DATETIME_FUNCS = true，则CURRENT_TIME，CURRENT_DATE和CURRENT_TIMESTAMP关键字将使用线程安全的gmtime_r()接口而不是gmtime()。在通常情况下，SQLITE_OMIT_DATETIME_FUNCS未定义或为false，则内置的日期和时间函数用于实现CURRENT_TIME，CURRENT_DATE和CURRENT_TIMESTAMP关键字，并且既不会调用gmtime_r()也不会调用gmtime()。

HAVE_ISNAN

如果HAVE_ISNAN选项为true，则SQLite调用系统库isnan()函数来确定双精度浮点值是否为NaN。如果HAVE_ISNAN未定义或为false，那么SQLite将替代自己的本地实现isnan()。

HAVE_LOCALTIME_R

如果HAVE_LOCALTIME_R选项为true，那么SQLite使用线程安全localtime_r()库例程而不是localtime()来帮助实现本地时间修饰符到内置的日期和时间函数。

HAVE_LOCALTIME_S

如果HAVE_LOCALTIME_S选项为true，那么SQLite使用线程安全localtime_s()库例程而不是localtime()来帮助实现本地时间修饰符到内置日期和时间函数。

HAVE_MALLOC_USABLE_SIZE

如果HAVE_MALLOC_USABLE_SIZE选项为true，则SQLite尝试使用malloc_usable_size()接口来查找从标准库malloc()或realloc()例程获得的内存分配的大小。此选项仅适用于使用标准库malloc()的情况。在Apple系统上，使用“zone malloc”，因此此选项不适用。当然，如果应用程序使用SQLITE_CONFIG_MALLOC提供自己的malloc实现，则此选项不起作用。如果HAVE_MALLOC_USABLE_SIZE选项被省略或者为false，那么SQLite使用一个围绕系统malloc()和realloc()的包装，它将每个分配放大8个字节，并将分配的大小写入最初的8个字节，然后SQLite还实现了自己的本地版本的malloc_usable_size()，它可以查询8字节前缀来查找分配大小。这种方法可行，但并不理想。鼓励应用程序尽可能使用HAVE_MALLOC_USABLE_SIZE。

HAVE_STRCHRNUL

如果HAVE_STRCHRNUL选项为true，那么SQLite使用strchrnul()库函数。如果这个选项丢失或者错误，那么SQLite替代它自己的strchrnul()的自制实现。

HAVE_USLEEP

如果HAVE_USLEEP选项为true，则默认的unix VFS使用usleep()系统调用来实现xSleep方法。如果此选项未定义或为false，则unix上的xSleep使用sleep()实现，这意味着无论参数为多少，sqlite3_sleep()的最小等待间隔为1000毫秒。

HAVE_UTIME

如果HAVE_UTIME选项为true，那么内置但非标准的“unix-dotfile”VFS将使用utime()系统调用而不是utimes()来设置锁定文件上的最后访问时间。

SQLITE_BYTEORDER=(0|1234|4321)

SQLite需要知道目标CPU的本地字节顺序是大端还是小端。对于big-endian计算机，SQLITE_BYTEORDER预处理器设置为4321，对于小端计算机，SQLITE_BYTEORDER预处理器设置为1234，或者可以为0，表示字节顺序必须在运行时确定。代码中有#ifdefs自动为所有常见平台和编译器设置SQLITE_BYTEORDER。但是，在为隐蔽目标编译SQLite时适当地设置SQLITE_BYTEORDER可能是有利的。如果目标字节顺序不能在编译时确定，那么SQLite就会退回到执行运行时检查的状态，虽然性能会有所下降，但运行时检查总能正常工作。

4.选项设置默认参数值
SQLITE_DEFAULT_AUTOMATIC_INDEX=<0 or 1>

此宏为新打开的数据库连接确定PRAGMA automatic_index的初始设置。对于SQLite到3.7.17的所有版本，如果省略此编译时选项，则通常会为新的数据库连接启用自动索引。但是，这在未来的SQLite版本中可能会发生变化。另请参见：SQLITE_OMIT_AUTOMATIC_INDEX

SQLITE_DEFAULT_AUTOVACUUM=<0 or 1 or 2>

该宏确定SQLite是否创建auto_vacuum标志默认设置为OFF（0），FULL（1）或INCREMENTAL（2）的数据库。默认值为0表示数据库是在关闭自动关闭的情况下创建的。在任何情况下，编译时默认值都可能被PRAGMA auto_vacuum命令覆盖。

SQLITE_DEFAULT_CACHE_SIZE=<N>

该宏为每个附加数据库设置页面缓存的默认最大大小。正值表示该限制是N页。如果N为负，则意味着将高速缓存大小限制为-N * 1024个字节。建议的最大缓存大小可以由PRAGMA cache_size命令覆盖。默认值是-2000，这意味着每个缓存最多可以有2048000个字节。

SQLITE_DEFAULT_FILE_FORMAT=<1 or 4>

SQLite在创建新数据库文件时使用的默认模式格式编号由该宏设置。架构格式都非常相似。格式1和格式4之间的区别在于格式4理解降序索引并且对布尔值具有更紧密的编码。从3.3.0（2006-01-10）开始，所有版本的SQLite都可以读写1到4之间的任何模式格式。但是旧版本的SQLite可能无法读取大于1的格式。因此，旧版本的SQLite将会能够读写由较新版本的SQLite创建的数据库文件，对于SQLite版本，通过3.7.9（2011-11-01），默认架构格式被设置为1。从版本3.7.10开始 （2012-01-16），默认架构格式为4.可以使用PRAGMA legacy_file_format命令在运行时设置新数据库的架构格式号。

SQLITE_DEFAULT_FILE_PERMISSIONS=N

unix下新创建的数据库文件的默认数字文件权限。如果未指定，则默认值为0644，这意味着这些文件是全局可读的，但只能由创建者写入。

SQLITE_DEFAULT_FOREIGN_KEYS=<0 or 1>

此宏确定是否为新的数据库连接默认启用或禁用外键约束的强制实施。每个数据库连接总是可以使用foreign_keys编译指示来强制启用和禁用外键约束和运行时。外键约束的强制默认情况下通常是关闭的，但如果此编译时参数设置为1，则外键约束的强制将默认打开。

SQLITE_DEFAULT_MMAP_SIZE=N

此宏为每个打开的数据库文件设置将用于内存映射I / O的内存量的默认限制。如果N为零，那么内存映射I / O默认是禁用的。此编译时限制和SQLITE_MAX_MMAP_SIZE可以在启动时使用sqlite3_config（SQLITE_CONFIG_MMAP_SIZE）调用或在运行时使用mmap_size编译指示来修改。

SQLITE_DEFAULT_JOURNAL_SIZE_LIMIT=<bytes>

此选项设置永久日记模式和独占锁定模式下回滚日记文件的大小限制以及WAL模式下预写日志文件的大小。如果省略此编译时选项，则回滚日志或预写日志的大小没有上限。日志文件大小限制可以在运行时使用journal_size_limit编译指示来更改。

SQLITE_DEFAULT_LOCKING_MODE=<1 or 0>

如果设置为1，则默认locking_mode设置为EXCLUSIVE。如果省略或设置为0，则默认locking_mode为NORMAL。

SQLITE_DEFAULT_LOOKASIDE=SZ,N

将后备存储器分配器内存池的缺省大小设置为每个SZ个字节的N个条目。可以在启动时使用sqlite3_config（SQLITE_CONFIG_LOOKASIDE）和/或使用sqlite3_db_config（db，SQLITE_DBCONFIG_LOOKASIDE）打开每个数据库连接来修改此设置。

SQLITE_DEFAULT_MEMSTATUS=<1 or 0>

此宏用于确定使用sqlite3_config()的SQLITE_CONFIG_MEMSTATUS参数启用和禁用的功能是否默认可用。默认值是1（启用了SQLITE_CONFIG_MEMSTATUS相关功能）。当内存使用情况跟踪被禁用时，sqlite3_memory_used()和sqlite3_memory_highwater()接口，sqlite3_status64（SQLITE_STATUS_MEMORY_USED）接口和SQLITE_MAX_MEMORY编译时选项都不起作用。

SQLITE_DEFAULT_PCACHE_INITSZ=N

此宏决定了当不使用SQLITE_CONFIG_PAGECACHE配置选项时页缓存模块最初分配的页数，而是从sqlite3_malloc()获取页缓存的内存。此宏设置的页面数量分配在一个分配中，这减少了内存分配器上的负载。

SQLITE_DEFAULT_PAGE_SIZE=<bytes>

该宏用于设置创建数据库时使用的默认页面大小。指定的值必须是2的幂。默认值为4096. PRAGMA page_size命令可以在运行时重写编译时默认值。

SQLITE_DEFAULT_SYNCHRONOUS=<0-3>

该宏决定了PRAGMA同步设置的默认值。如果在编译时未覆盖，则默认设置为2（FULL）。

SQLITE_DEFAULT_WAL_SYNCHRONOUS=<0-3>

此宏确定以WAL模式打开的数据库文件的PRAGMA同步设置的默认值。如果在编译时未覆盖，则此值与SQLITE_DEFAULT_SYNCHRONOUS相同。如果SQLITE_DEFAULT_WAL_SYNCHRONOUS与SQLITE_DEFAULT_SYNCHRONOUS不同，并且应用程序未使用PRAGMA同步语句修改数据库文件的同步设置，则当数据库连接首次切换到WAL模式时，同步设置更改为由SQLITE_DEFAULT_WAL_SYNCHRONOUS定义的值。如果SQLITE_DEFAULT_WAL_SYNCHRONOUS值在编译时未被覆盖，那么它总是与SQLITE_DEFAULT_SYNCHRONOUS相同，因此不会发生自动同步设置更改。

SQLITE_DEFAULT_WAL_AUTOCHECKPOINT=<pages>

此宏设置WAL自动检查点功能的默认页数。如果未指定，则默认页数为1000。

SQLITE_DEFAULT_WORKER_THREADS=N

该宏设置SQLITE_LIMIT_WORKER_THREADS参数的默认值。SQLITE_LIMIT_WORKER_THREADS参数设置单个预准备语句将启动的辅助线程的最大数量，以帮助查询。如果未指定，则默认最大值为0.此处设置的值不能超过SQLITE_MAX_WORKER_THREADS。

SQLITE_EXTRA_DURABLE

用于使默认PRAGMA同步设置为EXTRA而不是FULL的SQLITE_EXTRA_DURABLE编译时选项。此选项不再受支持。改为使用SQLITE_DEFAULT_SYNCHRONOUS = 3。

SQLITE_FTS3_MAX_EXPR_DEPTH=N

此宏设置与FTS3或FTS4全文索引中MATCH运算符右侧对应的搜索树的最大深度。全文搜索使用递归算法，因此树的深度是有限的，以防止使用太多的堆栈空间。默认限制为12.此限制足以满足MATCH运算符右侧最多4095个搜索条件，并且它将堆栈空间使用量保持为小于2000字节。对于普通的FTS3 / FTS4查询，搜索树深度近似为MATCH运算符右侧的术语数的基数2对数。但是，对于短语查询和NEAR查询，搜索树深度在右侧项的数量上是线性的。因此默认深度限制为12就足以支持MATCH上最多4095个普通术语，对于11或12个短语或近似术语来说，它是足够的。即便如此，对于大多数应用来说，默认值已经足够了。

SQLITE_LIKE_DOESNT_MATCH_BLOBS

如果任一操作数是BLOB，则此编译时选项会使LIKE运算符始终返回False。LIKE的默认行为是在比较完成之前将BLOB操作数强制转换为TEXT。此编译时选项使SQLite在处理使用LIKE运算符的查询时运行更有效，代价是破坏向后兼容性。但是，向后兼容性中断可能只是一个技术问题。LIKE处理逻辑中存在一个长期存在的bug（请参阅https://www.sqlite.org/src/info/05f43be8fdda9f）导致它对BLOB操作数的不当行为，并且没有人在近10年的活跃使用中观察到该错误。所以对于更多的用户来说，启用这个编译时选项可能是安全的，从而可以节省LIKE查询的CPU时间。此编译时选项仅影响SQL LIKE运算符，并且不会影响sqlite3_strlike()C语言接口。

SQLITE_MAX_MEMORY=N

此选项将SQLite将从malloc()请求的内存总量限制为N个字节。SQLite试图分配新的内存，这会导致SQLite所有分配的总和超过N bytes will result in an out-of-memory error. This is a hard upper limit. See also the sqlite3_soft_heap_limit() interface. This limit is only functional if memory usage statistics are available via the sqlite3_memory_used() and sqlite3_status64(SQLITE_STATUS_MEMORY_USED) interfaces. Without that memory usage information, SQLite has no way of knowing when it is about to go over the limit, and thus is unable to prevent the excess memory allocation. Memory usage tracking is turned on by default, but can be disabled at compile-time using the SQLITE_DEFAULT_MEMSTATUS option, or at start-time using sqlite3_config(SQLITE_CONFIG_MEMSTATUS).

SQLITE_MAX_MMAP_SIZE=N

此宏为任何单个数据库用于内存映射I / O的地址空间量设置了硬上限。将此值设置为0会完全禁用内存映射I / O，并会导致与构建内存映射I / O相关的逻辑被忽略。此选项会将默认的内存映射I / O地址空间大小（由SQLITE_DEFAULT_MMAP_SIZE或sqlite3_config（SQLITE_CONFIG_MMAP_SIZE）或运行时内存映射的I / O地址空间大小（由sqlite3_file_control（SQLITE_FCNTL_MMAP_SIZE）或PRAGMA mmap_size设置）更改为只要其他设置小于此处定义的最大值。

SQLITE_MAX_SCHEMA_RETRY=N

只要数据库模式发生变化，准备好的语句就会自动重新表示以适应新的模式。这里有一个竞争条件，如果一个线程不断地改变模式，另一个线程可能在预先准备的语句的重新编译和重新编译上转动，并且从未得到任何真正的工作。此参数通过在重新编译准备语句的尝试次数达到固定次数后强制旋转线程放弃来防止无限循环。默认设置为50，这对于大多数应用程序来说已经足够了。

SQLITE_MAX_WORKER_THREADS=N

设置sqlite3_limit（db，SQLITE_LIMIT_WORKER_THREADS，N）设置的上限，该设置确定单个准备语句将用于辅助CPU密集型计算（主要是排序）的辅助线程的最大数量。另请参阅SQLITE_DEFAULT_WORKER_THREADS选项。

SQLITE_MINIMUM_FILE_DESCRIPTOR=N

unix VFS永远不会使用小于N的文件描述符。N的默认值为3.避免使用低编号的文件描述符可以防止意外的数据库损坏。例如，如果使用文件描述符2打开数据库文件，然后assert()失败并调用write（2，...），这可能会通过用断言错误消息覆盖部分数据库文件而导致数据库损坏。只使用较高值的文件描述符可以避免这个潜在的问题。通过将此编译时选项设置为0，可以禁止防止使用低编号的文件描述符。

SQLITE_POWERSAFE_OVERWRITE=<0 or 1>

此选项更改了有关unix和windows VFSes的基础文件系统的powersafe覆盖的默认假设。将SQLITE_POWERSAFE_OVERWRITE设置为1会导致SQLite认为应用程序级写入不能更改字节范围之外的字节，即使写入发生在断电之前。将SQLITE_POWERSAFE_OVERWRITE设置为0时，SQLite会假定在同一扇区中写入字节的其他字节可能由于掉电而更改或损坏。

SQLITE_REVERSE_UNORDERED_SELECTS

此选项会导致默认情况下启用PRAGMA reverse_unordered_selects设置。启用时，缺少ORDER BY子句的SELECT语句将按相反顺序运行。此选项对于检测何时应用程序（错误地）假定SELECT中没有ORDER BY子句的行的顺序始终相同时非常有用。

SQLITE_SORTER_PMASZ=N

如果通过PRAGMA线程设置启用了多线程处理，那么当要排序的内容量超过由SQLITE_CONFIG_PMASZ启动时选项确定的cache_size和PMA大小的最小值时，排序操作将尝试启动助手线程。此编译时选项设置SQLITE_CONFIG_PMASZ启动时选项的默认值。默认值是250。

SQLITE_STMTJRNL_SPILL=N

SQLITE_STMTJRNL_SPILL编译时选项确定SQLITE_CONFIG_STMTJRNL_SPILL开始时间设置的默认设置。该设置决定了语句日志从内存移动到磁盘的大小阈值。

SQLITE_WIN32_MALLOC

此选项允许使用Windows Heap API函数进行内存分配，而不是使用标准库malloc()和free()例程。

YYSTACKDEPTH=<max_depth>

该宏设置SQLite中SQL解析器使用的LALR（1）堆栈的最大深度。默认值是100.一个典型的应用程序将使用少于约20层的堆栈。其应用程序包含需要超过100个LALR（1）堆栈条目的SQL语句的开发人员应认真考虑重构其SQL，因为它可能远远超出任何人理解的能力。

5. Options To Set Size Limits
有编译时选项会设置SQLite中各种结构大小的上限。编译时选项通常设置一个硬上限，可以在运行时使用sqlite3_limit()接口在单个数据库连接上进行更改。

用于设置上限的编译时选项单独记录。以下是可用设置的列表：

SQLITE_MAX_ATTACHED
SQLITE_MAX_COLUMN
SQLITE_MAX_COMPOUND_SELECT
SQLITE_MAX_EXPR_DEPTH
SQLITE_MAX_FUNCTION_ARG
SQLITE_MAX_LENGTH
SQLITE_MAX_LIKE_PATTERN_LENGTH
SQLITE_MAX_PAGE_COUNT
SQLITE_MAX_SQL_LENGTH
SQLITE_MAX_VARIABLE_NUMBER
6.控制操作特性的选项
SQLITE_4_BYTE_ALIGNED_MALLOC

在大多数系统上，malloc()系统调用返回一个对齐到8字节边界的缓冲区。但是在某些系统上（例如：windows），malloc()返回4字节对齐的指针。此编译时选项必须用于从malloc()返回4字节对齐指针的系统。

SQLITE_CASE_SENSITIVE_LIKE

如果此选项存在，那么内置的LIKE运算符将区分大小写。运行时使用case_sensitive_like编译指示可以实现同样的效果。

SQLITE_DIRECT_OVERFLOW_READ

当该选项存在时，数据库文件的溢出页面中包含的内容在读取事务期间直接从磁盘中读取，绕过页面缓存。在大量读取大BLOB的应用程序中，此选项可能会提高读取性能。

SQLITE_HAVE_ISNAN

如果该选项存在，那么SQLite将使用系统数学库中的isnan()函数。这是HAVE_ISNAN配置选项的别名。

SQLITE_OS_OTHER=<0 or 1>

该选项会导致SQLite省略其用于Unix，Windows和OS / 2的内置操作系统界面。结果库将没有默认的操作系统界面。在使用SQLite之前，应用程序必须使用sqlite3_vfs_register()来注册适当的接口。应用程序还必须为sqlite3_os_init()和sqlite3_os_end()接口提供实现。通常的做法是提供的sqlite3_os_init()调用sqlite3_vfs_register()。SQLite会在初始化时自动调用sqlite3_os_init()。当为具有自定义操作系统的嵌入式平台构建SQLite时，通常使用此选项。

SQLITE_SECURE_DELETE

此编译时选项更改secure_delete附注的默认设置。如果不使用此选项，则secure_delete默认为关闭。当此选项存在时，secure_delete默认为打开。secure_delete设置会导致删除的内容被零覆盖。由于必须发生额外的I / O，所以性能损失很小。另一方面，secure_delete可以防止在数据库文件被删除后未被使用的敏感信息片段留在未使用的部分。有关其他信息，请参阅secure_delete附注中的文档。

SQLITE_THREADSAFE=<0 or 1 or 2>

该选项控制SQLite中是否包含代码，以使其能够在多线程环境中安全地运行。默认值是SQLITE_THREADSAFE = 1，可以安全地在多线程环境中使用。当使用SQLITE_THREADSAFE = 0进行编译时，所有的互斥代码都被忽略，并且在多线程程序中使用SQLite是不安全的。使用SQLITE_THREADSAFE = 2进行编译时，只要没有两个线程同时尝试使用相同的数据库连接（或从该数据库连接派生的任何预准备语句），就可以在多线程程序中使用SQLite。换句话说，SQLITE_THREADSAFE = 1将默认的线程模式设置为Serialized。SQLITE_THREADSAFE = 2将默认线程模式设置为多线程。并且SQLITE_THREADSAFE = 0将线程模式设置为单线程。SQLITE_THREADSAFE的值可以在运行时使用sqlite3_threadsafe()接口确定。当使用SQLITE_THREADSAFE = 1或SQLITE_THREADSAFE = 2编译SQLite时，可以在运行时使用sqlite3_config()接口和其中一个动词来更改线程模式：

SQLITE_CONFIG_SINGLETHREAD
SQLITE_CONFIG_MULTITHREAD
SQLITE_CONFIG_SERIALIZED
sqlite3_open_v2()的SQLITE_OPEN_NOMUTEX和SQLITE_OPEN_FULLMUTEX标志也可用于在运行时调整各个数据库连接的线程模式。

请注意，当SQLite使用SQLITE_THREADSAFE = 0进行编译时，构建SQLite线程安全的代码将被省略。发生这种情况时，不可能在启动时或运行时更改线程模式。

有关在多线程环境中使用SQLite的其他信息，请参阅线程模式文档。

SQLITE_TEMP_STORE=<0 through 3>

该选项控制临时文件是存储在磁盘还是存储器中。此编译时选项的各种设置的含义如下：SQLITE_TEMP_STORE含义0始终使用临时文件1默认情况下使用文件，但允许PRAGMA temp_store命令覆盖2默认情况下使用内存，但允许PRAGMA temp_store命令覆盖3始终使用内存默认设置是1.其他信息可以在tempfiles.html中找到。

SQLITE_TRACE_SIZE_LIMIT=N

如果此宏定义为正整数N，则扩展到sqlite3_trace()输出中参数的字符串和BLOB的长度限制为N个字节。

SQLITE_USE_URI

该选项默认启用URI文件名处理逻辑。

7.启用正常关闭功能的选项
SQLITE_ALLOW_URI_AUTHORITY

如果授权部分不是空的或“本地主机”，则URI文件名通常会引发错误。但是，如果SQLite使用SQLITE_ALLOW_URI_AUTHORITY编译时选项进行编译，那么URI将转换为统一命名约定（UNC）文件名，并以这种方式传递给底层操作系统。SQLite的某些未来版本可能会更改为默认启用此功能。

SQLITE_ALLOW_COVERING_INDEX_SCAN=<0 or 1>

此C预处理宏决定了SQLITE_CONFIG_COVERING_INDEX_SCAN配置设置的默认设置。它默认为1（开），这意味着覆盖索引在可能的情况下用于全表扫描，以减少I / O并提高性能。但是，使用覆盖索引进行全面扫描会导致结果以与传统不同的顺序显示，这可能会导致一些（编码不正确）传统应用程序中断。因此，覆盖索引扫描选项可以在编译时在系统上禁用，以最大限度地减少在传统应用程序中暴露错误的风险。

SQLITE_ENABLE_8_3_NAMES=<1 or 2>

如果定义了这个C预处理器宏，那么将包含额外的代码，允许SQLite在仅支持8 + 3文件名的文件系统上运行。如果此宏的值为1，则默认行为是继续使用长文件名，并且如果使用具有“ 8_3_names=1”查询参数的URI文件名打开数据库连接，则仅使用8 + 3个文件名。如果此宏的值为2，则使用8 + 3文件名将成为默认值，但在使用8_3_names=0查询参数时可能会被禁用。

SQLITE_ENABLE_API_ARMOR

定义后，此C预处理器宏会激活额外的代码，以尝试检测SQLite API的滥用情况，例如将NULL指针传递给所需的参数或在销毁后使用对象。

SQLITE_ENABLE_ATOMIC_WRITE

如果定义了此C预处理器宏，并且数据库文件的sqlite3_io_methods对象的xDeviceCharacteristics方法报告（通过SQLITE_IOCAP_ATOMIC位之一）文件系统支持原子写入，并且事务涉及仅更改数据库的单个页面文件，那么事务只提交数据库单个页面的单个写入请求，并且不创建或写入回滚日志。在支持原子写入的文件系统上，此优化可以显着提高小型更新的速度。但是，很少文件系统支持此功能，并且检查此功能的代码路径会降低缺乏原子写入能力的系统的写入性能，因此默认情况下禁用此功能。

SQLITE_ENABLE_BATCH_ATOMIC_WRITE

这个编译时选项使SQLite能够利用底层文件系统中的批处理原子写入功能。从SQLite版本3.21.0（2017-10-24）开始，这仅在F2FS上受支持。但是，该接口通常使用SQLITE_FCNTL_BEGIN_ATOMIC_WRITE和SQLITE_FCNTL_COMMIT_ATOMIC_WRITE使用sqlite3_file_control()，因此该功能将来可以添加到其他文件系统时间。启用此选项时，SQLite会自动检测基础文件系统是否支持批量原子写入，并且何时它会避免为回写控制编写回滚日志。这可以使交易快两倍，同时减少SSD存储设备的磨损。未来版本的SQLite可能默认启用批量原子写入功能，此时此编译时选项将变得多余。

SQLITE_ENABLE_COLUMN_METADATA

当定义这个C预处理器宏时，SQLite包含一些额外的API，可以方便地访问有关表和查询的元数据。通过此选项启用的API是：

sqlite3_column_database_name()
sqlite3_column_database_name16()
sqlite3_column_table_name()
sqlite3_column_table_name16()
sqlite3_column_origin_name()
sqlite3_column_origin_name16()
SQLITE_ENABLE_DBPAGE_VTAB

该选项启用sqlite_dbpage虚拟表。

SQLITE_ENABLE_DBSTAT_VTAB

该选项启用dbstat虚拟表。

SQLITE_ENABLE_EXPLAIN_COMMENTS

该选项为SQLite添加额外的逻辑，将注释文本插入到EXPLAIN的输出中。这些额外的注释使用额外的内存，从而使准备好的语句更大，速度更慢，所以它们在默认情况下以及在大多数应用程序中被关闭。但是一些应用程序（例如SQLite的命令行外壳程序）会将EXPLAIN输出的清晰度与原始性能相比较，并且这些编译时选项可供他们使用。如果启用了SQLITE_DEBUG，则SQLITE_ENABLE_EXPLAIN_COMMENTS编译时选项也会自动启用。

SQLITE_ENABLE_FTS3

在合并中定义此选项时，全文搜索引擎的版本3会自动添加到版本中。

SQLITE_ENABLE_FTS3_PARENTHESIS

该选项修改FTS3中的查询模式解析器，使其支持运算符AND和NOT（除了通常的OR和NEAR），还允许查询表达式包含嵌套括号。

SQLITE_ENABLE_FTS3_TOKENIZER

该选项启用fts3_tokenizer()接口的双参数版本。假设fts3_tokenizer()的第二个参数是一个指向函数的指针（编码为BLOB），该函数实现了应用程序定义的标记器。如果敌对行动者能够用任意的第二个参数运行fts3_tokenizer()的双参数版本，他们可以使用崩溃或控制进程。由于安全考虑，除非使用此编译时选项，否则从版本3.11.0（2016-02-15）开始禁用双参数fts3_tokenizer()功能。版本3.12.0（2016-03-29）添加了sqlite3_db_config（db，SQLITE_DBCONFIG_ENABLE_FTS3_TOKENIZER，1,0）接口，该接口在运行时为特定的数据库连接激活fts3_tokenizer()的双参数版本。

SQLITE_ENABLE_FTS4

在合并中定义此选项时，全文搜索引擎的版本3和4会自动添加到版本中。

SQLITE_ENABLE_FTS5

在合并中定义此选项时，全文搜索引擎（fts5）的版本5会自动添加到版本中。

SQLITE_ENABLE_ICU

该选项会导致国际组件的Unicode或“ICU”扩展到SQLite被添加到版本。

SQLITE_ENABLE_IOTRACE

当SQLite内核和命令行界面（CLI）都使用此选项编译时，CLI会提供一个名为“.iotrace”的额外命令，该命令提供I / O活动的低级日志。此选项是试验性的，可能会在未来版本中停用。

SQLITE_ENABLE_JSON1

在合并中定义此选项时，JSON SQL函数会自动添加到构建中。

SQLITE_ENABLE_LOCKING_STYLE

此选项为Mac OS X的操作系统界面层启用额外的逻辑。额外的逻辑尝试确定底层文件系统的类型，并为该文件系统类型选择正确工作的替代锁定策略。五种锁定策略可用：

POSIX锁定样式。这是其他（非Mac OS X）Unix的默认锁定风格和样式。使用fcntl()系统调用获取并释放锁。
法新社锁定风格。此锁定样式用于使用AFP（Apple归档协议）协议的网络文件系统。锁通过调用库函数_AFPFSSetLock()来获得。
羊群锁定风格。这用于不支持POSIX锁定样式的文件系统。使用flock()系统调用获取并释放锁。
点文件锁定样式。此锁定样式在文件系统不支持flock或POSIX锁定样式时使用。通过在文件系统中相对于数据库文件（“点文件”）的公知位置创建和输入数据库锁，并通过删除相同文件来放弃数据库锁。
没有锁定风格。如果以上都不支持，则使用此锁定样式。没有使用数据库锁定机制。当使用这个系统时，单个数据库被多个客户访问是不安全的。
此外，还提供了五个额外的VFS实现以及默认设置。通过在调用sqlite3_open_v2()时指定其中一个额外的VFS实现，应用程序可以绕过文件系统检测逻辑并显式选择上述锁定样式之一。五个额外的VFS实现被称为“unix-posix”，“unix-afp”，“unix-flock”，“unix-dotfile”和“unix-none”。

SQLITE_ENABLE_MEMORY_MANAGEMENT

该选项为SQLite增加了额外的逻辑，允许它根据请求释放未使用的内存。必须启用此选项才能使sqlite3_release_memory()接口起作用。如果没有使用这个编译时选项，那么sqlite3_release_memory()接口是一个no-op。

SQLITE_ENABLE_MEMSYS3

此选项包含SQLite中的代码，用于实现替代内存分配器。这种替代内存分配器只有在sqlite3_config()的SQLITE_CONFIG_HEAP选项用于提供大量内存时，才会进行所有内存分配。MEMSYS3内存分配器使用dlmalloc()之后的混合分配算法。可以一次启用SQLITE_ENABLE_MEMSYS3和SQLITE_ENABLE_MEMSYS5中的一个。

SQLITE_ENABLE_MEMSYS5

此选项包含SQLite中的代码，用于实现替代内存分配器。这种替代内存分配器只有在sqlite3_config()的SQLITE_CONFIG_HEAP选项用于提供大量内存时，才会进行所有内存分配。MEMSYS5模块将所有的分配都提高到下一个功率的两倍，并且使用了一种先合适的伙伴分配器算法，该算法可以在受到某些操作限制的情况下提供有力的保证，防止碎片和故障。

SQLITE_ENABLE_NULL_TRIM

此选项启用优化，省略行末尾的NULL列，以节省磁盘空间。SQLite版本3.1.6（2005-03-17）及更早版本无法读取使用此选项生成的数据库。此外，启用此选项生成的数据库很容易触发sqlite3_blob_reopen()接口中的e6e962d6b0f06f46错误。由于这些原因，默认情况下禁用此优化。但是，此优化可能会在未来的SQLite版本中默认启用。

SQLITE_ENABLE_PREUPDATE_HOOK

此选项启用几个新的API，在对rowid表进行任何更改之前提供回调。可以使用回调来记录更改发生之前的行状态。preupdate钩子的作用与更新钩子类似，除了在更改之前调用回调，而不是之后，并且除非使用此编译时选项，否则preupdate钩子接口将被省略。preupdate钩子接口最初被添加来支持会话扩展。

SQLITE_ENABLE_QPSG

此选项会导致查询计划程序稳定性保证（QPSG）默认处于打开状态。通常，QPSG已关闭，并且必须在运行时使用sqlite3_db_config()接口的SQLITE_DBCONFIG_ENABLE_QPSG选项激活。

SQLITE_ENABLE_RBU

启用实现RBU扩展的代码。

SQLITE_ENABLE_RTREE

该选项使SQLite包含对R * Tree索引扩展的支持。

SQLITE_ENABLE_SESSION

该选项启用会话扩展。

SQLITE_ENABLE_STMT_SCANSTATUS

该选项启用sqlite3_stmt_scanstatus()接口。通常从构建中省略sqlite3_stmt_scanstatus()接口，因为即使在不使用该功能的语句上，它的性能损失也很小。

SQLITE_ENABLE_STMTVTAB

该编译时选项启用SQLITE_STMT虚拟表逻辑。

SQLITE_RTREE_INT_ONLY

此编译时选项已弃用且未经测试。

SQLITE_ENABLE_SQLLOG

此选项启用额外的代码（特别是sqlite3_config()的SQLITE_CONFIG_SQLLOG选项），该代码可用于创建应用程序执行的所有SQLite处理的日志。这些日志可用于对应用程序的行为进行离线分析，特别是对性能分析。为了使SQLITE_ENABLE_SQLLOG选项有用，需要一些额外的代码。该“test_sqllog.c”SQLite源代码树中的源代码文件是所需额外代码的工作示例。在unix和windows系统上，开发人员可以将“test_sqllog.c”源代码文件的文本附加到“sqlite3.c”合并的末尾，使用-DSQLITE_ENABLE_SQLLOG选项重新编译应用程序，然后使用环境变量控制日志记录。请参阅“test_sqllog.c”源文件中的标题注释以获取更多详细信息。

SQLITE_ENABLE_STAT2

此选项用于使ANALYZE命令在sqlite_stat2表中收集索引直方图数据。但是从SQLite版本3.7.9（2011-11-01）开始，该功能被SQLITE_ENABLE_STAT3取代。SQLITE_ENABLE_STAT2编译时选项现在是空操作。
SQLITE_ENABLE_STAT3

此选项为ANALYZE命令和查询计划程序添加了额外的逻辑，可以帮助SQLite在某些情况下选择更好的查询计划。ANALYZE命令得到了增强，可以从每个索引的最左边一列收集直方图数据，并将该数据存储在sqlite_stat3表中。查询计划员然后将使用直方图数据来帮助它做出更好的索引选择。但请注意，在查询计划器中使用直方图数据违反了对某些应用程序很重要的查询计划程序稳定性保证。

SQLITE_ENABLE_STAT4

此选项为ANALYZE命令和查询计划程序添加了额外的逻辑，可以帮助SQLite在某些情况下选择更好的查询计划。ANALYZE命令已得到增强，可从每个索引的所有列收集直方图数据，并将该数据存储在sqlite_stat4表中。查询计划员然后将使用直方图数据来帮助它做出更好的索引选择。这种编译时选项的缺点是它违反了查询规划器的稳定性保证，这使得确保批量生产应用程序的一致性能变得更加困难。SQLITE_ENABLE_STAT4是对SQLITE_ENABLE_STAT3的增强。STAT3仅记录每个索引最左列的直方图数据，而STAT4增强记录每个索引所有列的直方图数据。

SQLITE_ENABLE_TREE_EXPLAIN

这个编译时选项不再使用。

SQLITE_ENABLE_UPDATE_DELETE_LIMIT

此选项在UPDATE和DELETE语句上启用可选的ORDER BY和LIMIT子句。如果定义了这个选项，那么在使用'lemon'工具生成parse.c文件时也必须定义它。正因为如此，这个选项只能在从源代码构建库的时候使用，而不能从合并或者为网站上的非Unix平台提供的预先打包的C文件集合中使用。

SQLITE_ENABLE_UNKNOWN_SQL_FUNCTION

当SQLITE_ENABLE_UNKNOWN_SQL_FUNCTION编译时选项被激活时，SQLite将在运行EXPLAIN或EXPLAIN QUERY PLAN时抑制“unknown function”错误。SQLite不会抛出错误，而会插入一个名为“unknown()”的替代非操作函数。用“unknown()”代替未被识别的函数只会出现在EXPLAIN和EXPLAIN QUERY PLAN中，而不是普通的语句。在命令行shell中使用时，SQLITE_ENABLE_UNKNOWN_SQL_FUNCTION功能允许将包含应用程序定义函数的SQL文本粘贴到shell中进行分析和调试，而无需创建和加载实现应用程序定义函数的扩展。

SQLITE_ENABLE_UNLOCK_NOTIFY

该选项启用sqlite3_unlock_notify()接口及其关联的功能。有关更多信息，请参阅标题为使用SQLite解锁通知功能的文档。

SQLITE_SOUNDEX

该选项启用soundex()SQL函数。

SQLITE_USE_ALLOCA

如果启用此选项，那么将在适当的情况下使用alloca()内存分配器。这导致了一个更小和更快的二进制。当然，SQLITE_USE_ALLOCA编译时只适用于支持alloca()的系统。

SQLITE_USE_FCNTL_TRACE

此选项会导致SQLite发出额外的SQLITE_FCNTL_TRACE文件控件，以向VFS提供补充信息。“vfslog.c”扩展使用它来提供增强的VFS活动日志。

YYTRACKMAXSTACKDEPTH

此选项会使用sqlite3_status（SQLITE_STATUS_PARSER_STACK，...）接口跟踪和报告LALR（1）解析器堆栈深度。SQLite的LALR（1）解析器具有固定的堆栈深度（在编译时使用YYSTACKDEPTH选项确定）。此选项可用于帮助确定应用程序是否已接近超过最大LALR（1）堆栈深度。

8.选项可以禁用通常打开的功能
SQLITE_DISABLE_LFS

如果定义了此C预处理器宏，则禁用大文件支持。

SQLITE_DISABLE_DIRSYNC

如果定义了此C预处理器宏，则会禁用目录同步。SQLite通常会尝试在删除文件时同步父目录，以确保目录条目在磁盘上立即更新。

SQLITE_DISABLE_FTS3_UNICODE

如果定义了此C预处理器宏，则FTS3中的unicode61标记器从构建中被省略，并且对于应用程序不可用。

SQLITE_DISABLE_FTS4_DEFERRED

如果此C预处理器宏禁用FTS4中的“延期标记”优化。“延迟标记”优化可避免为集合中大多数文档中的术语加载大量发布列表，而只是在文档源中扫描这些标记。无论是否进行此优化，FTS4都应该得到完全相同的答案。

SQLITE_DISABLE_INTRINSIC

此选项禁止在GCC和Clang中使用编译器特定的内置函数，例如__builtin_bswap32()和__builtin_add_overflow()，或者使用MSVC中的_byteswap_ulong()和_ReadWriteBarrier()。

9. Options To Omit Features
通过省略未使用的功能，可以使用以下选项来减小已编译库的大小。这可能仅适用于空间特别紧张的嵌入式系统，因为即使包含SQLite库的所有功能都相对较小。不要忘记告诉你的编译器优化二进制大小！（如果使用GCC，则为-Os选项）。告诉编译器对大小进行优化通常比使用任何这些编译时选项对库占用量的影响要大得多。您还应该验证调试选项是否被禁用。

本节中的宏不需要值。以下编译开关都具有相同的效果：

-DSQLITE_OMIT_ALTERTABLE

-DSQLITE_OMIT_ALTERTABLE=1

-DSQLITE_OMIT_ALTERTABLE=0

如果定义了这些选项中的任何一个，那么当使用'lemon'工具生成parse.c文件并编译生成keywordhash.h文件的'mkkeywordhash'工具时，也必须定义相同的一组SQLITE_OMIT_ *选项。正因为如此，这些选项只能在库来自标准源而不是混合时才能使用。某些SQLITE_OMIT_ *选项在与合并一起使用时可能会起作用，或者似乎起作用。但这并不能保证。一般情况下，请始终从规范源编译以利用SQLITE_OMIT_ *选项。

重要说明：SQLITE_OMIT_ *选项可能不适用于合并。SQLITE_OMIT_ *编译时选项通常只有在SQLite由规范源文件构建时才能正常工作。  
可以生成与预定的一组SQLITE_OMIT_ *选项配合使用的特殊版本的SQLite合并。为此，请在规范源代码分发中复制Makefile.linux-gcc makefile模板。将副本的名称更改为“Makefile”。然后编辑“Makefile”来设置适当的编译时选项。然后键入：

make clean; make sqlite3.c
然后可以将生成的“sqlite3.c”合并代码文件（及其关联的头文件“sqlite3.h”）移动到非unix平台，以便使用本机编译器进行最终编译。

SQLITE_OMIT_ *选项不受支持。通过这个，我们的意思是说，在当前版本中省略构建代码的SQLITE_OMIT_ *选项可能会在下一版本中变为无效。或者反过来说：在当前版本中没有任何操作的SQLITE_OMIT_ *可能会导致在下一个版本中排除代码。另外，并不是所有的SQLITE_OMIT_ *选项都经过测试。某些SQLITE_OMIT_ *选项可能会导致SQLite发生故障和/或提供不正确的答案。

 Important Note: The SQLITE_OMIT_* compile-time options are mostly unsupported. 
以下是可用的OMIT选项：

SQLITE_OMIT_ALTERTABLE

定义此选项时，ALTER TABLE命令不包含在库中。执行ALTER TABLE语句会导致解析错误。

SQLITE_OMIT_ANALYZE

定义此选项时，从构建中省略ANALYZE命令。

SQLITE_OMIT_ATTACH

当定义这个选项时，从构建中省略ATTACH和DETACH命令。

SQLITE_OMIT_AUTHORIZATION

定义此选项会忽略库中的授权回调功能。库中不存在sqlite3_set_authorizer()API函数。

SQLITE_OMIT_AUTOINCREMENT

该选项省略了AUTOINCREMENT功能。当定义宏时，声明为“INTEGER PRIMARY KEY AUTOINCREMENT”的列的行为与插入NULL时声明为“INTEGER PRIMARY KEY”的列的行为相同。sqlite_sequence系统表既不被创建，也不被尊重，如果它已经存在。

SQLITE_OMIT_AUTOINIT

为了向后兼容缺少sqlite3_initialize()接口的旧版SQLite，sqlite3_initialize()接口会在进入某些关键接口（如sqlite3_open()，sqlite3_vfs_register()和sqlite3_mprintf()）时自动调用。通过使用SQLITE_OMIT_AUTOINIT C预处理器宏构建SQLite，可以省略以这种方式自动调用sqlite3_initialize()的开销。当使用SQLITE_OMIT_AUTOINIT构建时，SQLite将不会自动初始化，并且应用程序需要在开始使用SQLite库之前直接调用sqlite3_initialize()。

SQLITE_OMIT_AUTOMATIC_INDEX

该选项用于省略自动索引功能。另请参阅：SQLITE_DEFAULT_AUTOMATIC_INDEX。

SQLITE_OMIT_AUTORESET

默认情况下，如果需要，sqlite3_step()接口将自动调用sqlite3_reset()来重置预准备语句。此编译时选项更改该行为，以便sqlite3_step()在返回除SQLITE_ROW，SQLITE_BUSY或SQLITE_LOCKED之外的任何其他名称后再次调用时，将返回SQLITE_MISUSE，除非存在对sqlite3_reset()的干预调用。在SQLite 版本3.6.23.1（2010-03-26）及更早版本，sqlite3_step()用于在没有对sqlite3_reset()进行干预调用的情况下返回SQLITE_ROW以外的任何内容时再次调用SQLITE_MISUSE。这对一些写得不好的智能手机应用程序造成了问题，这些应用程序没有正确处理SQLITE_LOCKED和SQLITE_BUSY错误返回。不是修复了许多有缺陷的智能手机应用程序，而是在3.6.23.2中更改了SQLite的行为以自动重置准备好的语句。但是这种改变引起了其他实际上正在寻找SQLITE_MISUSE返回以终止其查询循环的不正确应用程序中的问题。（任何时候应用程序都会从​​SQLite获取SQLITE_MISUSE错误代码，这意味着应用程序正在滥用SQLite接口并因此被错误地实现。版本3.7.5（2011-02-01），努力让所有（破碎的）应用程序再次运行，而不必实际修复应用程序。

SQLITE_OMIT_AUTOVACUUM

如果定义了此选项，则库不能创建或写入支持auto_vacuum的数据库。执行PRAGMA auto_vacuum语句不是错误（因为未知的PRAGMA被忽略），但不返回值或修改数据库文件中的自动真空标志。如果支持自动吸引的数据库由使用此选项编译的库打开，它将以只读模式自动打开。

SQLITE_OMIT_BETWEEN_OPTIMIZATION

此选项禁用使用带有BETWEEN运算符的WHERE子句条件的索引。

SQLITE_OMIT_BLOB_LITERAL

定义此选项时，不可能使用X'ABCD'语法在SQL语句中指定blob。

SQLITE_OMIT_BTREECOUNT

定义此选项时，会优化加速计数表中所有条目的优化（换句话说，有助于“SELECT count（*）FROM table”运行速度更快的优化）将被忽略。

SQLITE_OMIT_BUILTIN_TEST

此编译时选项已重命名为SQLITE_UNTESTABLE。

SQLITE_OMIT_CAST

该选项会导致SQLite省略对CAST运算符的支持。

SQLITE_OMIT_CHECK

该选项使SQLite省略对CHECK约束的支持。解析器仍然会接受SQL语句中的CHECK约束，它们将不会被强制执行。

SQLITE_OMIT_COMPILEOPTION_DIAGS

此选项用于省略SQLite中的编译时选项诊断，包括sqlite3_compileoption_used()和sqlite3_compileoption_get()C / C ++函数，sqlite_compileoption_used()和sqlite_compileoption_get()SQL函数以及compile_options编译指示。

SQLITE_OMIT_COMPLETE

该选项可以省略sqlite3_complete()和sqlite3_complete16()接口。

SQLITE_OMIT_COMPOUND_SELECT

该选项用于省略复合SELECT功能。使用UNION，UNION ALL，INTERSECT或EXCEPT复合SELECT操作符的SELECT语句将导致解析错误。在VALUES子句中具有多个值的INSERT语句作为复合SELECT在内部实现。因此，该选项也禁止使用INSERT INTO ... VALUES ...语句插入多个单行的功能。

SQLITE_OMIT_CTE

这个选项会导致对公用表表达式的支持被忽略。

SQLITE_OMIT_DATETIME_FUNCS

如果定义了这个选项，则省略SQLite的内置日期和时间操作函数。具体来说，SQL函数julianday()，date()，time()，datetime()和strftime()不可用。缺省列值CURRENT_TIME，CURRENT_DATE和CURRENT_TIMESTAMP仍然可用。

SQLITE_OMIT_DECLTYPE

该选项使SQLite省略对sqlite3_column_decltype()和sqlite3_column_decltype16()接口的支持。

SQLITE_OMIT_DEPRECATED

此选项会导致SQLite省略对已标记为已弃用的接口的支持。这包括sqlite3_aggregate_count()，sqlite3_expired()，sqlite3_transfer_bindings()，sqlite3_global_recover()，sqlite3_thread_cleanup()和sqlite3_memory_alarm()接口。

SQLITE_OMIT_DISKIO

该选项省略了写入磁盘的所有支持，并且强制数据库仅存在于内存中。此选项尚未维护，可能不适用于较新版本的SQLite。

SQLITE_OMIT_EXPLAIN

定义该选项会导致库中省略EXPLAIN命令。试图执行EXPLAIN语句会导致解析错误。

SQLITE_OMIT_FLAG_PRAGMAS

该选项省略了对查询和设置布尔属性的PRAGMA命令子集的支持。

SQLITE_OMIT_FLOATING_POINT

该选项用于省略SQLite库中的浮点数支持。指定时，将浮点数指定为文字（即“1.01”）会导致解析错误。将来，该选项还可能禁用其他浮点功能，例如sqlite3_result_double()，sqlite3_bind_double()，sqlite3_value_double()和sqlite3_column_double()API函数。

SQLITE_OMIT_FOREIGN_KEY

如果定义了此选项，则不会识别外键约束语法。

SQLITE_OMIT_GET_TABLE

此选项会导致对sqlite3_get_table()和sqlite3_free_table()的支持被省略。

SQLITE_OMIT_INCRBLOB

该选项会导致对增量BLOB I / O的支持被忽略。

SQLITE_OMIT_INTEGRITY_CHECK

该选项省略对integrity_check编译指示的支持。

SQLITE_OMIT_LIKE_OPTIMIZATION

该选项禁止SQLite使用索引来帮助解决WHERE子句中的LIKE和GLOB运算符。

SQLITE_OMIT_LOAD_EXTENSION

该选项省略了SQLite中的整个扩展加载机制，包括sqlite3_enable_load_extension()和sqlite3_load_extension()接口。

SQLITE_OMIT_LOCALTIME

该选项省略了日期和时间函数中的“localtime”修饰符。尝试在不支持本地时间概念的平台上编译日期和时间函数时，此选项有时很有用。

SQLITE_OMIT_LOOKASIDE

该选项省略了后备内存分配器。

SQLITE_OMIT_MEMORYDB

当定义这个时，库不尊重特殊数据库名称“：memory：”（通常用于创建内存数据库）。如果将“：memory：”传递给sqlite3_open()，sqlite3_open16()或sqlite3_open_v2()，则将打开或创建一个具有该名称的文件。

SQLITE_OMIT_OR_OPTIMIZATION

此选项禁用SQLite将OR索引与由OR运算符连接的WHERE子句的条款一起使用的功能。

SQLITE_OMIT_PAGER_PRAGMAS

定义此选项会省略与构建中的寻呼机子系统相关的编译指示。

SQLITE_OMIT_PRAGMA

该选项用于省略库中的PRAGMA命令。请注意，除此之外，定义忽略特定杂注的宏是有用的，因为它们也可能会删除其他子系统中的支持代码。该宏只能删除PRAGMA命令。

SQLITE_OMIT_PROGRESS_CALLBACK

可以定义此选项以省略在长时间运行的SQL语句期间发出“进度”回调的功能。库中不存在sqlite3_progress_handler()API函数。

SQLITE_OMIT_QUICKBALANCE

此选项省略了另一种更快的B-Tree平衡程序。使用这个选项使得SQLite稍微小一点，但是它的运行速度稍慢。

SQLITE_OMIT_REINDEX

定义此选项时，REINDEX命令不包含在库中。执行REINDEX语句会导致解析错误。

SQLITE_OMIT_SCHEMA_PRAGMAS

定义此选项会忽略用于从构建中查询数据库模式的编译指示。

SQLITE_OMIT_SCHEMA_VERSION_PRAGMAS

定义此选项将忽略用于从构建中查询和修改数据库架构版本和用户版本的编译指示。具体来说，schema_version和user_version PRAGMA被省略。

SQLITE_OMIT_SHARED_CACHE

此选项构建SQLite时不支持共享缓存模式。在与共享高速缓存管理关联的B-Tree子系统中，sqlite3_enable_shared_cache()和相当数量的逻辑被省略。

SQLITE_OMIT_SUBQUERY

如果已定义，则将省略对子选择和IN()运算符的支持。

SQLITE_OMIT_TCL_VARIABLE

如果定义了该宏，则省略用于将SQL变量自动绑定到TCL变量的特殊“$”语法。

SQLITE_OMIT_TEMPDB

此选项省略对TEMP或TEMPORARY表的支持。

SQLITE_OMIT_TRACE

此选项省略对sqlite3_profile()和sqlite3_trace()接口及其关联逻辑的支持。

SQLITE_OMIT_TRIGGER

定义此选项将省略对TRIGGER对象的支持。在这种情况下，CREATE TRIGGER或DROP TRIGGER命令都不可用，尝试执行这两个命令都会导致解析错误。该选项也禁用外键约束的强制执行，因为实现触发器的代码以及此选项忽略的代码也用于实现外键操作。

SQLITE_OMIT_TRUNCATE_OPTIMIZATION

SQLite的默认构建，如果DELETE语句没有WHERE子句并且在没有触发器的表上操作，则会发生优化，通过删除和重新创建表来导致DELETE发生。删除和重新创建表格通常比逐行删除表格内容要快得多。这是“截断优化”。

SQLITE_OMIT_UTF16

该宏用于省略对UTF16文本编码的支持。当定义了所有返回或接受UTF16编码文本的API函数不可用时。这些函数可以通过它们以'16'结尾的事实来标识，例如sqlite3_prepare16()，sqlite3_column_text16()和sqlite3_bind_text16()。

SQLITE_OMIT_VACUUM

定义此选项时，库中不包含VACUUM命令。执行VACUUM语句会导致解析错误。

SQLITE_OMIT_VIEW

定义此选项将省略对VIEW对象的支持。在这种情况下，CREATE VIEW和DROP VIEW命令都不可用，而尝试执行这两个命令都会导致解析错误。警告：如果定义了此宏，将无法打开架构包含VIEW对象的数据库。

SQLITE_OMIT_VIRTUALTABLE

该选项省略了对SQLite中虚拟表机制的支持。

SQLITE_OMIT_WAL

该选项省略了“预写日志”（aka“WAL”）功能。

SQLITE_OMIT_WSD

此选项构建不包含可写入静态数据（WSD）的SQLite库的一个版本。WSD是全局变量和/或静态变量。有些平台不支持WSD，为了使SQLite能够运行这些平台，这个选项是必需的。与使SQLite库变小的其他OMIT选项不同，此选项实际上增加了SQLite的大小，并使其运行速度稍慢。如果正在为不支持WSD的嵌入式目标构建SQLite，则只能使用此选项。

SQLITE_OMIT_XFER_OPT

该选项省略了对帮助“INSERT INTO ... SELECT ...”形式的语句运行速度更快的优化的支持。

SQLITE_UNTESTABLE

一个标准的SQLite构建包含少量与sqlite3_test_control()相关的逻辑，以便练习难以验证的SQLite核心部分。这个编译时选项省略了额外的测试逻辑。这个编译时选项在SQLite版本3.16.0（2017-01-02）之前称为“SQLITE_OMIT_BUILTIN_TEST”。该名称已更改为更好地描述使用它的含义。设置这个编译时选项可以防止SQLite被完全测试。分支测试覆盖率从100％下降到大约95％。SQLite开发人员遵循美国宇航局的“飞行你测试和测试你飞行的东西”的原则。如果此选项启用交付但禁用测试，则违反此原则。但是，如果在测试过程中启用此选项，则不是所有分支都可以访问。因此，

SQLITE_ZERO_MALLOC

此选项忽略了构建中的默认内存分配程序和调试内存分配程序，并替换始终失败的分区内存分配程序。SQLite不会运行这个存根分配器，因为它将无法分配内存。但是这个存根可以在开始时使用sqlite3_config（SQLITE_CONFIG_MALLOC，...）或sqlite3_config（SQLITE_CONFIG_HEAP，...）替换。所以这个编译时选项的最终效果是，它允许将SQLite编译并链接到不支持malloc()，free()和/或realloc()的系统库。

10.分析和调试选项
SQLITE_DEBUG

SQLite源代码包含数千个assert()语句，用于验证内部假设和子例程前置条件和后置条件。这些assert()语句通常会关闭（它们不生成代码），因为将它们打开会使SQLite运行速度降低大约三倍。但是对于测试和分析，打开assert()语句是有用的。SQLITE_DEBUG编译时选项执行此操作。SQLITE_DEBUG还启用了一些其他调试功能，例如特殊的PRAGMA语句，它们打开用于VDBE和代码生成器故障排除和分析的跟踪和列表功能。

SQLITE_MEMDEBUG

SQLITE_MEMDEBUG选项会导致一个检测调试内存分配器被用作SQLite中的默认内存分配器。仪器化内存分配器检查动态分配内存的滥用情况。误用的例子包括在释放内存之后使用内存，注销内存分配的结尾，释放​​先前未从内存分配器获得的内存，或者未能初始化新分配的内存。

11.特定于Windows的选项
SQLITE_WIN32_HEAP_CREATE

此选项强制启用Win32本地内存分配程序时创建专用堆以保存所有内存分配。

SQLITE_WIN32_MALLOC_VALIDATE

如果启用assert()，则此选项会强制Win32本机内存分配程序启用时对HeapValidate()函数进行策略性调用。

12. Compiler Linkage Control
以下宏指定某些类型的SQLite构建的接口链接。Makefiles通常会自动处理这些宏。应用程序开发人员不需要担心这些宏。以下关于这些宏的文档包含完整性。

SQLITE_API

该宏为SQLite标识了一个外部可见的接口。这个宏有时被设置为“extern”。但是这个定义是特定于编译器的。

SQLITE_APICALL

该宏标识SQLite中公共接口例程使用的调用约定。这个宏通常被定义为什么都不是，尽管在Windows上它有时可以设置为“__cdecl”或“__stdcall”。“__cdecl”设置是默认设置，但是当SQLite打算编译为Windows系统库时使用“__stdcall”。单个函数声明应该包含以下内容之一：SQLITE_APICALL，SQLITE_CALLBACK，SQLITE_CDECL或SQLITE_SYSCALL。

SQLITE_CALLBACK

这个宏指定了SQLite中回调指针所使用的调用约定。这个宏通常被定义为什么都不是，尽管在Windows上它有时可以设置为“__cdecl”或“__stdcall”。“__cdecl”设置是默认设置，但是当SQLite打算编译为Windows系统库时使用“__stdcall”。单个函数声明应该包含以下内容之一：SQLITE_APICALL，SQLITE_CALLBACK，SQLITE_CDECL或SQLITE_SYSCALL。

SQLITE_CDECL

该宏指定SQLite中可变参数接口例程使用的调用约定。这个宏通常被定义为什么都没有，尽管在Windows上它有时可以设置为“__cdecl”。此宏用于可变参数例程，因此不能设置为“__stdcall”，因为__stdcall调用约定不支持可变参数函数。单个函数声明应该包含以下内容之一：SQLITE_APICALL，SQLITE_CALLBACK，SQLITE_CDECL或SQLITE_SYSCALL。

SQLITE_SYSCALL

该宏标识操作系统接口用于SQLite构建平台的调用约定。这个宏通常被定义为什么都没有，尽管在Windows上它有时可以设置为“__stdcall”。单个函数声明应该包含以下内容之一：SQLITE_APICALL，SQLITE_CALLBACK，SQLITE_CDECL或SQLITE_SYSCALL。

SQLITE_TCLAPI

该宏指定了TCL库接口例程使用的调用约定。该宏不被SQLite核心使用，但仅被TCL接口和TCL测试套件使用。这个宏通常被定义为什么都没有，尽管在Windows上它有时可以设置为“__cdecl”。该宏在TCL库接口例程上使用，这些例程始终编译为__cdecl，即使在更喜欢使用__stdcall的平台上也是如此，因此除非平台作为支持__stdcall的自定义TCL库构建，否则不应将此宏设置为__stdcall。此宏不能与SQLITE_APICALL，SQLITE_CALLBACK，SQLITE_CDECL或SQLITE_SYSCALL结合使用。


## crash
- plcrashreporter  [![github][1]](https://github.com/plausiblelabs/plcrashreporter)

[1]: /img/github.png
