---
layout:         post
title:          Elasticsearch
subtitle:       Elasticsearch
date:           2021-04-28 12:02:00
author:         nomadli
header-img:     ../img/bg-coffee.jpeg
catalog:        true
tags:
        - other
---

* content
{:toc}

## 基本信息
- [github](https://github.com/elastic/elasticsearch)
- 基于Lucene全文搜索引擎 
- 索引在ES中可以表示服务本身、服务中的不同数据库(_index)、数据库中的不同表|数据类型(_type)、表中的行(_id)、行中的列(文档)
- 数据内部类型有string, number, Boolean, date
- 路径一般为/_index/_type/_id
- /_index/_type/_id/_create 创建
- 搜索时为路径一般为/_index/_type/_search?q=key:value
    - match             按单词匹配, 不care顺序, 非全部匹配
    - match_phrase      查找子句
        - slop          几次变换可以匹配 如 ABC 查找AC 1次变换可以匹配, 查找CA需要3(A|C->CA->C A) 
    - match_phrase_prefix在查询时类似match_phrase,最后一个词作为前缀查询,用来搜索提示
        - slop          与match_phrase一致
        - max_expansions限制返回条数
    - match_all         所有数据
    - multi_match       多字段使用相同的查询模式
        - fields        哪些字段可以正则
        - type          评分算法
            - most_fields   采用类似bool的评分
            - best_fields   采用类似dis_max的评分方式
            - cross_fields  采用匹配的最低分
        - operator      and 必须所有字段都包含所有的词, 与cross_fields结合只要所有次都出现过一次(不一定在一个字段中)
    - range             范围匹配 gt >    gte >=    lt <      lte <=
    - term              精确匹配,如果字段是数组精确匹配其中一个值
    - terms             多条件精确匹配
    - exists            存在某值
    - missing           不存在某值
    - bool              组合多条其它语句, 可嵌套, 可被其它语句嵌套, 采用子句的算术平均
        - must          条件必须满足。
        - must_not      条件必须不满足。
        - should        满足其中一条
    - dis_max           组合查询语句, 评分算法与bool不同,采用子句的最高评分
        - tie_breaker   将其他语句的评分与tie_breaker[0-1]相乘+最高,归一
    - boosting          权重组合查询, 对每个查询模块做不同的权重合并排序
        - positive      实际查询模块
        - negative      权重变化模块,将positive结果中符合的结果权重做negative_boost乘数
            - negative_boost
    - filter            必须满足条件但不评分
    - filtered          包装filter
    - constant_score    将其它查询作为子查询,使子查询都不评分
    - prefix            查询词前缀
    - wildcard          通配符匹配 ?*
    - regexp            正则匹配词
    - indices_boost     将某个索引下的文档分数*乘数
    - sort              指定排序字段,暗含不进行评分,可以多级排序,根据数组中条件出现的顺序,也可使用评分(_score)次级排序
    - track_scores      强制要计算评分
    - from              分页开始位置
    - size              分页limit条
    - preference        _primary, _primary_first, _local, _only_node:xyz, _prefer_node:xyz, _shards:2,3
        - 设置搜索偏好, 例如日志时间戳排序, 可能很多日志时间戳一致,不指定顺序会导致结果排序随机
    - timeout           设置超时时间, 可以接受部分结果
    - scroll            采用游标分页, 返回最大条数为 size*分片数
        - scroll_id     上次返回的游标ID 还是会有相同时间导致的排序混乱
    - aggs              聚合信息
        - all_字段名称    对字段所有内容按条件统计总数
        - avg_字段名称    对字段所有内容按条件算平均值
    - rescore           对结果使用另一种查询做评分排序
        - window_size   对每个分片的前多少条做重新评分
    - _source           指定返回字段
- 修改 /_index/_type/_id?version=5[&version_type=external]
- 部分更新 /_index/_type/_id/_update
- mget 多条件查询
- _bulk 多条 create、index、update、delete提交
- /_index/_type/_validate/query 检查查询语句错误
- /_index/_type/query?explain  返回评分标准
- /_index/_type/_id/_explain 返回此文档没有匹配或匹配的原因
- format=yaml 以yaml格式给出结果
```JSON
    //搜索也可以使用JSON
    /库/表|类型/_search
    {
        "query" : {
            "match" : {
                "about" : "rock climbing",  #分词查找
                "operator": "and",          #结果包括两个词
                "minimum_should_match": "75%", #这里是控制有多少词被匹配到
                "analyzer": "standard"      #搜索用的分析器
            }
        },
        "highlight": {          #高亮哪个字段
            "fields" : {
                "about" : {}
            }
        },
        "range" : {
            "timestamp" : {
                "gt" : "now-1h"         #实时计算一小时前的
            }
        },
        "range" : {
            "timestamp" : {
                "gt" : "2014-01-01 00:00:00",
                "lt" : "2014-01-01 00:00:00||+1M"   #相对时间
            }
        },
        "bool": {
            "should": [
              { "match": { "title": "brown", "boost": 2}}, #boost 评分乘数, 结果归一化
              { "match": { "title": "fox", "boost": 0.5}}
            ],
            "minimum_should_match": 1,              #should 必须有一个个匹配
        },
        "multi_match": {
            "query":  "Quick brown fox",
            "fields": [ "*_title", "chapter_title^2" ] #boost=2
        },
        "sort": [
            {
                "sort": {                   #用数组排序
                    "dates": {
                        "order": "asc",
                        "mode":  "min"      #min max avg sum 使用数组中哪个值进行比较
                    }
                }
            },
            {"dtate": { "order": "desc" }},
            {"_score": { "order": "desc" }}
        ]
    }
    GET /_index/_analyze              #查看分词结果
    {
      "field": "productID",
      "text": "XHDK-A-1293-#fJ3"
    }
    GET /_index/_type/_validate/query?explain #查看分词结果
    ?search_type=dfs_query_then_fetch   #强制在所有分片全局计算分数

    GET /docs_2014_*/_search 
    {
      "indices_boost": { 
        "docs_2014_10": 3,
        "docs_2014_09": 2
      }
    }
```
- 行即文档采用JSON

## 分片
- 分片是用来存储数据、分布在不同的机器、主分片默认分5片,每个分片可以保存最多Integer.MAX_VALUE - 128条数据
- 每个分片底层对应一个Lucene,在逻辑上是一个独立的搜索引擎
- 主分片数量不能修改
- ES将每个引擎的结果汇总返回

## 索引 _index
- config/elasticsearch.yml->action.auto_create_index: false 禁止自动ID
- _index并不会在索引列表里
- DELETE /one,two  DELETE /index_* DELETE /_all DELETE /*   action.destructive_requires_name: true 禁止模糊
- analyzer: "name" 指定使用特殊的分析器
- POST _reindex 重新在新的_index中索引当前_index, 当修改_type后或修改_index后,
- PUT /my_index_v1/_alias/my_index 设置_index别名,为了重新索引后不修改应用程序,使用别名
- GET /*/_alias/my_index 查询别名指向哪些_index
- GET /my_index_v1/_alias/* 查询当前_index有哪些别名
```JSON
    PUT /blogs
    {
        "settings" : {                                  #生成索引名字为blogs
            "number_of_shards" : 3,                     #分3片主索引
            "number_of_replicas" : 1,                   #每个主索引分片有一个备份
            "refresh_interval": "30s",                  #每30秒刷盘一次, 默认1秒,新文档30秒内无法搜索,-1关闭自动刷新
            "index.translog.durability": "async",       #操作日志异步刷新, request写请求同步刷日志默认值
            "index.translog.sync_interval": "5s",       #操作日志5秒刷新一次
            "default": "standard",                      #默认字段分析器
            "default_search": "standard",               #默认搜索分析器
            "analysis": {                               #定义分析器,主要用来 字符过滤、分词、词单元过滤
                "char_filter": {                        #字符过滤器
                    "&_to_and": {                       #将&替换为and
                        "type":       "mapping",
                        "mappings": [ "&=> and "]
                    }
                },
                "filter": {                             #词单元过滤器
                    "my_stopwords": {
                        "type":       "stop",
                        "stopwords": [ "the", "a" ]
                    }
                },
                "filter": {
                    "my_shingle_filter": {
                        "type":             "shingle",
                        "min_shingle_size": 2,          #将句子中最少几个连续的词也作为索引
                        "max_shingle_size": 2,          #将句子中最多几个连续的词也作为索引
                        "output_unigrams":  false       #单个词独立建索引
                    }
                },
                "filter": {
                    "autocomplete_filter": {            #nomadli-> n no nom noma nomad
                        "type":     "edge_ngram",       #定义一个n-gram过滤器,直接生成词前缀索引而不是运行时匹配
                        "min_gram": 1,                  #生成最小前缀为1个字符,如对nomadli->n
                        "max_gram": 5                   #生成最大前缀为5个字符,如对nomadli->nomad
                    }
                },
                "analyzer": {
                    "my_analyzer": {                    #name es_std
                        "type":         "custom",       #自定义解析器
                        "char_filter":  [ "html_strip", "&_to_and" ],   #使用哪些字符过滤器
                        "tokenizer":    "standard",     #分词器器
                        "filter":       [ "lowercase", "my_stopwords" ] #使用哪些词单元过滤
                    }
                },
                "analyzer": {                           #分词器
                    "es_std": {                         #name es_std
                        "type":      "standard",        #standard 是系统自带的分析器,在此基础上修改某些东西
                        "stopwords": "_spanish_"        #禁止分析预定义西班牙语词汇, 也可以是词列表
                    }
                }
            }
        }
    }
    PUT /blogs/_settings
    {
        "number_of_replicas" : 2        #修改备份分片数量
    }
    POST /_aliases                      #原子操作批量别名
    {
        "actions": [
            { "remove": { "index": "my_index_v1", "alias": "my_index" }},
            { "add":    { "index": "my_index_v2", "alias": "my_index" }}
        ]
    }
    POST /_index/_flush                 #刷新索引段,生成新段
    POST /_flush?wait_for_ongoing       #刷新所有的索引
    POST /_index/_optimize?max_num_segments=1   #将索引段合并为一个
```

## 类型 _type
- _type 添加了索引但并不会存储
- JSON中在不同层次出现的相同Key type要相同, 否则Lucene会错
```JSON
    PUT /_index/_mapping/_type                      #设置类型
    {
        "dynamic": "strict",                        #未设置字段默认抛异常,true动态添加字段,false忽略该字段
        "date_detection": false,                    #关闭新字段日期自动识别,日期作为string
        "dynamic_templates": [
            { 
                "es": {                             #es自动识别新字段模板
                    "match":              "*_es",   #字段以_es结尾 path_match关键字需要匹配全路径a.b.*.*_es
                    "match_mapping_type": "string", #JSON数据类型为string
                    "mapping": {
                        "type":           "string",#作为string类型
                        "analyzer":       "spanish"#使用spanish分析器
                    }
                }
            },
            {
                "en": {                             #en自动识别新字段模板
                    "match":              "*",      #匹配所有字段
                    "match_mapping_type": "string", #JSON数据类型为string
                    "mapping": {
                        "type":           "string",#作为string类型
                        "analyzer":       "english"#使用english分析器
                    }
                }
            }
        ],
        "_default_": {
            "_all": { "enabled":  false }           #设置所有字段默认设置
        },
        "properties": {                             #设置常见字段的类型及解析器
            "title": {
                "type":      "string",
                "copy_to":   "other_profertie",     #将内容复制给某字段,比如姓与名复制给姓名,fields不能使用
                "position_increment_gap": 100,      #数组中值分词后将按index*100作为词在文档中出现位置
                "include_in_all": true,             #是否包含在_all的搜索中
                "analyzer":  "my_analyzer",         #not_analyzed表示此值为精确值不解析 no不能被搜索
                "index_analyzer": "standard",       #建索引时用的分析器
                "search_analyzer": "standard",      #搜索用的分析器,
                "index_options": "docs",            #禁用词频统计
                "norms": { "enabled": false },      #禁用文本长度评分, 词只要出现分数就一样,不管文章有多长
                "disable_coord": true,              #关闭协调因子评分, 类似(出现词分数和)/总查询词个数=分数
                "fields": {                         #对title进行多次分词、近义词、不同时态、拼写错误搜索、全词匹配等
                    "raw": {                        #用在字段上产生两种索引 title.raw 代表不分词比较(字符串比较)
                        "type": "string",
                        "index": "not_analyzed"
                    }
                }
            },
            "stash":  {
                "type":     "object",               #此字段为子obj, 其子字段自动添加
                "dynamic":  true 
            },
            "_all": {
                "analyzer": "whitespace",           #使用_all来查询时使用的分词器
                "enabled": false                    #禁止使用_all字段来搜索, 将source整个体作为字符串
            }
        }
    }
    PUT /_index
    {
        "mappings": {
            "_type": {                             #_type指实际名字,
                "_source": {
                    "enabled":  false              #不保存原始数据
                }
            }
        }
    }
```

## 锁
- PUT /fs/lock/global/_create | DELETE /fs/lock/global 全局锁
