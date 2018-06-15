---
layout: post
title:  "saltstack"
date:   2018-05-03 13:13:00
categories: Swig
excerpt: salt 相关
---

# master
- salt-key -L 查看所有通过的minion key
- salt 'key' test.ping 测试被控主机联通性
- salt 'key' cmd.run 'ls -ls' 执行linux命令
- salt 'key' cmd.exec_code python 'print sys.version'

# minion
- /etc/salt/minion


 