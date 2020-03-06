## 一，排序

对数组从小到大排序

```java
    public void sortUsingLambda(List<Integer> indexs) {
        Collections.sort(indexs, (s1, s2) -> s1.compareTo(s2));
    }
```
更简洁的实现方式

```java
    public void sortUsingLambda(List<Integer> indexs) {
        Collections.sort(indexs, Integer::compareTo);
    }
```
对数组从小到大排序
```java
    public void sortUsingLambda(List<Integer> indexs) {
        Collections.sort(indexs, (s1, s2) -> s2.compareTo(s1));
    }
```
从小到大简洁实现

```java
    public void sortUsingLambda(List<Integer> indexs) {
        Collections.sort(indexs, Comparator.reverseOrder());
    }
```

## 二，list转map
**①：取list中某2个字段作为Map的K,V**
```java
public Map<Long, String> getIdNameMap(List<Account> accounts) {
    return accounts.stream().collect(Collectors.toMap(Account::getId, Account::getUsername));
}
```
**②：将id和实体Bean做为K,V**
```java
public Map<Long, Account> getIdAccountMap(List<Account> accounts) {
    return accounts.stream().collect(Collectors.toMap(Account::getId, account -> account));
}
```
或者这样写
```java
public Map<Long, Account> getIdAccountMap(List<Account> accounts) {
    return accounts.stream().collect(Collectors.toMap(Account::getId, Function.identity()));
}
```
account -> account是一个返回本身的lambda表达式，后面的使用Function接口中的一个默认方法代替，使整个方法更简洁优雅。
**③：key存在重复记录时处理**

```java
public Map<String, Account> getNameAccountMap(List<Account> accounts) {
    return accounts.stream().collect(Collectors.toMap(Account::getUsername, Function.identity(), (key1, key2) -> key2));
}
```
**④：使用某个具体的Map类来保存，如保存时使用LinkedHashMap**

```java
public Map<String, Account> getNameAccountMap(List<Account> accounts) {
    return accounts.stream().collect(Collectors.toMap(Account::getUsername, Function.identity(), (key1, key2) -> key2, LinkedHashMap::new));
}
```
**⑤：List<Object>转List<String,Map<String, String>>**

```java
public Map<String,List<MCode>> getCodeListMap(){
        if(CollectionUtils.isEmpty(codeListMap)){
                List<MCode> codeList = this.getCodeList();
                Set<String> keySet = codeList.stream().map(code -> code.getCodeKbn()).collect(Collectors.toSet());
                Iterator<String> it = keySet.iterator();
                while(it.hasNext()) {
                        String key = it.next();
                        codeListMap.put(key, codeList.stream().filter(code -> code.getCodeKbn().equals(key)).collect(Collectors.toList()));
                }
        }
        return codeListMap;
}
```

三，Map转List

```java
Map<String, String> map = new HashMap<>();
// Convert all Map keys to a List
List<String> result = new ArrayList(map.keySet());
// Convert all Map values to a List
List<String> result2 = new ArrayList(map.values());
// Java 8, Convert all Map keys to a List
List<String> result3 = map.keySet().stream()
	.collect(Collectors.toList());
// Java 8, Convert all Map values  to a List
List<String> result4 = map.values().stream()
	.collect(Collectors.toList());
// Java 8, seem a bit long, but you can enjoy the Stream features like filter and etc.
List<String> result5 = map.values().stream()
	.filter(x -> !"apple".equalsIgnoreCase(x))
	.collect(Collectors.toList());
// Java 8, split a map into 2 List, it works!
// refer example 3 below
```

