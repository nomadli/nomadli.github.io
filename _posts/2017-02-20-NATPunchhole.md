---
layout:         post
title:          NAT Punch hole
subtitle:       NAT Punch hole
date:           2017-02-20 11:17:00
author:         nomadli
header-img:     ../img/bg-coffee.jpeg
catalog:        true
tags:
        - other
---

* content
{:toc}

## 通过服务器穿透
1.  信令服务器，作用是帮助客户机沟通IP和PORT信息。
2.  STUN Server，客户机判断自己所在的NAT环境。
3.  Full Cone NAT 与 Full Cone NAT：![1]
4.  Full Cone NAT 与 Restricted Cone NAT或Port Restricted Cone NAT. ![2]
5.  Full Cone NAT 与 Symmetric NAT ![3]
6.  Restricted Cone NAT 与 Restricted Cone NAT、Restricted Cone NAT 与 Port Restricted Cone NAT、Port Restricted Cone NAT 与 Port Restricted Cone NAT ![4]
7.  Restricted Cone NAT 与 Symmetric NAT ![5]
8.  Port Restricted Cone NAT 不能 Symmetric NAT ![6]
9.  Symmetric NAT 不能 Symmetric NAT ![7]

## 无服务器穿透





[1]: /img/nat_punch_hole/Full_Cone-Full_Cone.png
[2]: /img/nat_punch_hole/Full_Cone-Port_Restricted_Cone.png
[3]: /img/nat_punch_hole/Full_Cone-Symmetric_NAT.png
[4]: /img/nat_punch_hole/Port_Restricted_Cone-Port_Restricted_Cone.png
[5]: /img/nat_punch_hole/Restricted_Cone-Symmetric_NAT.png
[6]: /img/nat_punch_hole/Port_Restricted_Cone-Symmetric_NAT.png
[7]: /img/nat_punch_hole/Symmetric_NAT-Symmetric_NAT.png