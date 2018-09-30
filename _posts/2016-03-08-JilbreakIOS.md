---
layout:         post
title:          IOS 越狱
subtitle:       IOS 越狱
date:           2016-01-04 11:36:00
author:         nomadli
header-img:     ../img/post-bg-ios9-web.jpg
catalog:        true
tags:
        - IOS
---

* content
{:toc}

## 目标
1. 内核、amfid、libmiss.dylib：三者配合实现了代码签名
2. 内核：对内存页属性的保护
3. 获取 root 权限：重新 mount 磁盘分区需要 root 权限

## USB接口权限
1. AFC apple file connect 文件管理
2. MobileBackup				系统备份
3. image Mounter				dmg文件挂载
4. installation Proxy		安装服务

## 寄存器
- 16位和32位的寄存器是可以互相使用,含义相同
- 16位中使用32位寄存器,机器码前会加一个字节66
- 16位和32位中,sp、bp、si、di不能使用低8位,64位可以
- 寄存器对应表
8位|16位|32位|64位|64位新增
:--:|:--:|:--:|:--:|:--:
al|ax|eax|rax|r8
cl|cx|ecx|rcx|r9
dl|dx|edx|rdx|r10
bl|bx|ebx|rbx|r11
ah|sp|esp|rsp|r12
ch|bp|ebp|rbp|r13
dh|si|esi|rsi|r14
bh|di|edi|rdi|r15
- 在64位寄存器中可以拆分使用8、16、32位,默认使用高位,spl、bpl、sil、dil为低8位
64位寄存器|8位|16位|32位
:--:|:--:|:--:|:--:
rax|al|ax|eax
rcx|cl|cx|ecx
rdx|dl|dx|edx
rbx|bl|bx|ebx
rsp|ah、spl|sp|esp
rbp|ch、bpl|bp|ebp
rsi|dh、sil|si|esi
rdi|bh、dil|di|edi
r8|r8b|r8w|r8d
r9|r9b|r9w|r9d
r10|r10b|r10w|r10d
r11|r11b|r11w|r11d
r12|r12b|r12w|r12d
r13|r13b|r13w|r13d
r14|r14b|r14w|r14d
r15|r15b|r15w|r15d


## 汇编 mac inter x64
指令|作用
:--|:--
je|满足条件跳转
jnz|不满足条件跳转
jmp|无条件跳转
RAX RBX RCX RDX RBP RSI RDI RSP|64位通用寄存器
R8 -- R15|64位通用寄存器
RIP|指令寄存器
RAX|返回值
RSP|栈顶指针寄存器 call后会被还原
RBP|alloca()动态地在栈上分配空间时,作为栈顶指针寄存器,call后会还原
RDI,RSI,RDX,RCX,R8,R9|参数传递顺序,不会还原
RBX,RBP,R12,R13,R14,R15|call后会被还原
R10,R11|call不会还原
xmm0-xmm1|浮点参数和返回值,不会还原
xmm2-xmm7|浮点参数,不会还原
xmm8-xmm15|浮点寄存器,不会还原
mmx0-mmx7|浮点寄存器,不会还原
st0|临时寄存器,返回long double,不会还原
st1-st7|被调用者使用,不还原
fs|系统专用,不还原

## Hopper Disassembler v4
- 函数入口
```
push rbp
mov rbp, rsp
```
- 返回值在rax寄存器中
```
pop rbp
ret
```
- 函数调用 jmp call
- lea 取偏移地址
- block
```
void sub_xxxxxx(void * _block) 无参数block
(*(block + 0x10))(block, _cmd, block); block本身,_cmd为调用block的函数自身的_cmd,无参

void sub_100001456(void * _block, struct NSString * arg1) 有参数
(*(block + 0x10))(block, 参数, block);

引用block外部变量,生成一个结构体地址__NSConcreteStackBlock_xxxx
*(__NSConcreteStackBlock_xxxx + 0x8) = 0xc0000000;  固定值
*(__NSConcreteStackBlock_xxxx + 0x10) = block本身的sbu_xxxx;
*(__NSConcreteStackBlock_xxxx + 0x18) = 0x100002030; 描述地址
*(__NSConcreteStackBlock_xxxx + 0x20) 保存引用
```
- 128XMM 指令
```
pxor xmm0, xmm0     寄存器置0
pcmpeqd xmm0, xmm0  寄存器置1
movq                加载数据到mmx寄存器
punpcklbw       交叉组合低位4字中的字节
punpcklwd       交叉组合低位双字中字
punpckldq       交叉组合低位四字中的双字
punpcklqdq      交叉组合低位四字
```
