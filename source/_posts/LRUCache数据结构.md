title: LRUCache数据结构
toc: true
categories: 算法
tags:
  - java
thumbnail: /logo/suanfa.jpg
---
## LRUCache数据结构
```java
class LRUCache {
    
    private LinkedHashMap<Integer, Integer> cache;

    public LRUCache(int cacheSize) {
        cache = new LinkedHashMap<Integer, Integer>(16, 0.75f, true) {
            protected boolean removeEldestEntry(Map.Entry<Integer, Integer> eldest) {
                return size() > cacheSize;
            }
        };
    }

    public int get(int key) {
        return cache.getOrDefault(key, -1);
    }

    public void put(int key, int value) {
        cache.put(key, value);
    }
}
```