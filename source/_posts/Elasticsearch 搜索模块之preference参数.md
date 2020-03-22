title: Elasticsearch 搜索模块之preference参数
toc: true
categories: elasticsearch
tags:
  - elasticsearch
thumbnail: /logo/elasticsearch.jpg
---
##一，preference简述

elasticsearch可以使用preference参数来指定分片查询的优先级，即我们可以通过该参数来控制搜索时的索引数据分片。

如不设置该参数：在所有有效的主分片以及副本间轮询。

具体可看下：OperationRouting.java类
```java
public ShardIterator activeInitializingShardsRandomIt() {
    return activeInitializingShardsIt(shuffler.nextSeed());
}  
```

自增，以实现shard间轮询操作
```java
public int nextSeed() {
    return seed.getAndIncrement();
 }
``` 

```java
public ShardIterator activeInitializingShardsIt(int seed) {
    if (allInitializingShards.isEmpty()) {
        return new PlainShardIterator(shardId, shuffler.shuffle(activeShards, seed));
    }
    ArrayList<ShardRouting> ordered = new ArrayList<>(activeShards.size() + allInitializingShards.size());
    ordered.addAll(shuffler.shuffle(activeShards, seed));
    ordered.addAll(allInitializingShards);
    return new PlainShardIterator(shardId, ordered);
}
```

```java
private ShardIterator preferenceActiveShardIterator(IndexShardRoutingTable indexShard, String localNodeId, DiscoveryNodes nodes, @Nullable String preference) {
        if (preference == null || preference.isEmpty()) {
            if (awarenessAttributes.length == 0) {
                return indexShard.activeInitializingShardsRandomIt();
            } else {
                return indexShard.preferAttributesActiveInitializingShardsIt(awarenessAttributes, nodes);
            }
        }
        if (preference.charAt(0) == '_') {
            Preference preferenceType = Preference.parse(preference);
            if (preferenceType == Preference.SHARDS) {
                // starts with _shards, so execute on specific ones
                int index = preference.indexOf('|');

                String shards;
                if (index == -1) {
                    shards = preference.substring(Preference.SHARDS.type().length() + 1);
                } else {
                    shards = preference.substring(Preference.SHARDS.type().length() + 1, index);
                }
                String ids = Strings.splitStringByCommaToArray(shards);
                boolean found = false;
                for (String id : ids) {
                    if (Integer.parseInt(id) == indexShard.shardId().id()) {
                        found = true;
                        break;
                    }
                }
                if (!found) {
                    return null;
                }
                // no more preference
                if (index == -1 || index == preference.length() - 1) {
                    if (awarenessAttributes.length == 0) {
                        return indexShard.activeInitializingShardsRandomIt();
                    } else {
                        return indexShard.preferAttributesActiveInitializingShardsIt(awarenessAttributes, nodes);
                    }
                } else {
                    // update the preference and continue
                    preference = preference.substring(index + 1);
                }
            }
            preferenceType = Preference.parse(preference);
            switch (preferenceType) {
                case PREFER_NODES:
                    final Set<String> nodesIds =
                            Arrays.stream(
                                    preference.substring(Preference.PREFER_NODES.type().length() + 1).split(",")
                            ).collect(Collectors.toSet());
                    return indexShard.preferNodeActiveInitializingShardsIt(nodesIds);
                case LOCAL:
                    return indexShard.preferNodeActiveInitializingShardsIt(Collections.singleton(localNodeId));
                case PRIMARY:
                    return indexShard.primaryActiveInitializingShardIt();
                case REPLICA:
                    return indexShard.replicaActiveInitializingShardIt();
                case PRIMARY_FIRST:
                    return indexShard.primaryFirstActiveInitializingShardsIt();
                case REPLICA_FIRST:
                    return indexShard.replicaFirstActiveInitializingShardsIt();
                case ONLY_LOCAL:
                    return indexShard.onlyNodeActiveInitializingShardsIt(localNodeId);
                case ONLY_NODES:
                    String nodeAttributes = preference.substring(Preference.ONLY_NODES.type().length() + 1);
                    return indexShard.onlyNodeSelectorActiveInitializingShardsIt(nodeAttributes.split(","), nodes);
                default:
                    throw new IllegalArgumentException("unknown preference [" + preferenceType + "]");
            }
        }
        // if not, then use it as the index
        if (awarenessAttributes.length == 0) {
            return indexShard.activeInitializingShardsIt(Murmur3HashFunction.hash(preference));
        } else {
            return indexShard.preferAttributesActiveInitializingShardsIt(awarenessAttributes, nodes, Murmur3HashFunction.hash(preference));
        }
    }
``` 

二，结果震荡问题（Bouncing Results）
 
搜索同一query，结果ES返回的顺序却不尽相同，这就是请求轮询到不同分片，而未设置排序条件，相同相关性评分情况下，是按照所在segment中​lucene id来排序的，相同数据的不同备份之间该id是不能保证一致的，故造成结果震荡问题。
如设置该参数，则有一下9种情况

`_primary`:发送到集群的相关操作请求只会在主分片上执行。
`_primary_first`:指查询会先在主分片中查询，如果主分片找不到（挂了），就会在副本中查询。 
`_replica`:发送到集群的相关操作请求只会在副本上执行。
`_replica_first`：指查询会先在副本中查询，如果副本找不到（挂了），就会在主分片中查询。
`_local`: 指查询操作会优先在本地节点有的分片中查询，没有的话再在其它节点查询。
`_prefer_nodes:abc,xyz`:在提供的节点上优先执行（在这种情况下为'abc'或'xyz'）
`_shards:2,3`：限制操作到指定的分片。 （`2`和“3”）。这个偏好可以与其他偏好组合，但必须首先出现：`_shards：2,3 | _primary`
`_only_nodes:node1,node2`:指在指定id的节点里面进行查询，如果该节点只有要查询索引的部分分片，就只在这部分分片中查找，不同节点之间用“，”分隔。

custom(自定义)：注意自定义的preference参数不能以下划线"_"开头。
当preference为自定义时，即该参数不为空，且开头不以“下划线”开头时，特别注意：如果以用户query作为自定义preference时，一定要处理以下划线开头的情况，这种情况下如果不属于以上8种情况，则会抛出异常。



三，参考：

https://www.elastic.co/guide/en/elasticsearch/reference/current/search-request-preference.html