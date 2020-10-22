title: Kafka节点平滑迁移
toc: true
categories: kafka
tags:
  - java
thumbnail: /logo/kafka.jpg
---
## 一，查看所有的topic

```java
bin/kafka-topics.sh --zookeeper ip:port --list
```

## 二，将topic组装为JSON格式
kafka后台脚本只支持如下JSON格式节点迁移

```java
{
  "topics": [
    {
      "topic": "plat_order_core_dubbo_access_topic"
    }  ],
  "version": 1
}
```
生成该格式代码，可参考使用如下Java代码

```java
import com.alibaba.fastjson.JSONObject;
import com.google.common.collect.Lists;
import org.apache.commons.lang3.StringUtils;
 
import java.io.BufferedReader;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.nio.charset.StandardCharsets;
import java.util.ArrayList;
import java.util.List;
 
public class Topic2JsonUtil {
 
    public static void main(String[] args) {
        List<String> topicList = readFileAsListFromJarPath("topic.txt");
        List<JSONObject> topicJsonList = Lists.newArrayList();
        topicList.forEach(topic -> {
            JSONObject topicJson = new JSONObject();
            topicJson.put("topic", topic);
            topicJsonList.add(topicJson);
        });
        JSONObject resultJson = new JSONObject();
        resultJson.put("topics", topicJsonList);
        System.out.println(resultJson);
    }
 
    public static List<String> readFileAsListFromJarPath(String fileName) {
        List<String> result = new ArrayList<>();
        try {
            InputStream inputStream = Topic2JsonUtil.class.getClassLoader().getResourceAsStream(fileName);
            BufferedReader reader = new BufferedReader(new InputStreamReader(inputStream, StandardCharsets.UTF_8));
            String line;
            while ((line = reader.readLine()) != null) {
                if (StringUtils.isNotBlank(line)) {
                    result.add(line);
                }
            }
            reader.close();
        } catch (Exception e) {
            LogUtil.ROOT.error("", e);
        }
        StringBuilder builder = new StringBuilder();
        if (result.size() != 0) {
            result.forEach(builder::append);
        }
        return result;
    }
 
}
```

## 三，生成迁移计划
使用如下命令生成迁移计划
如下示例代表将topic所有的节点数据重新路由到4、5、6节点上，并将迁移计划输出到plan-move.json文件中

```java
./bin/kafka-reassign-partitions.sh --zookeeper ip:port --topics-to-move-json-file  topics-to-move.json  --broker-list  "4,5,6"  --generate  > plan-move.json
```
生成的计划格式化后如下：

```java
Current partition replica assignment
{
	"version": 1,
	"partitions": [{
		"topic": "erp_java_topic",
		"partition": 9,
		"replicas": [12, 11, 13, 1],
		"log_dirs": ["any", "any", "any", "any"]
	}]
}
Proposed partition reassignment configuration
{
	"version": 1,
	"partitions": [{
		"topic": "erp_java_topic",
		"partition": 9,
		"replicas": [4, 5, 6, 7],
		"log_dirs": ["any", "any", "any", "any"]
	}]
}
```
保留迁移计划JSON，即下面的JSON内容；
## 四，执行迁移计划
执行迁移计划
```java
./bin/kafka-reassign-partitions.sh --zookeeper ip:port --reassignment-json-file plan-move.json --execute
```
查看迁移进度
```java
./bin/kafka-reassign-partitions.sh --zookeeper ip:port --reassignment-json-file plan-move.json --verify
```
