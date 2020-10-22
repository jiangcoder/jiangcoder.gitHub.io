title: sqrt
toc: true
categories: 算法
tags:
  - java
thumbnail: /logo/suanfa.jpg
---
## sqrt
```java
public int mySqrt(int x) {
    if (x == 0)
        return 0;
    long left = 1;
    long right = x/2;
    while (left < right) {
        long mid = (right + left) / 2 + 1;
        if (mid >  x / mid) {
            right = mid - 1;
        } else {
            left = mid;
        }
    }
    return (int)left;
}
```