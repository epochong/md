## SpringBoot--RedisTemplate详解

### RedisTemplate

Spring封装了RedisTemplate对象来对redis进行各种操作，它支持所有的redis原生api。

RedisTemplate在Spring代码中的结构如下：

> org.springframework.data.redis.core
> Class RedisTemplate<K,V>
> java.lang.Object
>     org.springframework.data.redis.core.RedisAccessor
>         org.springframework.data.redis.core.RedisTemplate<K,V>

**K:**
模板中的Redis key的类型（通常为String）如：RedisTemplate<String, Object>

注意：如果没有特殊情况，切勿定义成RedisTemplate<Object, Object>，否则根据里氏替换原则，使用的时候会造成类型错误。

**V：**
模板中的Redis value的类型。

RedisTemplate中定义了对Redis的5种数据结构的操作

### StringRedisTemplate与RedisTemplate区别

两者的关系是StringRedisTemplate继承RedisTemplate。

数据是不共通的；也就是说：
StringRedisTemplate只能管理StringRedisTemplate里面的数据。
	RedisTemplate只能管理RedisTemplate中的数据。

其实他们两者之间的区别主要在于他们使用的序列化类:
		RedisTemplate使用的是JdkSerializationRedisSerializer  存入数据会将数据先序列化成字节数组然后在存入Redis数据库。 
		StringRedisTemplate使用的是StringRedisSerializer
使用时注意事项：
		当你的redis数据库里面本来存的是字符串数据或者你要存取的数据就是字符串类型数据的时候，那么你就使用StringRedisTemplate即可。
		但是如果你的数据是复杂的对象类型，而取出的时候又不想做任何的数据转换，直接从Redis里面取出一个对象，那么使用RedisTemplate是更好的选择。

### 操作字符串

```java
redisTemplate.opsForValue();
```

使用：

```java
System.out.println("缓存正在设置。。。。。。。。。");  
redisTemplate.opsForValue().set("key1","value1");  
redisTemplate.opsForValue().set("key2","value2");  
redisTemplate.opsForValue().set("key3","value3");  
redisTemplate.opsForValue().set("key4","value4");  
System.out.println("缓存已经设置完毕。。。。。。。");  
String result1 = redisTemplate.opsForValue().get("key1").toString();  
String result2 = redisTemplate.opsForValue().get("key2").toString();  
String result3 = redisTemplate.opsForValue().get("key3").toString();  
System.out.println("缓存结果为：result：" + result1 + "  "+ result2 + "   " + result3);  
```

### 操作hash

```java
redisTemplate.opsForHash();
```

使用：

```java
Map<String, String> map = new HashMap<String, String>();
map.put("key1", "value1");
map.put("key2", "value2");
map.put("key3", "value3");
map.put("key4", "value4");
map.put("key5", "value5");
redisTemplate.opsForHash().putAll("map", map);

Map<Object, Object> mapEntries = redisTemplate.opsForHash().entries("map");
//{key1=value1, key2=value2, key5=value5, key3=value3, key4=value4}

List<Object> mapValues = redisTemplate.opsForHash().values("map");
//[value1, value2, value5, value3, value4]

Set<Object> mapKeys = redisTemplate.opsForHash().keys("map");
//[key1, key2, key5, key3, key4]

String value = (String) redisTemplate.opsForHash().get("map", "key1");
//value1

System.out.println("redisTemplate.opsForHash().entries(\"map\"):" + mapEntries);
System.out.println("redisTemplate.opsForHash().values(\"map\"):" + mapValues);
System.out.println("redisTemplate.opsForHash().keys(\"map\"):" + mapKeys);
System.out.println("redisTemplate.opsForHash().get(\"map\", \"key1\"):" + value);
```

结果：

> redisTemplate.opsForHash().entries("map"):{key1=value1, key2=value2, key5=value5, key3=value3, key4=value4}
> redisTemplate.opsForHash().values("map"):[value1, value2, value5, value3, value4]
>
> redisTemplate.opsForHash().keys("map"):[key1, key2, key5, key3, key4]
>
> redisTemplate.opsForHash().get("map", "key1"):value1

### 操作list

```java
redisTemplate.opsForList();
```

使用：

```java
List<String> list1 = new ArrayList<>();
list1.add("a1");
list1.add("a2");
list1.add("a3");

List<String> list2 = new ArrayList<>();
list2.add("b1");
list2.add("b2");
list2.add("b3");

redisTemplate.opsForList().leftPush("listkey1", list1);
redisTemplate.opsForList().rightPush("listkey2", list2);
List<String> resultList1 = (List<String>) redisTemplate.opsForList().leftPop("listkey1");
List<String> resultList2 = (List<String>) redisTemplate.opsForList().rightPop("listkey2");
System.out.println("resultList1:" + resultList1);
System.out.println("resultList2:" + resultList2);
```

结果：

> resultList1:[a1, a2, a3]
> resultList2:[b1, b2, b3]

### 操作set

```java
redisTemplate.opsForSet();
```

使用：

```java
SetOperations<String, Object> set = redisTemplate.opsForSet();
set.add("set1", "22");
set.add("set1", "33");
set.add("set1", "44");
Set<Object> resultSet = redisTemplate.opsForSet().members("set1");
System.out.println("resultSet:" + resultSet);
```

结果：

resultSet:[22, 33, 44]

### 操作有序set

```java
redisTemplate.opsForZSet();
```

### StringRedisTemplate与RedisTemplate

两者的关系是StringRedisTemplate继承RedisTemplate。

两者的数据是不共通的；也就是说StringRedisTemplate只能管理StringRedisTemplate里面的数据，RedisTemplate只能管理RedisTemplate中的数据。

SDR默认采用的序列化策略有两种，一种是String的序列化策略，一种是JDK的序列化策略。

StringRedisTemplate默认采用的是String的序列化策略，保存的key和value都是采用此策略序列化保存的。

RedisTemplate默认采用的是JDK的序列化策略，保存的key和value都是采用此策略序列化保存的。

RedisTemplate配置如下：

```java
@Bean
public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory){
  Jackson2JsonRedisSerializer<Object> jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer<Object>(Object.class);
  ObjectMapper om = new ObjectMapper();
  om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
  om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
  jackson2JsonRedisSerializer.setObjectMapper(om);
  RedisTemplate<String, Object> template = new RedisTemplate<String, Object>();
  template.setConnectionFactory(redisConnectionFactory);
  template.setKeySerializer(jackson2JsonRedisSerializer);
  template.setValueSerializer(jackson2JsonRedisSerializer);
  template.setHashKeySerializer(jackson2JsonRedisSerializer);
  template.setHashValueSerializer(jackson2JsonRedisSerializer);
  template.afterPropertiesSet();
  return template;
}
```

Redis的String数据结构 （推荐使用StringRedisTemplate）
如果使用RedisTemplate需要更改序列化方式：

```java
RedisSerializer<String> stringSerializer = new StringRedisSerializer();
template.setKeySerializer(stringSerializer );
template.setValueSerializer(stringSerializer );
template.setHashKeySerializer(stringSerializer );
template.setHashValueSerializer(stringSerializer );
```

public interface ValueOperations<K,V>
Redis operations for simple (or in Redis terminology ‘string’) values.
ValueOperations可以对String数据结构进行操作：

#### set 

- void set(K key, V value);

使用：

```java
redisTemplate.opsForValue().set("name","tom");
```

结果：redisTemplate.opsForValue().get("name")  输出结果为tom

set void set(K key, V value, long timeout, TimeUnit unit);
使用：

```java
redisTemplate.opsForValue().set("name","tom",10, TimeUnit.SECONDS);
```

结果：redisTemplate.opsForValue().get("name")由于设置的是10秒失效，十秒之内查询有结果，十秒之后返回为null

- void set(K key, V value, long offset);

该方法是用 value 参数覆写(overwrite)给定 key 所储存的字符串值，从偏移量 offset 开始
使用：

```java
template.opsForValue().set("key","hello world");
template.opsForValue().set("key","redis", 6);
System.out.println("***************"+template.opsForValue().get("key"));
```

>  结果：`***************hello redis`

- void set(K key, long index, V value);

在列表中index的位置设置value值
使用：

```java
System.out.println(template.opsForList().range("listRight",0,-1));
template.opsForList().set("listRight",1,"setValue");
System.out.println(template.opsForList().range("listRight",0,-1));
```

结果:

[java, python, oc, c++]
[java, setValue, oc, c++]

#### setIfAbsent 

- `Boolean setIfAbsent(K key, V value);`

使用：

```
System.out.println(template.opsForValue().setIfAbsent("multi1","multi1"));//false  multi1之前已经存在
System.out.println(template.opsForValue().setIfAbsent("multi111","multi111"));//true  multi111之前不存在
```

> 结果：false
> true

#### multiSet 

- void multiSet(Map<? extends K, ? extends V> m);

为多个键分别设置它们的值
使用：

```java
Map<String,String> maps = new HashMap<String, String>();
maps.put("multi1","multi1");
maps.put("multi2","multi2");
maps.put("multi3","multi3");
template.opsForValue().multiSet(maps);
List<String> keys = new ArrayList<String>();
keys.add("multi1");
keys.add("multi2");
keys.add("multi3");
System.out.println(template.opsForValue().multiGet(keys));
```

结果：[multi1, multi2, multi3]

```java
SetOperations<String, Object> set = redisTemplate.opsForSet();
set.add("set1","22");  
set.add("set1","33");  
set.add("set1","44");  
Set<Object> resultSet = redisTemplate.opsForSet().members("set1");  
System.out.println("resultSet:" + resultSet); 
```

运行结果为：

运行结果为：

resultSet:[22, 33, 44]

#### multiSetIfAbsent 

- Boolean multiSetIfAbsent(Map<? extends K, ? extends V> m);

为多个键分别设置它们的值，如果存在则返回false，不存在返回true
使用：

```java
Map<String,String> maps = new HashMap<String, String>();
maps.put("multi11","multi11");
maps.put("multi22","multi22");
maps.put("multi33","multi33");
Map<String,String> maps2 = new HashMap<String, String>();
maps2.put("multi1","multi1");
maps2.put("multi2","multi2");
maps2.put("multi3","multi3");
System.out.println(template.opsForValue().multiSetIfAbsent(maps));
System.out.println(template.opsForValue().multiSetIfAbsent(maps2));
```

结果：true
false

#### get 

- `V get(Object key);`

使用：

```java
template.opsForValue().set("key","hello world");
System.out.println("***************"+template.opsForValue().get("key"));
```

结果：`***************hello world`

#### getAndSet 

- V getAndSet(K key, V value);

设置键的字符串值并返回其旧值
使用：

```java
template.opsForValue().set("getSetTest","test");
System.out.println(template.opsForValue().getAndSet("getSetTest","test2"));
```

结果：test

#### multiGet 

- List multiGet(Collection keys);

为多个键分别取出它们的值
使用：

```java
Map<String,String> maps = new HashMap<String, String>();
maps.put("multi1","multi1");
maps.put("multi2","multi2");
maps.put("multi3","multi3");
template.opsForValue().multiSet(maps);
List<String> keys = new ArrayList<String>();
keys.add("multi1");
keys.add("multi2");
keys.add("multi3");
System.out.println(template.opsForValue().multiGet(keys));
```

结果：[multi1, multi2, multi3]

#### increment

- `Long increment(K key, long delta);`

支持整数
使用：

```java
template.opsForValue().increment("increlong",1);
System.out.println("***************"+template.opsForValue().get("increlong"));
```

结果：`***************1`

- Double increment(K key, double delta);

也支持浮点数
使用：

```java
template.opsForValue().increment("increlong",1.2);
System.out.println("***************"+template.opsForValue().get("increlong"));
```

结果：`***************2.2`

#### append 

- `Integer append(K key, String value);`

如果key已经存在并且是一个字符串，则该命令将该值追加到字符串的末尾。如果键不存在，则它被创建并设置为空字符串，因此APPEND在这种特殊情况下将类似于SET。
使用：

```java
template.opsForValue().append("appendTest","Hello");
System.out.println(template.opsForValue().get("appendTest"));
template.opsForValue().append("appendTest","world");
System.out.println(template.opsForValue().get("appendTest"));
```

结果：

Hello
Helloworld

#### get 

- `String get(K key, long start, long end);`

截取key所对应的value字符串
使用：appendTest对应的value为Helloworld

```java
System.out.println("*********"+template.opsForValue().get("appendTest",0,5));
结果：*********Hellow
使用：System.out.println("*********"+template.opsForValue().get("appendTest",0,-1));
结果：*********Helloworld
使用：System.out.println("*********"+template.opsForValue().get("appendTest",-3,-1));
结果：*********rld
```

#### size 

- `Long size(K key);`

返回key所对应的value值得长度
使用：

```java
template.opsForValue().set("key","hello world");
System.out.println("***************"+template.opsForValue().size("key"));
```

结果：`***************11`

#### setBit 

- `Boolean setBit(K key, long offset, boolean value);`

对 key 所储存的字符串值，设置或清除指定偏移量上的位(bit)
key键对应的值value对应的ascii码,在offset的位置(从左向右数)变为value
使用：

```java
template.opsForValue().set("bitTest","a");
// 'a' 的ASCII码是 97。转换为二进制是：01100001
// 'b' 的ASCII码是 98  转换为二进制是：01100010
// 'c' 的ASCII码是 99  转换为二进制是：01100011
//因为二进制只有0和1，在setbit中true为1，false为0，因此我要变为'b'的话第六位设置为1，第七位设置为0
template.opsForValue().setBit("bitTest",6, true);
template.opsForValue().setBit("bitTest",7, false);
System.out.println(template.opsForValue().get("bitTest"));
```

结果：b

#### getBit

- `Boolean getBit(K key, long offset);`

获取键对应值的ascii码的在offset处位值
使用：

```java
System.out.println(template.opsForValue().getBit("bitTest",7));
```

结果：false

Redis的List数据结构
把RedisTemplate序列化方式改回之前的

```java
Jackson2JsonRedisSerializer<Object> jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer<Object>(Object.class);
ObjectMapper om = new ObjectMapper();
om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
jackson2JsonRedisSerializer.setObjectMapper(om);
RedisTemplate<String, Object> template = new RedisTemplate<String, Object>();
template.setKeySerializer(jackson2JsonRedisSerializer);
template.setValueSerializer(jackson2JsonRedisSerializer);
template.setHashKeySerializer(jackson2JsonRedisSerializer);
template.setHashValueSerializer(jackson2JsonRedisSerializer);
```

public interface ListOperations<K,V>
Redis列表是简单的字符串列表，按照插入顺序排序。你可以添加一个元素导列表的头部（左边）或者尾部（右边）
ListOperations专门操作list列表：

#### range

- `List range(K key, long start, long end);`

返回存储在键中的列表的指定元素。偏移开始和停止是基于零的索引，其中0是列表的第一个元素（列表的头部），1是下一个元素
使用：

```java
System.out.println(template.opsForList().range("list",0,-1));
```

结果:[c#, c++, python, java, c#, c#]

#### trim

- `void trim(K key, long start, long end);`

修剪现有列表，使其只包含指定的指定范围的元素，起始和停止都是基于0的索引
使用：

```java
System.out.println(template.opsForList().range("list",0,-1));
template.opsForList().trim("list",1,-1);//裁剪第一个元素
System.out.println(template.opsForList().range("list",0,-1));
```

结果:

[c#, c++, python, java, c#, c#]
[c++, python, java, c#, c#]

#### size

- `Long size(K key);`

返回存储在键中的列表的长度。如果键不存在，则将其解释为空列表，并返回0。当key存储的值不是列表时返回错误。
使用：

```java
System.out.println(template.opsForList().size("list"));
```

结果:6

#### leftPush

- `Long leftPush(K key, V value);`

将所有指定的值插入存储在键的列表的头部。如果键不存在，则在执行推送操作之前将其创建为空列表。（从左边插入）
使用：

```java
template.opsForList().leftPush("list","java");
template.opsForList().leftPush("list","python");
template.opsForList().leftPush("list","c++");
```

结果:返回的结果为推送操作后的列表的长度

- `Long leftPush(K key, V pivot, V value);`

把value值放到key对应列表中pivot值的左面，如果pivot值存在的话
使用：

```java
template.opsForList().leftPush("list","java","oc");
System.out.print(template.opsForList().range("list",0,-1));
```

结果：[c++, python, oc, java, c#, c#]

#### leftPushAll

- `Long leftPushAll(K key, V… values);`

批量把一个数组插入到列表中
使用：

```java
String[] stringarrays = new String[]{"1","2","3"};
template.opsForList().leftPushAll("listarray",stringarrays);
System.out.println(template.opsForList().range("listarray",0,-1));
```

结果:[3, 2, 1]

- `Long leftPushAll(K key, Collection values);`

批量把一个集合插入到列表中
使用：

```java
List<Object> strings = new ArrayList<Object>();
strings.add("1");
strings.add("2");
strings.add("3");
template.opsForList().leftPushAll("listcollection4", strings);
System.out.println(template.opsForList().range("listcollection4",0,-1));
```

结果:[3, 2, 1]

#### leftPushIfPresent

- `Long leftPushIfPresent(K key, V value);`

只有存在key对应的列表才能将这个value值插入到key所对应的列表中
使用：

```java
System.out.println(template.opsForList().leftPushIfPresent("leftPushIfPresent","aa"));   System.out.println(template.opsForList().leftPushIfPresent("leftPushIfPresent","bb"));
==========分割线===========
System.out.println(template.opsForList().leftPush("leftPushIfPresent","aa"));        System.out.println(template.opsForList().leftPushIfPresent("leftPushIfPresent","bb"));
```

结果:
0
0
==========分割线===========
1
2

- `Long leftPush(K key, V pivot, V value);`

把value值放到key对应列表中pivot值的左面，如果pivot值存在的话
使用：

```java
template.opsForList().leftPush("list","java","oc");
System.out.print(template.opsForList().range("list",0,-1));
```

结果：[c++, python, oc, java, c#, c#]

#### rightPush

- `Long rightPush(K key, V value);`

将所有指定的值插入存储在键的列表的头部。如果键不存在，则在执行推送操作之前将其创建为空列表。（从右边插入）
使用：

```java
template.opsForList().rightPush("listRight","java");
template.opsForList().rightPush("listRight","python");
template.opsForList().rightPush("listRight","c++");
```

结果:

- `Long rightPush(K key, V pivot, V value);`

把value值放到key对应列表中pivot值的右面，如果pivot值存在的话
使用：

```java
System.out.println(template.opsForList().range("listRight",0,-1));
template.opsForList().rightPush("listRight","python","oc");
System.out.println(template.opsForList().range("listRight",0,-1));
```

结果:

> [java, python, c++]
> [java, python, oc, c++]

#### rightPushAll

- `Long rightPushAll(K key, V… values);`

使用：

```java
String[] stringarrays = new String[]{"1","2","3"};
template.opsForList().rightPushAll("listarrayright",stringarrays);
System.out.println(template.opsForList().range("listarrayright",0,-1));
```

结果:[1, 2, 3]

- `Long rightPushAll(K key, Collection values);`

使用：

```java
List<Object> strings = new ArrayList<Object>();
strings.add("1");
strings.add("2");
strings.add("3");
template.opsForList().rightPushAll("listcollectionright", strings);
System.out.println(template.opsForList().range("listcollectionright",0,-1));
```

结果:[1, 2, 3]

#### rightPushIfPresent

- `Long rightPushIfPresent(K key, V value);`

只有存在key对应的列表才能将这个value值插入到key所对应的列表中
使用：

```java
System.out.println(template.opsForList().rightPushIfPresent("rightPushIfPresent","aa"));      System.out.println(template.opsForList().rightPushIfPresent("rightPushIfPresent","bb"));
System.out.println("==========分割线===========");
System.out.println(template.opsForList().rightPush("rightPushIfPresent","aa"));      System.out.println(template.opsForList().rightPushIfPresent("rightPushIfPresent","bb"));
```

结果:

> 0
> 0
> ==========分割线===========
> 1
> 2

void set(K key, long index, V value);
在列表中index的位置设置value值
使用：

```java
System.out.println(template.opsForList().range("listRight",0,-1));
template.opsForList().set("listRight",1,"setValue");
System.out.println(template.opsForList().range("listRight",0,-1));
```

结果:

[java, python, oc, c++]
[java, setValue, oc, c++]

#### remove

- `Long remove(K key, long count, Object value);`

从存储在键中的列表中删除等于值的元素的第一个计数事件。
计数参数以下列方式影响操作：
count> 0：删除等于从头到尾移动的值的元素。
count <0：删除等于从尾到头移动的值的元素。
count = 0：删除等于value的所有元素。
使用：

```java
System.out.println(template.opsForList().range("listRight",0,-1));
template.opsForList().remove("listRight",1,"setValue");//将删除列表中存储的列表中第一次次出现的“setValue”。
System.out.println(template.opsForList().range("listRight",0,-1));
```

结果:

[java, setValue, oc, c++]
[java, oc, c++]

#### index

- `V index(K key, long index);`

根据下表获取列表中的值，下标是从0开始的
使用：

```java
System.out.println(template.opsForList().range("listRight",0,-1));
System.out.println(template.opsForList().index("listRight",2));
```

结果:

[java, oc, c++]
c++

#### leftPop

- `V leftPop(K key);`

弹出最左边的元素，弹出之后该值在列表中将不复存在
使用：

```javaj
System.out.println(template.opsForList().range("list",0,-1));
System.out.println(template.opsForList().leftPop("list"));
System.out.println(template.opsForList().range("list",0,-1));
```

结果:
[c++, python, oc, java, c#, c#]
c++
[python, oc, java, c#, c#]

- V leftPop(K key, long timeout, TimeUnit unit);

移出并获取列表的第一个元素， 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。
使用：用法与 leftPop(K key);一样

Long rightPushAll(K key, V… values);
使用：

```java
String[] stringarrays = new String[]{"1","2","3"};
template.opsForList().rightPushAll("listarrayright",stringarrays);
System.out.println(template.opsForList().range("listarrayright",0,-1));
```

结果:[1, 2, 3]

- Long rightPushAll(K key, Collection values);

使用：

```java
List<Object> strings = new ArrayList<Object>();
strings.add("1");
strings.add("2");
strings.add("3");
template.opsForList().rightPushAll("listcollectionright", strings);
System.out.println(template.opsForList().range("listcollectionright",0,-1));
```

结果:[1, 2, 3]

#### rightPushIfPresent

- Long rightPushIfPresent(K key, V value);

只有存在key对应的列表才能将这个value值插入到key所对应的列表中
使用：

```java
System.out.println(template.opsForList().rightPushIfPresent("rightPushIfPresent","aa"));  System.out.println(template.opsForList().rightPushIfPresent("rightPushIfPresent","bb"));
System.out.println("==========分割线===========");
System.out.println(template.opsForList().rightPush("rightPushIfPresent","aa"));       System.out.println(template.opsForList().rightPushIfPresent("rightPushIfPresent","bb"));
```

结果:

0
0
==========分割线===========
1
2

#### remove

- Long remove(K key, long count, Object value);

从存储在键中的列表中删除等于值的元素的第一个计数事件。
计数参数以下列方式影响操作：
count> 0：删除等于从头到尾移动的值的元素。
count <0：删除等于从尾到头移动的值的元素。
count = 0：删除等于value的所有元素。
使用：

```java
System.out.println(template.opsForList().range("listRight",0,-1));
template.opsForList().remove("listRight",1,"setValue");//将删除列表中存储的列表中第一次次出现的“setValue”。
System.out.println(template.opsForList().range("listRight",0,-1));
```

结果:

[java, setValue, oc, c++]
[java, oc, c++]

#### rightPop

- V rightPop(K key, long timeout, TimeUnit unit);

移出并获取列表的最后一个元素， 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。
使用：用法与 rightPop(K key);一样

#### rightPopAndLeftPush

- V rightPopAndLeftPush(K sourceKey, K destinationKey);

用于移除列表的最后一个元素，并将该元素添加到另一个列表并返回。
使用：

```java
System.out.println(template.opsForList().range("list",0,-1));
template.opsForList().rightPopAndLeftPush("list","rightPopAndLeftPush");
System.out.println(template.opsForList().range("list",0,-1));
System.out.println(template.opsForList().range("rightPopAndLeftPush",0,-1));
```

结果:

[oc, java,c#]
[oc, java]
[c#]

- V rightPopAndLeftPush(K sourceKey, K destinationKey, long timeout, TimeUnit unit);

用于移除列表的最后一个元素，并将该元素添加到另一个列表并返回，如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。
使用：用法与rightPopAndLeftPush(K sourceKey, K destinationKey)一样