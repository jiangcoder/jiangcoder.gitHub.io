## 一，模板简述
template大致分成setting和mappings两部分：
索引可使用预定义的模板进行创建,这个模板称作Index templates。模板设置包括settings和mappings，通过模式匹配的方式使得多个索引重用一个模板。 
1. settings主要作用于index的一些相关配置信息，如分片数、副本数，tranlog同步条件、refresh等。
 
2. mappings主要是一些说明信息，大致又分为_all、_source、prpperties这三部分：
 
     (1) _all：主要指的是AllField字段，我们可以将一个或多个都包含进来，在进行检索时无需指定字段的情况下检索多个字段。设置“_all" : {"enabled" : true}
 
     (2) _source：主要指的是SourceField字段，Source可以理解为ES除了将数据保存在索引文件中，另外还有一份源数据。_source字段在我们进行检索时相当重要，如果在{"enabled" : false}情况下默认检索只会返回ID， 你需要通过Fields字段去到索引中去取数据，效率不是很高。但是enabled设置为true时，索引会比较大，这时可以通过Compress进行压缩和inclueds、excludes来在字段级别上进行一些限制，自定义哪些字段允许存储。
 
     (3) properties：这是最重要的步骤，主要针对索引结构和字段级别上的一些设置。
3.咱们通常在elasticsearch中 post mapping信息，每重新创建索引便到设置mapping，分片，副本信息。非常繁琐。强烈建议大家通过设置template方式设置索引信息。设置索引名，通过正则匹配的方式匹配到相应的模板。ps:直接修改mapping的优先级>索引template。索引匹配了多个template，当属性等配置出现不一致的，以order的最大值为准，order默认值为0
## 二，创建模板
例如：
 
```java
{
  "template": "pmall*",
  "settings": {
    "index.number_of_shards": 1,
    "number_of_replicas": 4,
    "similarity": {
      "IgnoreTFSimilarity": {
        "type": "IgoreTFSimilarity"
      }
    }
  },
  "mappings": {
    "_default_": {
      "_source": {
        "enabled": false
      }
    },
    "commodity": {
      "properties": {
        "sold": {
          "type": "long"
        },
        "online_time": {
          "type": "long"
        },
        "price": {
          "type": "long"
        },
        "publish_time": {
          "type": "long"
        },
        "id": {
          "type": "long"
        },
        "catecode": {
          "type": "integer"
        },
        "title": {
          "search_analyzer": "ikSmart",
          "similarity": "IgnoreTFSimilarity",
          "analyzer": "ik",
          "type": "text"
        },
        "content": {
          "index": false,
          "store": true,
          "type": "keyword"
        },
        "status": {
          "type": "integer"
        }
      }
    }
  }
}
```

 

## 三，删除模板

```java

DELETE /_template/template_1
```




## 四，查看模板：
 
```java
GET /_template/template_1
```


也可以通过模糊匹配得到多个模板信息
```java
GET /_template/temp* 
```


可以批量查看模板
```java
GET /_template/template_1,template_2
```


验证模板是否存在：
 
```java
HEAD _template/template_1
```


## 五：多个模板同时匹配，以order顺序倒排，order越大，优先级越高
 
```java

 
PUT /_template/template_1
{
    "template" : "*",
    "order" : 0,
    "settings" : {
        "number_of_shards" : 1
    },
    "mappings" : {
        "type1" : {
            "_source" : { "enabled" : false }
        }
    }
}

PUT /_template/template_2
{
    "template" : "te*",
    "order" : 1,
    "settings" : {
        "number_of_shards" : 1
    },
    "mappings" : {
        "type1" : {
            "_source" : { "enabled" : true }
        }
    }
}
```


 
## 六，模板版本号
 
模板可以选择添加版本号，这可以是任何整数值，以便简化外部系统的模板管理。版本字段是完全可选的，它仅用于模板的外部管理。要取消设置版本，只需替换模板即可

 
创建模板：

```java
PUT /_template/template_1
{
    "template" : "*",
    "order" : 0,
    "settings" : {
        "number_of_shards" : 1
    },
    "version": 123
}
```


查看模板版本号：

```java
GET /_template/template_1?filter_path=*.version
```



响应如下：

```java
{
  "template_1" : {
    "version" : 123
  }
}
```


## 七，参考：
[url=https://www.elastic.co/guide/en/elasticsearch/reference/5.4/indices-templates.html]indices-templates[/url]