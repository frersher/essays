# java8函数式编程

## 高级集合和收集器

### 收集器

* 转换成其他集合

  toList、toSet、toCollection 

  使用 toCollection，用定制的集合收集元素

  `stream.collect(toCollection(TreeSet::new))`

* 转换成值

​      maxBy、minBy允许按照特定的顺序生成一个值

```找出成员最多的乐队
//成员最多的乐队
public Optional<Artist> biggestGroup(Stream<Artist> artists) { Function<Artist,Long> getCount = artist -> artist.getMembers().count(); return artists.collect(maxBy(comparing(getCount)));
}
```

* 数据分块

  partitioningBy 接受一个留，并将其分为两个部分，使用Pre滴擦特对象判断元素属于哪一个部分，并根据布尔值返回一个Map到列表。因此，对于true List中的元素，Predicate返回true;对其他List中的元素，Predicate 返回 false。

  ![image-20181028211018643](/Users/cb/Library/Application Support/typora-user-images/image-20181028211018643.png)



```
//艺术家分成独唱和乐队两组
public Map<Boolean, List<Artist>> bandsAndSolo(Stream<Artist>       artists) { return artists.collect(partitioningBy(artist -> artist.isSolo()));
}
---
artist -> artist.isSolo() Lambda表达式可以使用方法引用代替 
Artist::isSolo
```



* 数据分组

  数据分组是更自然的数据分割，与分成true、false不同，可以使任意值对数据分组。

  ```
  //通过国籍对艺术家分组
  private static Map<String,List<Artist>> getArtistsByNational(Stream<Artist> artistStream){
          return artistStream.collect(groupingBy(artist -> artist.getNationality()));
      }
  ```




* 字符串

  很多时候收集流中的数据是为了生成一个字符串，如：取一组学生的姓名列表。

java8以前通过for循环不断迭代列表使用StringBuilder对象记录结果。代码很长

java8提供流和收集器可以通过清晰的代码解决这个问题

```
String result = students.stream.map(Student::getName()).collect(Collection.jioning(", ","[","]"));
```



* 组合收集器










