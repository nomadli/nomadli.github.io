---
layout:         post
title:          coroutine
subtitle:       coroutine
date:           2021-04-13 10:26:00
author:         nomadli
header-img:     ../img/bg-coffee.jpeg
catalog:        true
tags:
        - other
---

* content
{:toc}

## 常见库
- libtask ucontext_t 实现
- libpcl
- coro
- lthread
- libCoroutine
- libconcurrency
- libcoro
- ribs2
- [libmill->libdill](https://github.com/sustrik/libdill.git)
- libaco
- libco
- tbox

## 上下文
- 寄存器分类
    - 通用目的寄存器（AX\BX\CX\DX...)
    - 段寄存器(CS\DS\SS...)
    - 标识寄存器(EFLAGS)
    - 指令(IR)及指令指针寄存器(IP)
    - 其他寄存器，比如SP\GDTR\LDTR\CR3，等等
    - FPU、MMX、SSE、AVX :KB级数据量
- 上下文保存
    - setjmp, longjmp, jmp_buf
    - sigsetjmp, siglongjmp, sigjmp_buf
    - getcontext, setcontext, makecontext, swapcontext, ucontext_t(mcontext_t)
- 开销
    - 寄存器保存 减少不必要的保存恢复
    - cpu cache miss cpu亲和性,恢复到同一核上
    - 内存页管理TLB(转换旁路缓冲)
        - 没有PCID的CPU,CR3寄存器写操作,会清空整个TLB
        - 有PCID的CPU,某核上的页式映射变化后,使其他核页式映射失效(multi-CPU TLB shootdown)其他核中断几百个时钟周期
        - linux 内存映射 PGD+PUD+PMD+PTE+Offset
    - 分支预测(branch direction, branch target, return buffer)
- goroutine 上下文切换仅涉PC、SP、DX三个寄存器

## libmill 固定大小的栈 libdill 动态调整的栈
- 项目机构
    - AUTHORS
    - README.md
    - abi_version.sh
    - package_version.sh
    - autogen.sh
    - libmill.h                                     libmill头文件
    - debug.c/.h                                    调试函数
    - chan.c/.h                                     channel
    - cr.c/.h                                       协程coroutine
    - file.c                                        file hook
    - poller.c/.h kqueue.inc/epoll.inc/poll.inc     io多路复用
    - mfork.c                                       进程创建并初始化
    - tcp.c/udp.c/unix.c                            tcp、udp、unix套接字
    - timer.h/.c                                    定时器
    - utils.h                                       常用工具方法
    - ssl.c                                         ssl支持
    - dns                                           dns解析
    - slist.h/.c                                    单向链表
    - list.h/.c                                     双向链表
    - stack.h/.c                                    栈
```C
//chan.h
// choose语句是根据chan的状态来决定是否执行对应动作的分支控制语句
//
// 每个协程都会有一个choose数据结构来跟踪其当前正在执行的choose操作
struct mill_choosedata {
    // 每个choose语句中，又包含了多个从句构成的列表
    struct mill_slist clauses;
    // choose语句中otherwise从句是可选的，是否有otherwise从句，0否1是
    int othws;
    // 当前choose语句中，是否有指定deadline，未指定时为-1
    int64_t ddline;
    // 当前choose语句中，chan上事件就绪的从句数量
    int available;
};

// chan ep是对chan的使用者的描述，每个ep要么利用chan发送消息，要么接收消息
//
// 每个chan有一个sender和receiver，所以每个chan包括了sender、receiver两个mill_ep成员
struct mill_ep {
    // 类型（数据发送方 或 数据接收方）
    enum {MILL_SENDER, MILL_RECEIVER} type;
    // 初始化的choose操作的序号
    int seqnum;
    // choose语句中引用该mill_ep的从句数量
    int refs;
    // choose语句中引用该mill_ep并且已经处理过的数量
    int tmp;
    // choose语句中仍然在等待该mill_ep上事件就绪的从句列表
    struct mill_list clauses;
};

// chan
struct mill_chan_ {
    // channel里面存储的元素的尺寸(单位字节)
    size_t sz;
    // 每个chan上有一个seader和receiver
    // sender记录了等待在chan上执行数据发送操作的从句列表，receiver则记录了等待接收数据的从句列表
    struct mill_ep sender;
    struct mill_ep receiver;
    // 当前chan的引用计数（引用计数为0的时候chclose才会真正释放资源）
    int refcount;
    // 该chan上是否已经调用了chdone()，0否1是
    int done;
    // 存储消息数据的缓冲区紧跟在chan结构体后面
    // - bufsz代表消息缓冲区可容纳的最大消息数量
    // - items表示缓冲区中当前的消息数量
    // - first代表缓冲区中可接收的下一个消息的位置，缓冲区末尾有一个元素来存储chdone()写的数据
    size_t bufsz;
    size_t items;
    size_t first;
    // 调试信息
    struct mill_debug_chan debug;
};

// 该结构体代表choose语句中的一个从句，例如in、out、otherwise
struct mill_clause {
    // 等待this.ep事件就绪的从句列表(迭代器）
    struct mill_list_item epitem;
    // 该从句隶属的choose语句所包含的从句列表(迭代器)
    struct mill_slist_item chitem;
    // 创建该从句的协程
    struct mill_cr *cr;
    // 该从句正在等待的chan endpoint
    struct mill_ep *ep;
    // 对于out从句，val指向要发送的数据；对于in从句，val为NULL
    void *val;
    // 该从句执行完成后要跳转到第idx个从句
    int idx;
    // 是否有与当前从句匹配的pee(比如当前从句为ch上的写，是否有ch上的读从句)，0否1是
    int available;
    // 该从句是否在chan的sender或receiver列表中，0否1是
    int used;
};

// 返回包含该endpoint的chan
struct mill_chan_ *mill_getchan(struct mill_ep *ep);
```
```C
//chan.c
// 每个choose语句都要分配一个单独的序号
static int mill_choose_seqnum = 0;

// 返回包含ep的chan(根据端点类型获取)
struct mill_chan_ *mill_getchan(struct mill_ep *ep) {
    switch(ep->type) {
    case MILL_SENDER:
        return mill_cont(ep, struct mill_chan_, sender);
    case MILL_RECEIVER:
        return mill_cont(ep, struct mill_chan_, receiver);
    default:
        assert(0);
    }
}

// 创建一个chan
struct mill_chan_ *mill_chmake_(size_t sz, size_t bufsz, const char *created) {
    mill_preserve_debug();
    // 分配消息缓冲区的时候多申请一个元素空间用于存chdone()提交的数据，
    // chdone不能写消息缓冲区，因为会因为缓冲区满而阻塞chdone()操作，
    // libmill是单线程调度，一个阻塞就会导致整个进程被阻塞了
    struct mill_chan_ *ch = 
        (struct mill_chan_*)malloc(sizeof(struct mill_chan_) + (sz * (bufsz + 1)));
    if(!ch)
        return NULL;
    mill_register_chan(&ch->debug, created);
    // 初始化chan
    ch->sz = sz;
    ch->sender.type = MILL_SENDER;
    ch->sender.seqnum = mill_choose_seqnum;
    mill_list_init(&ch->sender.clauses);
    ch->receiver.type = MILL_RECEIVER;
    ch->receiver.seqnum = mill_choose_seqnum;
    mill_list_init(&ch->receiver.clauses);
    ch->refcount = 1;
    ch->done = 0;
    ch->bufsz = bufsz;
    ch->items = 0;
    ch->first = 0;
    mill_trace(created, "<%d>=chmake(%d)", (int)ch->debug.id, (int)bufsz);
    return ch;
}

// dup操作，只是增加chan引用计数
struct mill_chan_ *mill_chdup_(struct mill_chan_ *ch, const char *current) {
    if(mill_slow(!ch))
        mill_panic("null channel used");
    mill_trace(current, "chdup(<%d>)", (int)ch->debug.id);
    ++ch->refcount;
    return ch;
}

// 关闭chan，实际上减少引用计数直到为0再释放chan
void mill_chclose_(struct mill_chan_ *ch, const char *current) {
    if(mill_slow(!ch))
        mill_panic("null channel used");
    mill_trace(current, "chclose(<%d>)", (int)ch->debug.id);
    assert(ch->refcount > 0);
    --ch->refcount;
    if(ch->refcount)
        return;
    // 仍有依赖该chan的从句存在的话，关闭chan会出错
    if(!mill_list_empty(&ch->sender.clauses) ||
          !mill_list_empty(&ch->receiver.clauses))
        mill_panic("attempt to close a channel while it is still being used");
    mill_unregister_chan(&ch->debug);
    // 释放chan
    free(ch);
}

// 唤醒一个因为调用mill_choose_wait而阻塞的协程
// 
// choose从句中协程因为等待io事件而阻塞，所以这里唤醒阻塞的协程也意味着要清除掉这里的从句
static void mill_choose_unblock(struct mill_clause *cl) {
    struct mill_slist_item *it;
    struct mill_clause *itcl;
    for(it = mill_slist_begin(&cl->cr->choosedata.clauses); it; it = mill_slist_next(it)) {
        itcl = mill_cont(it, struct mill_clause, chitem);
        // 如果当前从句不再当前chan的sender/receiver列表中则不予处理；
        // 已经在的话则要将该从句删除，正式因为这个从句的io事件使得协程被阻塞的
        if(!itcl->used)
            continue;
        mill_list_erase(&itcl->ep->clauses, &itcl->epitem);
    }
    // 如果有指定deadline，也删除对应的定时器
    if(cl->cr->choosedata.ddline >= 0)
        mill_timer_rm(&cl->cr->timer);
    // 恢复该协程的执行
    mill_resume(cl->cr, cl->idx);
}

// choose语句初始化
static void mill_choose_init(const char *current) {
    mill_set_current(&mill_running->debug, current);
    mill_slist_init(&mill_running->choosedata.clauses);
    mill_running->choosedata.othws = 0;
    mill_running->choosedata.ddline = -1;
    mill_running->choosedata.available = 0;
    ++mill_choose_seqnum;
}

void mill_choose_init_(const char *current) {
    mill_trace(current, "choose()");
    mill_running->state = MILL_CHOOSE;
    mill_choose_init(current);
}

// choose in从句
void mill_choose_in_(void *clause, struct mill_chan_ *ch, size_t sz, int idx) {
    if(mill_slow(!ch))
        mill_panic("null channel used");
    if(mill_slow(ch->sz != sz))
        mill_panic("receive of a type not matching the channel");
    // 检查当前从句对应的可读事件是否就绪，就绪则++available记录一下
    int available = ch->done || !mill_list_empty(&ch->sender.clauses) || ch->items ? 1 : 0;
    if(available)
        ++mill_running->choosedata.available;
    // 如果当前从句可读事件未就绪，但是当前运行协程中choose语句中有从句事件就绪，返回
    if(!available && mill_running->choosedata.available)
        return;
    /* Fill in the clause entry. */
    struct mill_clause *cl = (struct mill_clause*) clause;
    cl->cr = mill_running;
    cl->ep = &ch->receiver;
    cl->val = NULL;
    cl->idx = idx;
    cl->available = available;
    cl->used = 1;
    mill_slist_push_back(&mill_running->choosedata.clauses, &cl->chitem);
    if(cl->ep->seqnum == mill_choose_seqnum) {
        ++cl->ep->refs;
        return;
    }
    cl->ep->seqnum = mill_choose_seqnum;
    cl->ep->refs = 1;
    cl->ep->tmp = -1;
}

// choose out从句
void mill_choose_out_(void *clause, struct mill_chan_ *ch, void *val, size_t sz, int idx) {
    if(mill_slow(!ch))
        mill_panic("null channel used");
    // 调用了chdone的chan不能再执行写操作
    if(mill_slow(ch->done))
        mill_panic("send to done-with channel");
    if(mill_slow(ch->sz != sz))
        mill_panic("send of a type not matching the channel");
    // 检查chan上是否写就绪
    int available = !mill_list_empty(&ch->receiver.clauses) || ch->items < ch->bufsz ? 1 : 0;
    if(available)
        ++mill_running->choosedata.available;
    // 如果chan上没有写就绪事件，但是当前协程上有其他choose从句事件就绪，返回
    if(!available && mill_running->choosedata.available)
        return;
    /* Fill in the clause entry. */
    struct mill_clause *cl = (struct mill_clause*) clause;
    cl->cr = mill_running;
    cl->ep = &ch->sender;
    cl->val = val;
    cl->available = available;
    cl->idx = idx;
    cl->used = 1;
    mill_slist_push_back(&mill_running->choosedata.clauses, &cl->chitem);
    if(cl->ep->seqnum == mill_choose_seqnum) {
        ++cl->ep->refs;
        return;
    }
    cl->ep->seqnum = mill_choose_seqnum;
    cl->ep->refs = 1;
    cl->ep->tmp = -1;
}

// choose从句deadline对应的超时回调，销毁所有的choose从句并resume协程
static void mill_choose_callback(struct mill_timer *timer) {
    struct mill_cr *cr = mill_cont(timer, struct mill_cr, timer);
    struct mill_slist_item *it;
    for(it = mill_slist_begin(&cr->choosedata.clauses); it; it = mill_slist_next(it)) {
        struct mill_clause *itcl = mill_cont(it, struct mill_clause, chitem);
        mill_assert(itcl->used);
        mill_list_erase(&itcl->ep->clauses, &itcl->epitem);
    }
    mill_resume(cr, -1);
}

// choose deadline从句
void mill_choose_deadline_(int64_t ddline) {
    if(mill_slow(mill_running->choosedata.othws || mill_running->choosedata.ddline >= 0))
        mill_panic("multiple 'otherwise' or 'deadline' clauses in a choose statement");
    if(ddline < 0)
        return;
    mill_running->choosedata.ddline = ddline;
}

// choose otherwise从句
void mill_choose_otherwise_(void) {
    if(mill_slow(mill_running->choosedata.othws ||
          mill_running->choosedata.ddline >= 0))
        mill_panic("multiple 'otherwise' or 'deadline' clauses in a choose statement");
    mill_running->choosedata.othws = 1;
}

// 往chan追加数据val
static void mill_enqueue(struct mill_chan_ *ch, void *val) {
    // 如果chan上还有关联的receiver执行choose in从句，唤醒对应的协程收数据（当然先写数据再唤醒）
    if(!mill_list_empty(&ch->receiver.clauses)) {
        mill_assert(ch->items == 0);
        struct mill_clause *cl = mill_cont(
            mill_list_begin(&ch->receiver.clauses), struct mill_clause, epitem);
        // 写数据
        memcpy(mill_valbuf(cl->cr, ch->sz), val, ch->sz);
        // 唤醒收数据的协程
        mill_choose_unblock(cl);
        return;
    }
    // 只写数据
    assert(ch->items < ch->bufsz);
    size_t pos = (ch->first + ch->items) % ch->bufsz;
    memcpy(((char*)(ch + 1)) + (pos * ch->sz) , val, ch->sz);
    ++ch->items;
}

// 从chan中取队首的数据val
static void mill_dequeue(struct mill_chan_ *ch, void *val) {
    // 拿chan上sender的第一个choose out从句
    struct mill_clause *cl = mill_cont(
        mill_list_begin(&ch->sender.clauses), struct mill_clause, epitem);
    // chan中valbuf当前无数据可读
    if(!ch->items) {
        // 调用了chdone后肯定没有sender要发送数据了，直接拷走数据即可（chdone追加的）
        if(mill_slow(ch->done)) {
            mill_assert(!cl);
            memcpy(val, ((char*)(ch + 1)) + (ch->bufsz * ch->sz), ch->sz);
            return;
        }
        // 还没有调用chdone，直接从choose out从句中拷走数据，再唤醒因为执行choose out阻塞的协程
        mill_assert(cl);
        memcpy(val, cl->val, ch->sz);
        mill_choose_unblock(cl);
        return;
    }
    // chan中valbuf当前有数据可读
    // - 读取chan中的数据；
    // - 如果对应的choose out从句cl存在，则拷贝其数据到chan valbuf并唤醒执行该从句的协程
    memcpy(val, ((char*)(ch + 1)) + (ch->first * ch->sz), ch->sz);
    ch->first = (ch->first + 1) % ch->bufsz;
    --ch->items;
    if(cl) {
        assert(ch->items < ch->bufsz);
        size_t pos = (ch->first + ch->items) % ch->bufsz;
        memcpy(((char*)(ch + 1)) + (pos * ch->sz) , cl->val, ch->sz);
        ++ch->items;
        mill_choose_unblock(cl);
    }
}

// choose wait从句
int mill_choose_wait_(void) {
    struct mill_choosedata *cd = &mill_running->choosedata;
    struct mill_slist_item *it;
    struct mill_clause *cl;

    // 每个协程都有一个对应的choosedata数据结构
    //    
    // 如果当前有就绪的choose in/out从句，则选择一个并执行
    if(cd->available > 0) {
        // 只有1个就绪的choose从句直接去检查el->ep->type就知道干什么了
        // 如果有多个就绪的choose从句，随机选择一个就绪的从句去执行
        int chosen = cd->available == 1 ? 0 : (int)(random() % (cd->available));

        for(it = mill_slist_begin(&cd->clauses); it; it = mill_slist_next(it)) {
            cl = mill_cont(it, struct mill_clause, chitem);
            if(!cl->available)
                continue;
            if(!chosen)
                break;
            --chosen;
        }
        struct mill_chan_ *ch = mill_getchan(cl->ep);
        // 根据choose从句类型决定是向chan发送数据，还是从chan读取数据
        if(cl->ep->type == MILL_SENDER)
            mill_enqueue(ch, cl->val);
        else
            mill_dequeue(ch, mill_valbuf(cl->cr, ch->sz));
        mill_resume(mill_running, cl->idx);
        return mill_suspend();
    }

    // 如果没有choose in/out从句事件就绪但是有otherwise从句，直接执行otherwise从句
    // - 这里实际上相当于将当前运行的协程重新加入调度队列，然后主动挂起当前协程
    if(cd->othws) {
        mill_resume(mill_running, -1);
        return mill_suspend();
    }

    // 如果指定了deadline从句，为其启动一个定时器，并绑定超时回调
    if(cd->ddline >= 0)
        mill_timer_add(&mill_running->timer, cd->ddline, mill_choose_callback);

    // 其他情况下，将当前协程和被查询的chan进行注册，等到直到有一个choose从句unblock
    for(it = mill_slist_begin(&cd->clauses); it; it = mill_slist_next(it)) {
        cl = mill_cont(it, struct mill_clause, chitem);
        if(mill_slow(cl->ep->refs > 1)) {
            if(cl->ep->tmp == -1)
                cl->ep->tmp =
                    cl->ep->refs == 1 ? 0 : (int)(random() % cl->ep->refs);
            if(cl->ep->tmp) {
                --cl->ep->tmp;
                cl->used = 0;
                continue;
            }
            cl->ep->tmp = -2;
        }
        mill_list_insert(&cl->ep->clauses, &cl->epitem, NULL);
    }
    // 如果有多个协程并发的执行chdone，只可能有一个执行成功，其他的都必须阻塞在下面这行
    return mill_suspend();
}

// 获取正在运行的协程的chan数据存储缓冲区valbuf
void *mill_choose_val_(size_t sz) {
    return mill_valbuf(mill_running, sz);
}

// 向chan中发送数据
void mill_chs_(struct mill_chan_ *ch, void *val, size_t sz,
      const char *current) {
    if(mill_slow(!ch))
        mill_panic("null channel used");
    mill_trace(current, "chs(<%d>)", (int)ch->debug.id);
    mill_choose_init(current);
    mill_running->state = MILL_CHS;
    struct mill_clause cl;
    mill_choose_out_(&cl, ch, val, sz, 0);
    mill_choose_wait_();
}

// 从chan中接收数据
void *mill_chr_(struct mill_chan_ *ch, size_t sz, const char *current) {
    if(mill_slow(!ch))
        mill_panic("null channel used");
    mill_trace(current, "chr(<%d>)", (int)ch->debug.id);
    mill_running->state = MILL_CHR;
    mill_choose_init(current);
    struct mill_clause cl;
    mill_choose_in_(&cl, ch, sz, 0);
    mill_choose_wait_();
    return mill_choose_val_(sz);
}

// chan上的chdone操作
void mill_chdone_(struct mill_chan_ *ch, void *val, size_t sz,
      const char *current) {
    if(mill_slow(!ch))
        mill_panic("null channel used");
    mill_trace(current, "chdone(<%d>)", (int)ch->debug.id);
    if(mill_slow(ch->done))
        mill_panic("chdone on already done-with channel");
    if(mill_slow(ch->sz != sz))
        mill_panic("send of a type not matching the channel");
    /* Panic if there are other senders on the same channel. */
    if(mill_slow(!mill_list_empty(&ch->sender.clauses)))
        mill_panic("send to done-with channel");
    /* Put the channel into done-with mode. */
    ch->done = 1;

    // 在valbuf末尾再追加一个元素，不能chs往valbuf中写因为这样没有receiver的情况下会阻塞
    memcpy(((char*)(ch + 1)) + (ch->bufsz * ch->sz) , val, ch->sz);

    // 追加上述一个多余的元素后，需要唤醒chan上所有等待的receiver
    while(!mill_list_empty(&ch->receiver.clauses)) {
        struct mill_clause *cl = mill_cont(
            mill_list_begin(&ch->receiver.clauses), struct mill_clause, epitem);
        memcpy(mill_valbuf(cl->cr, ch->sz), val, ch->sz);
        mill_choose_unblock(cl);
    }
}
```
```C
//libmill.h
// x86_64平台下协程上下文保存实现
#if defined(__x86_64__)
// 保存当前协程运行时上下文（就是保存处理器硬件上下文到指定内存区域ctx中备用）
//
// 这里使用宏来实现可以避免函数调用堆栈创建、销毁带来的开销，实现更高效地协程切换，
// linux gcc内联汇编，汇编相关参数可以分为“指令部”、“输出部”、“输入部”、“破坏部”，
// 这里将内存变量ctx的值传入寄存器rdx，并将最后rax寄存器的值赋值给变量ret，
// 指令将rax清零，将rbx、r12、rsp、r13、r14、r15、rcx、rdi、rsi依次保存到ctx为起始地址的内存中
#define mill_setjmp_(ctx) ({\
    int ret;\
    asm("lea     LJMPRET%=(%%rip), %%rcx\n\t"\  //==>返回地址(LJMPRET标号处)送入%%rcx
        "xor     %%rax, %%rax\n\t"\
        "mov     %%rbx, (%%rdx)\n\t"\
        "mov     %%rbp, 8(%%rdx)\n\t"\
        "mov     %%r12, 16(%%rdx)\n\t"\
        "mov     %%rsp, 24(%%rdx)\n\t"\
        "mov     %%r13, 32(%%rdx)\n\t"\
        "mov     %%r14, 40(%%rdx)\n\t"\
        "mov     %%r15, 48(%%rdx)\n\t"\
        "mov     %%rcx, 56(%%rdx)\n\t"\         //==>rcx又存储到56(%%rdx)
        "mov     %%rdi, 64(%%rdx)\n\t"\
        "mov     %%rsi, 72(%%rdx)\n\t"\
        "LJMPRET%=:\n\t"\
        : "=a" (ret)\
        : "d" (ctx)\
        : "memory", "rcx", "r8", "r9", "r10", "r11",\
          "xmm0", "xmm1", "xmm2", "xmm3", "xmm4", "xmm5", "xmm6", "xmm7",\
          "xmm8", "xmm9", "xmm10", "xmm11", "xmm12", "xmm13", "xmm14", "xmm15"\
          MILL_CLOBBER\
          );\
    ret;\
})

#if defined(__x86_64__)
// 恢复协程上下文信息到处理器中（从ctx开始的内存区域中加载之前保存的处理器硬件上下文）
//
// 要恢复某个协程cr的运行时，先获取其挂起之前保存的上下文cr->ctx，然后mill_longjmp(ctx)即可，
// 将ctx值加载到rax，采用相对寻址依次加载ctx为起始地址的内存区域中保存的上下文信息到寄存器，
// 最后回复执行
#define mill_longjmp_(ctx) \
    asm("movq   (%%rax), %%rbx\n\t"\
        "movq   8(%%rax), %%rbp\n\t"\
        "movq   16(%%rax), %%r12\n\t"\
        "movq   24(%%rax), %%rdx\n\t"\
        "movq   32(%%rax), %%r13\n\t"\
        "movq   40(%%rax), %%r14\n\t"\
        "mov    %%rdx, %%rsp\n\t"\
        "movq   48(%%rax), %%r15\n\t"\
        "movq   56(%%rax), %%rdx\n\t"\     //==>56(%%rax)中地址为返回地址，送入%%rdx
        "movq   64(%%rax), %%rdi\n\t"\
        "movq   72(%%rax), %%rsi\n\t"\
        "jmp    *%%rdx\n\t"\
        : : "a" (ctx) : "rdx" \
    )
#else
// 非x86_64要借助sigsetjmp\siglongjmp来实现协程上下文切换
#define mill_setjmp_(ctx) \
    sigsetjmp(*ctx, 0)
#define mill_longjmp_(ctx) \
    siglongjmp(*ctx, 1)
#endif

// go()的实现
#define mill_go_(fn) \
    do {\
        void *mill_sp;\
        // 获取当前正在运行的协程上下文，并及时进行保存，因为我们马上要调整栈帧了
        mill_ctx ctx = mill_getctx_();\
        if(!mill_setjmp_(ctx)) {\
            // 为即将新创建的协程分配对应的内存空间，并返回stack部分的当前栈顶（等于栈底）位置
            mill_sp = mill_prologue_(MILL_HERE_);\
            // - 栈帧中的空间分配，对于编译时可确定尺寸的就编译时分配，通过sub <size>,%rsp来实现；
            // - 栈帧中的空间分配，运行时才可以确定的就需要运行时分配，通过alloca(size)来实现；
            // 注意：
            // - gcc在x86_64下alloca的工作主要是sub <size>,%rsp外加一些内存对齐的操作，alloca在当
            //   前栈帧中分配空间并返回空间起始地址，但是不检查栈是否越界；
            // - 另外指针运算是无符号计算，小地址减去大地址的结果会在整个虚拟内存地址空间中滚动；
            // - mill_filler数组的分配是由alloca完成，分配完成后rsp将被调整为mill_sp指向的内存空间；
            // - 新的协程将以mill_sp作为当前栈顶运行，等当前协程恢复上下文并运行时，其根本意识不
            //   到该所谓的mill_filler的存在，因为保存其上下文操作是早于栈调整操作的；
            int mill_anchor[mill_unoptimisable1_];\
            mill_unoptimisable2_ = &mill_anchor;\
            char mill_filler[(char*)&mill_anchor - (char*)(mill_sp)];\
            mill_unoptimisable2_ = &mill_filler;\
            // 在新创建的协程栈空间中调用函数fn
            fn;\
            // fn执行结束后释放占用的协程内存空间，并mill_suspend让出cpu给其他协程
            mill_epilogue_();\
        }\
    } while(0)
```
```C
//cr.h
/ coroutine state
enum mill_state {
    MILL_READY,         //可以被调度
    MILL_MSLEEP,        //mill_suspend挂起等待mill_resume唤醒
    MILL_FDWAIT,        //mill_fdwait_，等待mill_poller_wait或者timer回调唤醒
    MILL_CHR,           //...
    MILL_CHS,           //...
    MILL_CHOOSE         //...
};

/* 
   协程内存布局如下：
   +----------------------------------------------------+--------+---------+
   |                                              stack | valbuf | mill_cr |
   +----------------------------------------------------+--------+---------+
   - mill_cr：包括coroutine的通用信息
   - valbuf：临时存储从chan中接收到的数据
   - stack：标准的c程序栈，栈从高地址向低地址方向增长
*/
struct mill_cr {
    // 协程状态，用于调试目的
    enum mill_state state;

    // 协程如果没有阻塞并且等待执行，会被加入到ready队列中，并设置is_ready=1；
    // 反之，设置is_ready=0，不加入ready队列中
    int is_ready;
    struct mill_slist_item ready;

    // 如果协程需要等待一个截止时间，就需要下面的定时器来实现超时回调
    struct mill_timer timer;

    // 协程在fdwait中等待fd上的io事件就绪，若fd为-1表示当前协程没关注特定fd上的io事件
    int fd;

    // 协程在fdwait中等待fd上的io就绪事件events，用于调试目的
    int events;

    // 协程执行choose语句时要使用的结构体
    struct mill_choosedata choosedata;

    // 协程暂停、恢复执行的时候需要保存、还原其上下文信息
#if defined(__x86_64__)
    uint64_t ctx[10];
#else
    sigjmp_buf ctx;
#endif

    // suspend挂起协程后resume恢复协程执行，resume第二个参数result会被设置到cr->result成员；
    // 其他协程suspend并切换到被resumed的线程时会return mill_running->result
    int result;

    // 如果协程需要的valbuf比预设的mill_valbuf要大的话，那就得从heap中动态分配；
    // 分配的内存空间地址、尺寸记录在这两个成员中
    void *valbuf;
    size_t valbuf_sz;

    // 协程本地存储（有点类似线程local存储）
    void *clsval;

#if defined MILL_VALGRIND
    /* Valgrind stack identifier. */
    int sid;
#endif

    // 调试信息
    struct mill_debug_cr debug;
};

// 主线程对应的假的coroutine
extern struct mill_cr mill_main;

// 记录当前正在运行的协程
extern struct mill_cr *mill_running;

// 挂起当前正在运行的协程，并切换到一个不同的is_ready=1的协程取运行；
// 一旦某个协程resume这个被挂起的协程，resume中传递的参数result将被该suspend函数返回
int mill_suspend(void);

// 调度之前被挂起的协程cr恢复执行，其实只是将其加入ready队列等待被调度而已
void mill_resume(struct mill_cr *cr, int result);

// 返回一个执行协程临时数据区valbuf的指针，返回的数据区容量至少为size bytes
void *mill_valbuf(struct mill_cr *cr, size_t size);

// 子进程中调用，目的是为了停止运行从父进程继承的协程
void mill_cr_postfork(void);
```
```C
//cr.c
// 协程临时数据区valbuf的大小，这里的临时数据区应该合理对齐；
// 如果当前有任何分配的协程栈，就不应该改变这里的尺寸，可能会影响到协程不同内存区域的计算
size_t mill_valbuf_size = 128;

// 主线程这个假协程对应的valbuf
char mill_main_valbuf[128];

volatile int mill_unoptimisable1_ = 1;
volatile void *mill_unoptimisable2_ = NULL;

// 主协程
struct mill_cr mill_main = {0};

// 默认当前正在运行的协程就是mill_run
struct mill_cr *mill_running = &mill_main;

// 等待被调度的就绪协程队列
struct mill_slist mill_ready = {0};

// 返回当前上下文信息
inline mill_ctx mill_getctx_(void) {
#if defined __x86_64__
    return mill_running->ctx;
#else
    return &mill_running->ctx;
#endif
}

// 返回协程临时数据区valbuf的起始地址
static void *mill_getvalbuf(struct mill_cr *cr, size_t size) {
    // 如果请求较小的valbuf则不需要在heap上动态分配
    // 另外要注意主协程没有为其分配栈，但是单独为其分配了valbuf
    if(mill_fast(cr != &mill_main)) {
        if(mill_fast(size <= mill_valbuf_size))
            return (void*)(((char*)cr) - mill_valbuf_size);
    }
    else {
        if(mill_fast(size <= sizeof(mill_main_valbuf)))
            return (void*)mill_main_valbuf;
    }
    // 如果请求较大的valbuf则需要在heap上动态分配，fixme!!!
    if(mill_fast(cr->valbuf && cr->valbuf_sz <= size))
        return cr->valbuf;
    void *ptr = realloc(cr->valbuf, size);
    if(!ptr)
        return NULL;
    cr->valbuf = ptr;
    cr->valbuf_sz = size;
    return cr->valbuf;
}

// 预准备count个协程，并分别初始化其栈尺寸、valbuf、valbuf_sz
void mill_goprepare_(int count, size_t stack_size, size_t val_size) {
    if(mill_slow(mill_hascrs())) {errno = EAGAIN; return;}
    // poller初始化
    mill_poller_init();
    if(mill_slow(errno != 0)) return;
    // 可能的话尅设置val_size稍微大一点以便能合理内存对齐
    mill_valbuf_size = (val_size + 15) & ~((size_t)0xf);
    // 为主协程（假的）分配valbuf
    if(mill_slow(!mill_getvalbuf(&mill_main, mill_valbuf_size))) {
        errno = ENOMEM;
        return;
    }
    // 为协程分配栈（这里分配时计算了stack+valbuf+mill_cr，是一个完整协程的内存空间大小）
    mill_preparestacks(count, stack_size + mill_valbuf_size + sizeof(struct mill_cr));
}

// 挂起当前正在运行的协程，并切换到一个is_ready=1的协程上去执行
// 被挂起的协程需要另一个协程调用resume(cr, result)方法来恢复其执行，恢复后suspend将返回result
int mill_suspend(void) {
    /* Even if process never gets idle, we have to process external events
       once in a while. The external signal may very well be a deadline or
       a user-issued command that cancels the CPU intensive operation. */
    static int counter = 0;
    if(counter >= 103) {
        mill_wait(0);
        counter = 0;
    }
    // 保存当前协程运行时的上下文信息
    if(mill_running) {
        mill_ctx ctx = mill_getctx_();
        if (mill_setjmp_(ctx))
            return mill_running->result;
    }
    while(1) {
        // 寻找一个is_ready=1的可运行的协程并恢复其执行
        if(!mill_slist_empty(&mill_ready)) {
            ++counter;
            struct mill_slist_item *it = mill_slist_pop(&mill_ready);
            mill_running = mill_cont(it, struct mill_cr, ready);
            mill_assert(mill_running->is_ready == 1);
            mill_running->is_ready = 0;
            mill_longjmp_(mill_getctx_());
        }
        // 找不到就要wait，可能要挂起当前协程直到被外部事件唤醒（io事件或者定时器超时）
        mill_wait(1);
        mill_assert(!mill_slist_empty(&mill_ready));
        counter = 0;
    }
}

// 恢复一个协程的运行，每个协程cr都在其内部保存了其运行时上下文信息
// 这里其实只是将其重新加入就绪队列等待被调度而已
inline void mill_resume(struct mill_cr *cr, int result) {
    mill_assert(!cr->is_ready);
    cr->result = result;
    cr->state = MILL_READY;
    cr->is_ready = 1;
    mill_slist_push_back(&mill_ready, &cr->ready);
}

/* mill_prologue_() and mill_epilogue_() live in the same scope with
   libdill's stack-switching black magic. As such, they are extremely
   fragile. Therefore, the optimiser is prohibited to touch them. */
#if defined __clang__
#define dill_noopt __attribute__((optnone))
#elif defined __GNUC__
#define dill_noopt __attribute__((optimize("O0")))
#else
#error "Unsupported compiler!"
#endif

// go()开始部分，启动一个新的协程，返回指向栈顶的指针
__attribute__((noinline)) dill_noopt 
void *mill_prologue_(const char *created) {
    ......
    // 分配并初始化新的stack
#if defined MILL_VALGRIND
    ......
#else
    // 先从cache中取，取不到动态分配
    struct mill_cr *cr = ((struct mill_cr*)mill_allocstack(NULL)) - 1;
#endif
    mill_register_cr(&cr->debug, created);
    cr->is_ready = 0;
    cr->valbuf = NULL;
    cr->valbuf_sz = 0;
    cr->clsval = NULL;
    cr->timer.expiry = -1;
    cr->fd = -1;
    cr->events = 0;
    mill_trace(created, "[%d]=go()", (int)cr->debug.id);
    // 挂起父协程并调度新创建的协程来运行
    mill_resume(mill_running, 0);    
    mill_running = cr;
    // 计算返回valbuf栈顶尺寸
    return (void*)(((char*)cr) - mill_valbuf_size);
}

// go结束部分，协程结束的时候执行清零动作
__attribute__((noinline)) dill_noopt
void mill_epilogue_(void) {
    mill_trace(NULL, "go() done");
    mill_unregister_cr(&mill_running->debug);
    if(mill_running->valbuf)
        free(mill_running->valbuf);
#if defined MILL_VALGRIND
    ......
#endif
    mill_freestack(mill_running + 1);
    mill_running = NULL;
    // 考虑到这里没有运行中的协程了，所以mill_suspend永远不会返回了
    mill_suspend();
}

void mill_yield_(const char *current) {
    mill_trace(current, "yield()");
    mill_set_current(&mill_running->debug, current);
    // 这里看起来有点可疑，但是没问题，我们可以在挂起一个协程之前就resume它来执行；
    // 这样做的目的是为了suspend之后能够使该协程重新获得被调度执行的机会
    mill_resume(mill_running, 0);
    mill_suspend();
}

// 返回valbuf起始地址
void *mill_valbuf(struct mill_cr *cr, size_t size) {
    void *ptr = mill_getvalbuf(cr, size);
    if(!ptr)
        mill_panic("not enough memory to receive from channel");
    return ptr;
}

// 返回协程本地存储指针
void *mill_cls_(void) {
    return mill_running->clsval;
}

// 设置协程本地存储操作
void mill_setcls_(void *val) {
    mill_running->clsval = val;
}

// fork之后子进程清空就绪协程队列列表
void mill_cr_postfork(void) {
    /* Drop all coroutines in the "ready to execute" list. */
    mill_slist_init(&mill_ready);
}
```