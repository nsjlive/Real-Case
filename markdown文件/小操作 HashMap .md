**需要 Map 的主键和取值时，应该迭代 entrySet()**

当循环中只需要 Map 的主键时，迭代 keySet() 是正确的。但是，当需要**主键和取值**时，迭代 entrySet() 才是更高效的做法，比先迭代 keySet() 后再去 get 取值性能更佳。

```java
Map<String, String> map = ...;
for (Map.Entry<String, String> entry : map.entrySet()) {
    String key = entry.getKey();
    String value = entry.getValue();
    ...
}
```

