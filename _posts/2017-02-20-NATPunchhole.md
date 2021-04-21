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
- 信令服务器，作用是帮助客户机沟通IP和PORT信息。
- STUN Server，客户机判断自己所在的NAT类型、公网IP、端口。
- Full Cone NAT 相同内网IP:端口映射到相同的端口,不限制外网IP端口
- Restricted Cone NAT 相同内网IP:端口映射到相同的端口, 外网IP必须是被主动访问过的
- Port Restricted Cone NAT 相同内网IP:端口映射到相同的端口, 外网IP:端口必须是被主动访问过的
- Symmetric NAT 地址4要素映射到不同的端口
- Full Cone NAT 与 Full Cone NAT：![1]
- Full Cone NAT 与 Restricted Cone NAT或Port Restricted Cone NAT. ![2]
- Full Cone NAT 与 Symmetric NAT ![3]
- Restricted Cone NAT 与 Restricted Cone NAT、Restricted Cone NAT 与 Port Restricted Cone NAT、Port Restricted Cone NAT 与 Port Restricted Cone NAT ![4]
- Restricted Cone NAT 与 Symmetric NAT ![5]
- Port Restricted Cone NAT 不能 Symmetric NAT ![6]
- Symmetric NAT 不能 Symmetric NAT ![7]

## 无服务器穿透(pwnat)
1. 服务器一定频率发送ICMP发送ping的响应包(固定内容、目标IP), 客户端发送ICMP超时包(包含服务器发的固定ICMP响应)
2. 服务器对称型Symmetric NAT无法穿透![8]

## 免费STUN服务器
- stun.xten.com:3478
- stun.voipbuster.com:3478
- stun.sipgate.net:3478
- stun.ekiga.net:3478
- stun.ideasip.com:3478
- stun.schlund.de:3478
- stun.voiparound.com:3478
- stun.voipbuster.com:3478
- stun.voipstunt.com:3478
- stun.counterpath.com:3478
- stun.1und1.de:3478
- stun.gmx.net:3478
- stun.callwithus.com:3478
- stun.counterpath.net:3478
- stun.internetcalls.com:3478
- numb.viagenie.ca:3478


[1]: /img/nat_punch_hole/Full_Cone-Full_Cone.png
[2]: /img/nat_punch_hole/Full_Cone-Port_Restricted_Cone.png
[3]: /img/nat_punch_hole/Full_Cone-Symmetric_NAT.png
[4]: /img/nat_punch_hole/Port_Restricted_Cone-Port_Restricted_Cone.png
[5]: /img/nat_punch_hole/Restricted_Cone-Symmetric_NAT.png
[6]: /img/nat_punch_hole/Port_Restricted_Cone-Symmetric_NAT.png
[7]: /img/nat_punch_hole/Symmetric_NAT-Symmetric_NAT.png
[8]: /img/nat_punch_hole/pwnat.jpg