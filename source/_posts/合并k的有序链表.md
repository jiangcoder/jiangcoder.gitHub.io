title: 合并k个有序链表
toc: true
categories: 算法
tags:
  - java
thumbnail: /logo/suanfa.jpg
---
## 合并k个有序链表
```java
class Solution {
    private PriorityQueue<ListNode> queue = new PriorityQueue<>((o1, o2) -> (o1.val - o2.val));
    public ListNode mergeKLists(ListNode[] lists) {
        if (lists.length == 0)
        return null;
        ListNode resultNode = new ListNode(0);
        ListNode currNode = resultNode;
        for(ListNode node : lists) {
            if (node == null)
            continue;
            queue.add(node);
        }
        while(!queue.isEmpty()) {
            ListNode nextNode = queue.poll();
            currNode.next = nextNode;
            currNode = currNode.next;
            if (nextNode.next != null) {
                queue.add(nextNode.next);
            }
        }
        return resultNode.next;
    }
}
```


