---
layout:         post
title:          WebAssembly
subtitle:       WebAssembly
date:           2022-10-10 11:06:00
author:         nomadli
header-img:     ../img/bg-coffee.jpeg
catalog:        true
tags:
        - other
---

* content
{:toc}

## 环境
- [运行时](https://github.com/bytecodealliance/wasmtime)
    - curl https://wasmtime.dev/install.sh -sSf | bash
- [c标准库实现](https://github.com/WebAssembly/wasi-libc)
- [wasi-sysroot](https://github.com/WebAssembly/wasi-sdk)
- [命令行编译工具](https://github.com/WebAssembly/wabt)
    - wasm2wat 反汇编wasm程序,转换为人类可读的格式
    - wat2wasm
    - wasm-strip x.wasm
- [wasm-opt wasm文件优化工具](https://github.com/WebAssembly/binaryen)
    - wasm-opt -O3 -o outx.wasm inx.wasm
- [wasm信息提取工具](https://github.com/rustwasm/twiggy)
- [应用广泛的图片压缩](https://github.com/GoogleChromeLabs/squoosh)
```shell
# 不使用任何标准库
# C to LLVM IR  x.ll
clang --target=wasm32 -emit-llvm -c -S x.c
    # --target=wasm32  目标WebAssembly格式
    # -emit-llvm       输出IR中间码而非机器码
    # -c               只编译不链接
    # -S               输出可读IR码,而非二进制

# LLVM IR to object files x.o
llc -march=wasm32 -filetype=obj x.ll

# Linking
wasm-ld  --no-entry --export-all -o x.wasm x.o
    # --no-entry        无入库函数
    # --export-all      导出所有符号

# run
wasmtime x.wasm --invoke function_name param1 param2 ...

#或
#llvm-ar llvm-nm llvm-strip llvm-ranlib
#LDFLAGS=${LDFLAGS} -Wl,--no-threads
clang --target=wasm32 -O3 -s -flto -nostdlib -Wl,--no-entry -Wl,--export-all -Wl,--lto-O3 -o x.wasm x.c
    # --target=wasm32       目标平台
    # -O3                   编译优化级别
    # -flto                 添加原数据
    # -nostdlib             不链接stdlib 不需要 sysroot
    # -Wl,--no-entry        无入口函数
    # -Wl,--export-all      符号全部导出
    # -Wl,--lto-O3          链接优化级别

# 使用wasi标准C库
clang --target=wasm32-unknown-wasi --sysroot /x/wasi-libc -O3 -s -o x.wasm x.c

# C++不支持异常 CXX_FLAGS=-fno-exceptions

fetch('./x.wasm').then(function(response) {
        return response.arrayBuffer()
    }).then(function(bytes) {
            return WebAssembly.instantiate(bytes)
        }).then(function(results) {
            const add = results.instance.exports.add;
            var result = add(1, 2);
        }).catch(console.error);
```
