title: 数组前k大的数
toc: true
categories: 算法
tags:
  - java
thumbnail: /logo/suanfa.jpg
---
## 数组前k大的数
```java
class KthLargest {

    final PriorityQueue<Integer> queue;
    final int k;
    public KthLargest(int k, int[] nums) {
        this.k = k;
        queue = new PriorityQueue<>();
        for (int num : nums) {
            add(num);
        }
    }

    public int add(int val) {
        if (queue.size() < k)
            queue.offer(val);
        else if (val > queue.peek()) {
            queue.poll();
            queue.offer(val);
        }
        return queue.peek();
    }
}
```

