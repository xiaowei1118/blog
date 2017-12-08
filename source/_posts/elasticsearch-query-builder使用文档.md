---
title: elasticSearch-query-builder使用文档
date: 2016-03-17 10:12:46
tags: [elasticSearch,Java]
categories: coding
---

### 前言
在这里，我想向大家推荐一个我自己开发的项目，也就是`elasticsearch-query-builder`,这个项目目前在github
上已经开源，有兴趣的朋友可以去fork或者star，你的star就是对我最大的鼓励。同时，本项目长期维护和更新，
我也接受并且很高兴有小伙伴向本项目pull request，或者协同开发，有兴趣的同学可以给我发邮件。
 
 `elasticsearch-query-builder`是一个非常方便构造`elasticsearch`(后面简称ES) DSL 查询语句的工具包，在`elasticsearch-query-builder`
 中，我尝试基于配置化的操作去构建ES的查询语句，并且接受外界传入参数，这极大的减少了在Java代码中构建ES
 查询语句的工作，并同时减少了代码量，使代码更加直观和清晰。基于使ES中DSL构造语句和Java代码分离的思想，
 `elasticsearch-query-builder`诞生了。去[Github](https://github.com/xiaowei1118/elasticsearch-query-builder)Fork!
 
 ### 构建
 `elasticsearch-query-builder`工程一般作为jar包为别的工程提供使用，当然，如果需要基于本项目做二次开发，这都需要将
 Github上克隆本项目到本地
 ```
 git clone https://github.com/xiaowei1118/elasticsearch-query-builder.git
 ```
 在将本项目克隆到本地后，执行`mvn package` 将本项目打成jar包，或者直接将本项目作为你们自己maven项目的module项目。
 
 
 ### elasticsearch-query-builder使用详细说明
 elastcisearch-query-builder接受配置文件(特定json格式)或者json格式的字符串配置，配置格式如下：
 ``` json
{
  "index": "user_portrait",
  "type": "docs",
  "from": "${from}",
  "size": "10",
  "include_source": ["name","age"],  //需要哪些字段
  "exclude_source": ["sex"],  //排除哪些字段
  "query_type": "terms_level_query",
  "terms_level_query": {
    "terms_level_type": "term_query",
    "term_query": {
      "value": "${value}",
      "key": "key",
      "boost": 2
    }
  },
  "aggregations": [
    {
      "aggregation_type": "terms",
      "name": "",
      "field": "field",
      "sub_aggregations": {
        "aggregation_type": "terms",
        "name": "sub",
        "field": "field",
        "size": "${size.value}",
        "sort": "asc",
        "sort_by": "_count"
      }
    }
  ],
  "highlight":{
      "fields": [
            {
              "field": "content",
              "number_of_fragment": 2,
              "no_match_size": 150
            }
       ],
      "pre_tags":["<em>"],
      "post_tags":["</em>"]
  },
  "sort": [
    "_score",
    {
      "field": "age",
      "order": "asc"
    }
  ]
}
```
#### 参数说明
##### # index
index表示elasticSearch中的索引或者别名。
##### # type
type表示elasticSearch索引或者别名下的type。
##### # from
from表示检索文档时的偏移量，相当于关系型数据库里的offset。
##### # include_source
include_source 搜索结果中包含某些字段，格式为json数组，``"include_source": ["name","age"]``。
#### # exclude_source
exclude_source 搜索结果中排除某些字段，格式为json数组，``"exclude_source":["sex"]``。
##### # query_type 
query_type表示查询类型，支持三种类型`terms_level_query`,`text_level_query`,`bool_level_query`,并且这三种类型
不可以一起使用。
+ `terms_level_query` 操作的精确字段是存储在反转索引中的。这些查询通常用于结构化数据, 
如数字、日期和枚举, 而不是全文字段,包含term_query,terms_query,range_query,exists_query 等类型。

+ `text_level_query` 查询通常用于在完整文本字段 (如电子邮件正文) 上运行全文查询。他们了解如何分析被查询的字段, 
并在执行之前将每个字段的分析器 (或 search_analyzer) 应用到查询字符串。<br/>
包含 match_query,multi_match_query,query_string,simple_query_string 等类型。

+  `bool_query` 与其他查询的布尔组合匹配的文档匹配的查询。bool 查询映射到 Lucene BooleanQuery。它是使用一个或多个布尔子句生成的, 每个子句都有一个类型化的实例。 
布尔查询的查询值包括: must,filter,should,must_not. 想要了解这几个类型的差异，可以查阅elasticSearch的相关文档 在每个布尔查询的查询类型值中, 
可以包含terms_level_query 和 text_level_query中任意的查询类型，如此便可以构造非常复杂的查询情况。

##### # terms_level_query查询类型
##### terms_level_type
terms_level查询类型，支持`term_query`,`terms_query`,`range_query`,`exists_query`查询。
+ `term_query` 
  + key 表示elasticSearch中需要查询的字段
  + value 表示要查询的值
  + boost 占搜索中的权重
```
"term_query": {
      "value": "",
      "key": "",
      "boost": 2
    }
```
+ `terms_query`
   + key,value,boost解释同`term_query`。
   + value 可以传入多个，以逗号隔开，如"[1,2]"。
```
 "terms_query": {
      "value": "[1,2]", //数组
      "key": ""
    },
```
+ `range_query`，给定的查询条件使一个范围
  + key 表示elasticSearch中需要查询的字段
  + range 表示要搜索的值范围，格式如"[a,b]",表示范围在a、b之间，a、b可以缺省，a缺省则表示没有下限，
  b缺省则表示没有上限，但ab不可以同时为空。a，b可以为时间或者数值。
  + boost 占搜索中的权重
  + format 如果范围使时间的化，format定义时间格式。
  + include_lower 布尔值，是否包含下限。
  + include_upper 布尔值，是否包含上限。
```
"range_query": {
      "key": "",
      "range": "", //[,]
      "boost": "",
      "format": "",
      "include_lower": true,
      "include_upper": false
}
```

+ `exists_query`,存在查询，查找字段不存在的文档。
   + key elasticSearch字段。
```
"exists_query": {
      "key": ""
}
```  

##### # text_level_query查询类型
##### text_query_type
text_level_query查询类型，支持match_query,multi_match_query,query_string,simple_query_string等。
+ match_query,普通的文本匹配查询。
  + key 供文本匹配的ES字段
  + value 需要搜索的文本关键字，会分词。
  + zero_terms_query 决定是否使用停词。all表示不使用停词，默认使none。
```
"match_query": {
  "key": "",
  "value": "this is a test",
  "zero_terms_query": "none" 
} 
```  
+ multi_match_query 在多个字段中进行文本匹配
  + value 需要搜索的文本关键字，会分词。
  + fields ES中的字段，可以多个，用逗号隔开，在字段旁边使用^表示该字段的权重，如"a^3,b"。
  + type 匹配类型，支持best_fields(默认),most_fields,cross_fields,phrase,phrase_prefix。
```
 "multi_match_query": {
      "value": "",
      "fields": "a^3,b", 
      "type": "best_fields" //most_fields,cross_fields,phrase,phrase_prefix
    },
```

+ query_string 字符串文本匹配。
    + value 需要搜索的文本关键字，会分词。
    + fields ES中的字段，格式为数组，如"[a,b]"
```
 "query_string": {
      "value": "",
      "fields": ""//数组
    }
``` 
+ simple_query_string 简单字符串匹配
    + value fields 同 query_string
    + default_operate 匹配逻辑，值为`and` 或者 `or`。
```
"simple_query_string": {
      "value": "",
      "fields": "", //数组
      "default_operate": "and"
    }
```
+ match_all_query 匹配所有文档

##### # bool_query 布尔查询
bool_query一般用于构建复杂的查询，而这正是`elasticsearch-query-builder`最拿手的地方。
##### bool_type 
+ must 
+ should
+ filter
+ must_not

##### # aggregation 聚合


  

 