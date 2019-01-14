# java8函数式编程

lambda被称为闭包，允许函数作为参数传给某块方法，代码本身当作数据处理，实现简洁的语言结构！函数式接口：在接口中只有抽象一个方法，可以隐士转化lambda表达式，所以提出了一个FuctionInterface的接口，用来约束接口中只有一个方法

函数式接口：定义了一个方法的接口

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





#### 数据并行化处理

##### 并行化流操作

只需要改变一个方法的调用，如果已经有一个Sream对象，调用它的parallel方法就能使其拥有并行操作的能力，如果想从一个集合类创建一个流，调用parallelStream 就能立即获得一个拥有并行能力的流。





##### 影响并行流是否快于串行流的因素

* 数据大小：数据足够大的时候、并行处理额外的开销的占比最小，才有效
* 原数据结构：每个管道基于一些初始化的数据通常是集合，数据分割相对容易
* 装箱：处理基本数据比处理 装箱数据快
* 核数：拥有核数越多，获得潜在提升的幅度越大














