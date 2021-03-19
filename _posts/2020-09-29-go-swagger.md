---
layout:         post
title:          go-swagger
subtitle:       Distributed go swagger use
date:           2020-09-29 15:48:00
author:         nomadli
header-img:     ../img/bg-coffee.jpeg
catalog:        true
tags:
        - other
---

* content
{:toc}

## [地址](https://github.com/go-swagger/go-swagger)
- go build cmd/swagger
- swagger generate spec -o ./swagger.json 从当前目录源码注释生成项目文档
- swagger validate ./swagger.json 验证项目文档
- swagger generate server -f ./swagger.json -A name 从文档生成服务器代码
- swagger generate client -f ./swagger.json -A name 从文档生成客户端代码
- swagger generate model --spec={spec} 生成数据模型
- swagger serve ./swagger.json 启动文档及测试服务

## 注释
- 项目注释
```go
// Terms Of Service:
//
// 反诈骗服务
//
//     Schemes: http, https
//     Host: http://xxx
//     BasePath: /api/v1
//     Version: 0.0.1
//     Contact: nomadli<dzym79@qq.com>
//     License: © 2021 nomadli
//
//     Consumes:
//        - application/json
//        - application/xml
//
//     Produces:
//        - application/json
//
// swagger:meta
package main //这里要紧挨着
```
- 将下列注释给你解开函数
```go
// swagger:operation <GET|PUT|DELETE|POST|PATCH> /x/x/{id} [tag] [operation id]
// ---
// summary: 标题
// description: 描述
// parameters:
// - name: 参数名
//   in: <header|body|query|path>     #参数位置 query=url中的参数?后面   path包含在url里的部分
//   description: 参数描述
//   type: string|integer 类型
//   format: int64|int32 子类型
//   default: 默认值
//   required: true 是否必须
// responses:
//   200:
//      description: the result
//      schema:
//          type: array
//          items:
//              type: object
//              required:
//                  - id
//              properties:
//                  id:
//                      type: integer
//                      format: int64
//                      readOnly: true
//                  description:
//                      type: string
//                      minLength: 1
//                  completed:
//                      type: boolean
//   default:
//      description: the err result
//      schema:
//          type: object
//              required:
//                  - code
//              properties:
//                  code:
//                      type: integer
//                      format: int64
//                  message:
//                      type: string
//或者
//  200: repoResp
//  400: badReq
//  409: conflict
//  500: internal
```
