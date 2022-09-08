# **二十、Scala 篇**

### 1.1. 为什么使用 Scala？

Spark 支持 Scala，用 Scala 面向函数式编程，相比 Java 支持性更好、代码更简洁。

### 1.2. 什么是 Scala？

1. Scala，是一门多范式（支持面向函数、面向对象）编程语言。
2. Scala，类似 Java，也是一门基于 JVM 的语言。
3. Scala，可以和 Java 无缝相互操作，在任意地方调用 Java 代码。

|            | Scala                                                        | Java                                                         |
| ---------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 变量声明   | val 常量、var 变量、lazy 懒加载，类型自动推测                | final 常量，指定类型                                         |
| 数据类型   | Byte、Char、Short、Int、Long、Float、Double、Boolean、StringOps、RichInt、RichDouble、RichChar | byte / Byte、short / Short、int / Integer、long / Long、float / Float、double / Double、char / Character、String |
| 操作符     | 不支持 ++ / --                                               | 支持 ++ / --                                                 |
| 返回值     | 最后一行                                                     | return 显示声明                                              |
| 语句终结符 | 可分号，可不分号                                             | 分号                                                         |
| if 表达式  | 有返回值                                                     | 无返回值                                                     |
| 循环       | for(i <- 1 to n)、for(i <- 1 until n)、for(c <- "hello scala")、or(i <- 1 to 10 if i % 2 == 0)、for(i <- 1 to 10) yield i * 2、while | for、while、do while                                         |
| 函数声明   | def name(var : varType) : Type 定义、Unit 空值、函数式编程、支持默认参数、sum(nums: Int*) | 修饰符 Type name(varType var) 定义、void 空值、函数式编程、不支持默认参数、sum(...) |
| 集合       | Array /  List、Tuple、ArrayBuffer / ListBuffer、可变 & 不可变集合 | new int[]、ArrayList、可不可变看具体实现                     |
| 类         | 主 Constructor、辅助 Constructor、case class                 | 支持构造函数重载、main 方法、static class                    |
| 对象       | object 伴生对象、apply 方法、main 方法、new class 得到实例对象 | new class 得到 class 实例对象                                |
| 接口       | trait、extends A with B                                      | interface、implements A, B                                   |
| 模式匹配   | match case + 变量值匹配、match case + 类型匹配、match case + Option | switch case                                                  |
| 隐式转换   | implicit def object2Cat(obj: Object): cat、增强上下文对象    | Integer 等包装类                                             |

### 1.3. Scala 安装步骤？

下载、解压、配置环境变量

### 1.4. Scala 基本语法？

#### 1）变量

1. Scala 变量分为两种：var 代表可变变量，val 代表不可变的常量，建议不可变的变量，都声明为 val 常量，提高程序的健壮性。
2. var 和 var 变量，如果没有指定类型，则 Scala 会自动根据值，进行类型推断，建议每个变量都手动指定类型 `var c: Int = 1`。

```scala
scala> var a = 1
a: Int = 1

scala> a = 2
a: Int = 2

scala> val b = 1
b: Int = 1

scala> b = 2
<console>:12: error: reassignment to val
       b = 2
```

#### 2）数据类型

| 类型       | 分类           |
| ---------- | -------------- |
| Byte       | 基本数据类型   |
| Char       |                |
| Short      |                |
| Int        |                |
| Long       |                |
| Float      |                |
| Double     |                |
| Boolean    |                |
| StringOps  | 增强版数据类型 |
| RichInt    |                |
| RichDouble |                |
| RichChar   |                |

#### 3）操作符

与 Java 类似，但 Scala 没有提供 `++` 和 `--` 操作符。

```scala
scala> var count = 1
count: Int = 1

scala> count++
<console>:13: error: value ++ is not a member of Int
       count++
            ^

scala> count--
<console>:13: error: value -- is not a member of Int
       count--
```

#### 4）if 表达式

if 表达式有返回值，即 if 或者 else 块中，最后一行语句返回的值。

```scala
scala> val age = 20
age: Int = 20

scala> if(age > 18) 1 else 0
res0: Int = 1

scala> val res = if(age > 18) 1 else 0
res: Int = 1

scala> res
res1: Int = 1
```

#### 5）命令行多行模式

```scala
scala> :paste
// Entering paste mode (ctrl-D to finish)

val age = 20
if(age > 18)
  1
else
  0

// Exiting paste mode, now interpreting.

age: Int = 20
res2: Int = 1
```

#### 6）语句终结符

1. Scala 默认不需要语句终结符，此时视每一行作为一个语句。
2. 而如果一行要放多条语句，则前一条语句必须使用语句终结符 ";" 分号。

```scala
scala> val age = 20 if(age > 18) 1 else 0
<console>:1: error: ';' expected but 'if' found.
       val age = 20 if(age > 18) 1 else 0
                    ^

scala> val age = 20; if(age > 18) 1 else 0
age: Int = 20
res3: Int = 1
```

#### 7）循环

##### 1、for 循环

```scala
// 遍历1~10
scala> :paste
// Entering paste mode (ctrl-D to finish)

val n = 10
for(i <- 1 to n)
println(i)

// Exiting paste mode, now interpreting.

1
2
3
4
5
6
7
8
9
10
n: Int = 10

// 遍历1~(10-1)
scala> :paste
// Entering paste mode (ctrl-D to finish)

val n = 10
for(i <- 1 until n)
println(i)

// Exiting paste mode, now interpreting.

1
2
3
4
5
6
7
8
9
n: Int = 10

// 遍历字符串每个字符
scala> for(c <- "hello scala") println(c)
h
e
l
l
o

s
c
a
l
a

// for循环+多条语句
scala> :paste
// Entering paste mode (ctrl-D to finish)

for(i <- 1 to 5)
{
  println(i)
  println("hehe")
}

// Exiting paste mode, now interpreting.

1
hehe
2
hehe
3
hehe
4
hehe
5
hehe
```

##### 2、while 循环

```scala
scala> :paste
// Entering paste mode (ctrl-D to finish)

var n = 10
while(n > 0)
{
println(n)
n -= 1
}

// Exiting paste mode, now interpreting.

10
9
8
7
6
5
4
3
2
1
n: Int = 0
```

##### 3、高级 for 循环 - if 守卫

```scala
scala> for(i <- 1 to 10 if i % 2 == 0) println(i)
2
4
6
8
10
```

##### 4、高级 for 循环 - for 推导式

```scala
// 根据yield后指定规则，对迭代的数据进行处理，并将结果集组合为一个集合
scala> for(i <- 1 to 10) yield i * 2
res7: scala.collection.immutable.IndexedSeq[Int] = Vector(2, 4, 6, 8, 10, 12, 14, 16, 18, 20)
```

#### 8）集合

1. Scala 集合，分为可变集合，以及不可变集合。
2. 可变集合，指的是初始化后，仍然可以改变集合中的元素，位于 `scala.collection.mutable` 包中。
3. 不可变集合，指的是初始化后，不能再改变集合中的元素，位于 `scala.collection.immutable` 包中，如果集合声明时，未指定具体的包，则默认为不可变集合。

![1662440699791](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1662440699791.png)

| 类型          | 长度（添加） | 元素（更新） | 元素类型 |
| ------------- | ------------ | ------------ | -------- |
| Array         | 不可变       | 可变         | 固定     |
| Tuple         | 不可变       | 可变         | 不固定   |
| List          | 不可变       | 不可变       | 固定     |
| ListBuffer    | 可变         | 可变         | 固定     |
| ArrayBuffer   | 可变         | 可变         | 固定     |
| HashSet       | 可变、不可变 | 可变、不可变 | 固定     |
| LinkedHashSet | 可变         | 可变         | 固定     |
| SortedSet     | 可变、不可变 | 可变、不可变 | 固定     |
| HashMap       | 可变、不可变 | 可变、不可变 | 固定     |
| LinkedHashMap | 可变         | 可变         | 固定     |
| SortedMap     | 可变、不可变 | 可变、不可变 | 固定     |

##### 1、Array

1. 类似于 Java#数组，即长度都是不可变的，本质上，Scala 数组的底层实现，就是 Java 数组。
2. 在数组初始化后，长度就固定下来了，然后元素会根据类型进行初始化，也可以直接使用 Array(...) 创建数组，此时会自动推断元素类型。

```scala
scala> val a = new Array[Int](10)
a: Array[Int] = Array(0, 0, 0, 0, 0, 0, 0, 0, 0, 0)

scala> a(0)
res14: Int = 0

scala> a(0) = 1

scala> a(0)
res16: Int = 1

// 自动推断数组类型
scala> val a = Array("hello", "world")
a: Array[String] = Array(hello, world)

scala> val a = Array("hello", 30)
a: Array[Any] = Array(hello, 30)
```

##### 2、Tuple

1. Tuple，元组，和 Array 类似，都是长度不可变的，但不同的是，元组可以包含不同类型的元素。
2. 注意，Tuple 的下标是从 1 开始的。
3. 目前 Scala 元组的最大长度为 22，更长的话需要使用集合或者 Array。

```scala
// 构造tuple
scala> val t = (1, 3.14, "hehe")
t: (Int, Double, String) = (1,3.14,hehe)

scala> val t = Tuple3(1, 3.14, "hehe")
t: (Int, Double, String) = (1,3.14,hehe)

// 获取tuple对应下标元素
scala> t._1
res40: Int = 1

scala> t._3
res41: String = hehe
```

##### 3、Seq

###### ① List

List，代表不可变集合

```scala
scala> val l = List(1, 2, 3, 4)
l: List[Int] = List(1, 2, 3, 4)

// 不可追加
scala> l += 5
<console>:13: error: value += is not a member of List[Int]
  Expression does not convert to assignment because receiver is not assignable.
       l += 5
         ^
// 不可修改
scala> l(2) = 5
<console>:13: error: value update is not a member of List[Int]
       l(2) = 5

// 获取第一个元素
scala> l.head
res2: Int = 1

// 获取第一个元素之后的所有元素
scala> l.tail
res3: List[Int] = List(2, 3, 4)

// 拼接元素
scala> l.head :: l.tail
res4: List[Int] = List(1, 2, 3, 4)

// 迭代集合
scala> for(i <- l) println(i)
1
2
3
4
```

###### ② ListBuffer

类似于 Java#LinkedList，代表可以支持动态增加、或者删除元素。

```scala
scala> val lb = scala.collection.mutable.ListBuffer[Int]()
lb: scala.collection.mutable.ListBuffer[Int] = ListBuffer()

// 追加元素
scala> lb += 1
res7: lb.type = ListBuffer(1)

scala> lb += (2, 3, 4, 5)
res8: lb.type = ListBuffer(1, 2, 3, 4, 5)

// 迭代集合
scala> for(i <- lb) println(i)
1
2
3
4
5

// ListBuffer转List
scala> val l = lb.toList
l: List[Int] = List(1, 2, 3, 4, 5)

// List转ListBuffer
scala> l.to[scala.collection.mutable.ListBuffer]
res33: scala.collection.mutable.ListBuffer[Int] = ListBuffer(1, 2, 3, 4, 5)
```

###### ③ ArrayBuffer

类似于 Java#ArrayList，长度可变，支持添加、删除元素。

```scala
// 不带元素构造
scala> val b = new scala.collection.mutable.ArrayBuffer[Int]()
b: scala.collection.mutable.ArrayBuffer[Int] = ArrayBuffer()

// 带元素构造
scala> val b = new scala.collection.mutable.ArrayBuffer[Int](1, 2, 3, 4, 5)
<console>:11: error: overloaded method constructor ArrayBuffer with alternatives:
  ()scala.collection.mutable.ArrayBuffer[Int] <and>
  (initialSize: Int)scala.collection.mutable.ArrayBuffer[Int]
 cannot be applied to (Int, Int, Int, Int, Int)
       val b = new scala.collection.mutable.ArrayBuffer[Int](1, 2, 3, 4, 5)
               ^
scala> val b = scala.collection.mutable.ArrayBuffer[Int](1, 2, 3, 4, 5)
b: scala.collection.mutable.ArrayBuffer[Int] = ArrayBuffer(1, 2, 3, 4, 5)

// 添加元素
scala> b += 6
res17: b.type = ArrayBuffer(1, 2, 3, 4, 5, 6)

scala> b += (7, 8, 9)
res18: b.type = ArrayBuffer(1, 2, 3, 4, 5, 6, 7, 8, 9)

// 添加元素到指定位置
scala> b.insert(3, 30)

scala> b
res20: scala.collection.mutable.ArrayBuffer[Int] = ArrayBuffer(1, 2, 3, 30, 4, 5, 6, 7, 8, 9)

// 移除指定位置的元素
scala> b.remove(1)
res21: Int = 2

scala> b
res22: scala.collection.mutable.ArrayBuffer[Int] = ArrayBuffer(1, 3, 30, 4, 5, 6, 7, 8, 9)

// ArrayBuffer转Array
scala> val barr = b.toArray
barr: Array[Int] = Array(1, 3, 30, 4, 5, 6, 7, 8, 9)

// Array转ArrayBuffer
scala> barr.toBuffer
res25: scala.collection.mutable.Buffer[Int] = ArrayBuffer(1, 3, 30, 4, 5, 6, 7, 8, 9)

// 直接迭代ArrayBuffer
scala> for(i <- ab) println(i)
1
2
3
4
5

// 根据下标，迭代ArrayBuffer
scala> for(i <- 0 until ab.length) println(ab(i))
1
2
3
4
5

// ArrayBuffer求和
scala> val sum = ab.sum
sum: Int = 15

// ArrayBuffer求最大值
scala> val max = ab.max
max: Int = 5

// Array排序
scala> val abarr = ab.toArray
abarr: Array[Int] = Array(1, 2, 3, 4, 5)

scala> scala.util.Sorting.quickSort(abarr)

scala> abarr
res39: Array[Int] = Array(1, 2, 3, 4, 5)
```

##### 4、Set

Set，代表没有重复元素的集合，分为可变和不可变集合，默认情况下使用的是不可变集合。

```scala
// 创建不可变集合
scala> val set = Set(1, 2, 3)
set: scala.collection.immutable.Set[Int] = Set(1, 2, 3)
// 添加元素失败
scala> set += 4
<console>:13: error: value += is not a member of scala.collection.immutable.Set[Int]
  Expression does not convert to assignment because receiver is not assignable.
       set += 4
           ^
// 生成新的不可变集合
scala> set + 4
res3: scala.collection.immutable.Set[Int] = Set(1, 2, 3, 4)

// 创建可变集合
scala> val set = scala.collection.mutable.Set(1, 2, 3)
<console>:11: error: object collect is not a member of package scala
       val set = scala.collect.mutable.Set(1, 2, 3)
// 添加元素成功
scala> set += 4
res4: set.type = Set(1, 2, 3, 4)

// 迭代集合
scala> for(i <- set) println(i)
1
2
3
4
```

###### ① HashSet

类似于 Java#HashSet，元素无序，分为可变集合、以及不可变集合。

```scala
// HashSet，既是Class可以new，又是Object可以不new
scala> val s = new scala.collection.mutable.HashSet[Int]()
s: scala.collection.mutable.HashSet[Int] = Set()

scala> s += 1
res6: s.type = Set(1)

scala> s += 2
res8: s.type = Set(1, 2)

scala> s += 3
res9: s.type = Set(1, 2, 3)

scala> s += (5)
res10: s.type = Set(1, 5, 2, 3)

scala> s.+=(6)
res11: s.type = Set(1, 5, 2, 6, 3)
```

###### ② LinkedHashSet

类似于 Java#LinkedHashSet，元素按添加顺序排序，只有可变集合。

```scala
scala> val s = scala.collection.mutable.LinkedHashSet[Int]()
s: scala.collection.mutable.LinkedHashSet[Int] = Set()

scala> s += 1
res12: s.type = Set(1)

scala> s += 2
res13: s.type = Set(1, 2)

scala> s += (3, 4, 5)
res14: s.type = Set(1, 2, 3, 4, 5)
```

###### ③ SortedSet

类似于 Java#SortedSet，元素按字典序排序，分为可变集合、以及不可变集合。

```scala
scala> val s = scala.collection.mutable.SortedSet[Int]();
s: scala.collection.mutable.SortedSet[Int] = TreeSet()

scala> s += 1
res15: s.type = TreeSet(1)

scala> s += 2
res16: s.type = TreeSet(1, 2)

scala> s += (3, 4, 5, 1, 2)
res17: s.type = TreeSet(1, 2, 3, 4, 5)
```

##### 5、Map

Map 是一种可迭代的键值对 key-value 结构，分为可变和不可变集合，默认使用的不可变 Map。

```scala
// 默认创建不可变集合
scala> val ages = Map("jack" -> 30, "tom" -> 25, "jessic" -> 23)
ages: scala.collection.immutable.Map[String,Int] = Map(jack -> 30, tom -> 25, jessic -> 23)

scala> val ages = Map(("jack", 30), ("tom", 25), ("jessic", 23))
ages: scala.collection.immutable.Map[String,Int] = Map(jack -> 30, tom -> 25, jessic -> 23)

scala> val ages = Map(("jack" -> 30), ("tom" -> 25), ("jessic" -> 23))
ages: scala.collection.immutable.Map[String,Int] = Map(jack -> 30, tom -> 25, jessic -> 23)

// 获取存在的key
scala> ages("jack")
res0: Int = 30

// 获取不存在的key，会报错
scala> ages("jack1")
java.util.NoSuchElementException: key not found: jack1
  at scala.collection.immutable.Map$Map3.apply(Map.scala:242)
  ... 28 elided

// getOrElse不会报错
scala> val age = ages.getOrElse("jack1", 0)
age: Int = 0

scala> val age = ages.getOrElse("jack", 0)
age: Int = 30

// 不可变集合添加、修改元素报错
scala> ages("jack") = 31
<console>:13: error: value update is not a member of scala.collection.immutable.Map[String,Int]
       ages("jack") = 31

// 创建可变集合
scala> val ages = scala.collection.mutable.Map("jack" -> 30, "tom" -> 25, "jessic" -> 23)
ages: scala.collection.mutable.Map[String,Int] = Map(jessic -> 23, jack -> 30, tom -> 25)

// 可变集合添加、修改元素成功
scala> ages("jack") = 31

scala> ages("jack")
res5: Int = 31

scala> ages("jack1") = 32

scala> ages("jack1")
res7: Int = 32

// 可变集合批量添加元素
scala> ages += ("hehe" -> 35, "haha" -> 40)
res8: ages.type = Map(hehe -> 35, jessic -> 23, jack -> 31, jack1 -> 32, tom -> 25, haha -> 40)

// 可变集合批量删除元素
scala> ages -= ("hehe", "haha")
res9: ages.type = Map(jessic -> 23, jack -> 31, jack1 -> 32, tom -> 25)

// 迭代Map集合
scala> for((key, value) <- ages) println(key + ": " + value)
jessic: 23
jack: 31
jack1: 32
tom: 25

// 迭代Map#key集合
scala> for(key <- ages.keySet) println(key)
jessic
jack
jack1
tom

// 迭代Map#value集合
scala> for(value <- ages.values) println(value)
23
31
32
25
```

###### ① HashMap

类似于 Java#HashMap，元素无序，分为可变集合、以及不可变集合。

```scala
scala> val ages = scala.collection.immutable.HashMap("b" -> 30, "a" -> 15, "c" -> 20, "g" -> 39)
ages: scala.collection.immutable.HashMap[String,Int] = Map(a -> 15, b -> 30, g -> 39, c -> 20)

scala> val ages = scala.collection.mutable.HashMap("b" -> 30, "a" -> 15, "c" -> 20, "g" -> 39)
ages: scala.collection.mutable.HashMap[String,Int] = Map(b -> 30, g -> 39, a -> 15, c -> 20)
```

###### ② LinkedHashMap

类似于 Java#LinkedHashMap，元素按添加顺序排序，只有可变集合。

```java
scala> val ages = scala.collection.immutable.LinkedHashMap("b" -> 30, "a" -> 15, "c" -> 20, "g" -> 39)
<console>:11: error: object LinkedHashMap is not a member of package scala.collection.immutable
       val ages = scala.collection.immutable.LinkedHashMap("b" -> 30, "a" -> 15, "c" -> 20, "g" -> 39)
                                             ^
                                             
// LinkedHashMap只有可变集合
scala> val ages = scala.collection.mutable.LinkedHashMap("b" -> 30, "a" -> 15, "c" -> 20, "g" -> 39)
ages: scala.collection.mutable.LinkedHashMap[String,Int] = Map(b -> 30, a -> 15, c -> 20, g -> 39)
```

###### ③ SortedMap

类似于 Java#SortedMap，元素按字典序排序，分为可变集合、以及不可变集合。

```scala
scala> val ages = scala.collection.immutable.SortedMap("b" -> 30, "a" -> 15, "c" -> 20, "g" -> 39)
ages: scala.collection.immutable.SortedMap[String,Int] = Map(a -> 15, b -> 30, c -> 20, g -> 39)

scala> val ages = scala.collection.mutable.SortedMap("b" -> 30, "a" -> 15, "c" -> 20, "g" -> 39)
ages: scala.collection.mutable.SortedMap[String,Int] = TreeMap(a -> 15, b -> 30, c -> 20, g -> 39)
```

#### 9）函数

##### 1、标准定义

1. 定义函数需要使用 `def` 关键字。
2. 必须指定所有入参的类型，但出参可不指定，因为可以根据返回值自行推断类型。
3. 函数最后一行就是整个函数的返回值，所以无需使用 `return` 表示。

```scala
// 无返回值函数
scala> def sayHello(name: String) = println("hello " + name)
sayHello: (name: String)Unit

scala> sayHello("scala")
hello scala

// 带返回值函数，可以不显示声明返回值类型，scala会根据返回值，自动判断类型
// def sayHello(name: String, age: Int) : Int = {
def sayHello(name: String, age: Int) = {
  println("my name is " + name + ", age is " + age)
  age
}

// Exiting paste mode, now interpreting.

sayHello: (name: String, age: Int)Int

scala> sayHello("scala", 2022)
my name is scala, age is 2022
res45: Int = 2022
```

##### 2、默认参数

没传参数时，则使用默认值

```scala
scala> def sayHello(fName: String, mName: String = "mid", lName: String = "last") = fName + ", " + mName + ", " + lName
sayHello: (fName: String, mName: String, lName: String)String

scala> sayHello("zhang")
res46: String = zhang, mid, last

scala> sayHello("zhang", "san")
res47: String = zhang, san, last

scala> sayHello("zhang", "san", "2")
res48: String = zhang, san, 2
```

##### 3、带名参数

使用时函数时，指定入参的名称，这样就不用保持参数的顺序了

 ```scala
scala> sayHello(fName="jack", lName="Tom", mName="Mick")
res2: String = jack, Mick, Tom
 ```

##### 4、可变参数

传入的参数个数动态可变，类似于 Java#... 不定长参数

```scala
scala> :paste
// Entering paste mode (ctrl-D to finish)

def sum(nums: Int*) = {
  var res = 0
  for(num <- nums) res += num
  res
}

// Exiting paste mode, now interpreting.

sum: (nums: Int*)Int

scala> sum(1)
res5: Int = 1

scala> sum(1, 2)
res6: Int = 3

scala> sum(1, 2, 3)
res7: Int = 6
```

##### 5、过程

如果定义函数时，没有指定 `=` 连接函数体，或者显示声明返回值为 `Unit`，那么这样的函数称之为过程。

```scala
// 函数(有返回值)
def sayHello(name: String) = "hello," + name
def sayHello(name: String) : String = "hello," + name

// 过程(无返回值)
def sayHello(name: String) { "hello," + name }
def sayHello(name: String) : Unit = "hello," + name
```

##### 6、函数式编程

1. Scala 和 Java 一样，既是一门面向对象的语言，也是一门面向过程的语言。
2. 但 Scala 中，函数是面向过程的体现，可以脱离类和对象，称之为函数式编程。
3. 在 Scala 中，函数可以独立定义、独立存在、可以直接把函数作为值赋值给变量（但此时函数后面必须加上空格和下划线）。

```scala
scala> def sayHello(name: String) {println("hello " + name)
     | }
sayHello: (name: String)Unit

scala> val sayHelloFunc = sayHello
sayHello   sayHelloFunc

// 直接调用报错
scala> val sayHelloFunc = sayHello
<console>:12: error: missing argument list for method sayHello
Unapplied methods are only converted to functions when a function type is expected.
You can make this conversion explicit by writing `sayHello _` or `sayHello(_)` instead of `sayHello`.
       val sayHelloFunc = sayHello
                          ^

// 必须后面加空格和下划线
scala> val sayHelloFunc = sayHello _
sayHelloFunc: String => Unit = $Lambda$1152/922145372@1bfa5a13

scala> sayHelloFunc("scala")
hello scala
```

##### 7、匿名函数

1. 在 Scala 中，函数可以不命名，这种函数称之为匿名函数。
2. 匿名函数的具体语法格式为，`（参数名:参数类型）=> 函数体`。

```scala
scala> val sayHelloFunc = (name: String) => println("hello " + name)
sayHelloFunc: String => Unit = $Lambda$1172/1393284987@58a9e64d

scala> sayHelloFunc("scala")
hello scala
```

##### 8、高阶函数

1. 在 Scala 中，可以直接把函数作为参数，传入另一个函数中，像这种，把函数作为参数的函数，称之为高阶函数。
2. 高阶函数可以自动推断出，传入函数的入参类型，所以，调用时可以省略传入函数的入参类型。
3. 对于传入函数只有一个入参时，可以省略传入函数的小括号。

```scala
scala> val sayHelloFunc = (name: String) => println("hello " + name)
sayHelloFunc: String => Unit = $Lambda$1039/1215795615@64ae105d

// 高阶函数体，没指定括号，则报错
scala> def greetting(func: (String) => Unit, name: String) func(name)
<console>:1: error: '=' expected but identifier found.
       def greetting(func: (String) => Unit, name: String) func(name)
                                                           ^

scala> def greetting(func: (String) => Unit, name: String) { func(name) }
greetting: (func: String => Unit, name: String)Unit

scala> greetting(sayHelloFunc, "scala")
hello scala

// 匿名函数 + 高阶函数
scala> greetting((name: String) => println("hello " + name), "scala")
hello scala

// 省略匿名函数的入参类型
scala> greetting((name) => println("hello " + name), "scala")
hello scala

// 匿名函数只有一个入参时，可以省略入参的小括号
scala> greetting(name => println("hello " + name), "scala")
hello scala
```

##### 9、常用高阶函数

| 高阶函数   | 作用                                                         |
| ---------- | ------------------------------------------------------------ |
| map        | 给一个元素，处理一个，最后返回一个，即一进一出               |
| flatMap    | 给一个元素，处理一个，最后返回一个、或者多个元素，即一进多出 |
| foreach    | 给一个元素，处理一个，但没有返回值，即迭代操作               |
| filter     | 给一个元素，执行一次判断逻辑，如果判断返回 true，则保留该元素，否则会把元素过滤掉 |
| reduceLeft | 从左侧位置开始，对元素进行 reduce 操作，类似于 MapReduce#reduce |

```scala
// map + 匿名函数
scala> Array(1,2,3,4,5).map(num => num * 2)
res4: Array[Int] = Array(2, 4, 6, 8, 10)
// map + 匿名函数 + 简写（通配符"_"）
scala> Array(1,2,3,4,5).map(_ * 2)
res5: Array[Int] = Array(2, 4, 6, 8, 10)

// flatMap + 匿名函数
scala> Array("hello you", "hello me").flatMap(line => line.split(" "))
res6: Array[String] = Array(hello, you, hello, me)
// flatMap + 匿名函数 + 简写（通配符"_"）
scala> Array("hello you", "hello me").flatMap(_.split(" "))
res7: Array[String] = Array(hello, you, hello, me)

// foreach + 匿名函数
scala> Array(1,2,3,4,5).map(_ * 2).foreach(num => println(num))
2
4
6
8
10
// foreach + 匿名函数 + 简写（通配符"_"）
scala> Array(1,2,3,4,5).map(_ * 2).foreach(println (_))
2
4
6
8
10
scala> Array(1,2,3,4,5).map(_ * 2).foreach(println _)
2
4
6
8
10

// filter + 匿名函数
scala> Array(1,2,3,4,5).filter(num => num % 2 == 0)
res11: Array[Int] = Array(2, 4)
// filter + 匿名函数 + 简写（通配符"_"）
scala> Array(1,2,3,4,5).filter(_ % 2 == 0)
res12: Array[Int] = Array(2, 4)

// reduceLeft + 匿名函数: (((1 + 2) + 3) + 4) + 5
scala> Array(1,2,3,4,5).reduceLeft((t1, t2) => t1 + t2)
res13: Int = 15
// reduceLeft + 匿名函数 + 简写（通配符"_"）: (((1 + 2) + 3) + 4) + 5
scala> Array(1,2,3,4,5).reduceLeft(_ + _)
res14: Int = 15
// reduceRight + 匿名函数 + 简写（通配符"_"）: 1 + (2 + (3 + (4 + 5)))
scala> Array(1,2,3,4,5).reduceRight(_ + _)
res15: Int = 15

// 单词计数，第一种实现
scala> val lines = Array("abc", "dec", "eaea", "qq")
lines: Array[String] = Array(abc, dec, eaea, qq)
scala> lines.flatMap(_.split(" ")).map((_, 1)).map(_._2).reduceLeft(_ + _)
res4: Int = 4

// 单词计数，第二种实现
scala> lines.flatMap(_.split(" ")).map(1).reduceLeft(_ + _)
<console>:13: error: type mismatch;
 found   : Int(1)
 required: String => ?
       lines.flatMap(_.split(" ")).map(1).reduceLeft(_ + _)
scala> lines.flatMap(_.split(" ")).map(word => 1).reduceLeft(_ + _)
```

#### 10）修饰符

##### 1、lazy

如果把一个变量声明为 `lazy`，那么只有在第一次使用该变量时，变量对应的表达式才会进行计算。

```scala
scala> lazy val sum = 1 + 2
sum: Int = <lazy>

scala> sum
res8: Int = 3
```

#### 11）类

定义类，和 Java 一样，都是使用 `class` 关键字，然后使用 `new` 关键字来创建对象。

```scala
scala> :paste
// Entering paste mode (ctrl-D to finish)

class Person {
var name = "scala"

// 如果定义时指定了"()", 那么在调用时可写可不写"()"
def sayHello() {
println("hello " + name)
}

// 如果定义时没指定"()", 那么在调用时必须不写"()"
def get = name
}

// Exiting paste mode, now interpreting.

defined class Person

scala> val p = new Person()
p: Person = Person@2506b881

scala> p.sayHello()
hello scala

scala> p.sayHello
hello scala

scala> p.get
res3: String = scala

scala> p.get()
<console>:13: error: not enough arguments for method apply: (index: Int)Char in class StringOps.
Unspecified value parameter index.
       p.get()
```

##### 1、主 Constructor

主构造函数，参数与类名放在一起，代码则在类中，没有定义在任何 `def` 方法、或者代码块中的代码，都是属于主构造函数代码。

```java
scala> :paste
// Entering paste mode (ctrl-D to finish)

class Student(val name: String, val age: Int) {
	println("name is: " + name + ", age is: " + age)
}

// Exiting paste mode, now interpreting.

defined class Student

scala> new Student("zs", 19)
name is: zs, age is: 19
res5: Student = Student@699bb304

// 带默认值的主构造函数
scala> :paste
// Entering paste mode (ctrl-D to finish)

class Student(val name: String = "jack", val age: Int = 20) {
println("name is: " + name + ", age is: " + age)
}

// Exiting paste mode, now interpreting.

defined class Student

scala> new Student
name is: jack, age is: 20
res6: Student = Student@4e1104f4
    
scala> new Student("zs", 19)
name is: zs, age is: 19
res7: Student = Student@156f0281
    
scala> new Student("zs")
name is: zs, age is: 20
res8: Student = Student@fbdba1c
```

##### 2、辅助 Constructor

1. 同一个类可以定义多个辅助构造函数，相当于构造函数的重载。
2. 辅助构造函数之间，可以相互调用，但第一行必须调用主构造函数。

```scala
scala> :paste
// Entering paste mode (ctrl-D to finish)

class Student {
    var name = "jack"
    var age = 10

	def this(name: String) {
    	this()
    	this.name = name
	}

	def this(name: String, age: Int) {
    	this(name)
    	this.age = age
    }
}

// Exiting paste mode, now interpreting.

defined class Student
warning: previously defined object Student is not a companion to class Student.
Companions must be defined together; you may wish to use :paste mode for this.

scala> new Student
res11: Student = Student@4ce1ab4c

scala> new Student("tom")
res10: Student = Student@664c411d

scala> new Student("tom", 30)
res12: Student = Student@2fbc704a
```

##### 3、case class

1. case class，样例类，类似于 Java#javabean，在 Scala 中，只需要定义 field，Scala 就会自动提供 getter 和setter 方法（其实就是访问成员变量，并没有 get/set 方法），无需自行指定 method。
2. case class 主构造函数，接收的参数，通常不需要使用 var 或者 val 来修饰，Scala 默认会自动使用 val 来修饰。
3. 同时，Scala 还会自动为 case calss 定义对应的 object 伴生对象，以及实现它的 `apply` 方法，在该方法中，接收主构造函数相同的参数，并返回 case calss 对象。

```scala
// case class相当于快速构建class+object
class Person
case class Teacher(name: String, sub: String) extends Person
case class Student(name: String, cla: String) extends Person
```

##### 4、Option

1. Option，有两种值，一种是 Some，表示有值；另一种是 None，表示没值。
2. Option，通常用于模式匹配中，用于判断某个变量有值还是没值，这比 if + null 更加简洁明了。

```scala
scala> val ages = Map("a" -> 1, "b" -> 2)
ages: scala.collection.immutable.Map[String,Int] = Map(a -> 1, b -> 2)

scala> ages("a")
res37: Int = 1

scala> ages.get("a")
res38: Option[Int] = Some(1)

scala> ages("a1")
java.util.NoSuchElementException: key not found: a1
  at scala.collection.immutable.Map$Map2.apply(Map.scala:184)
  ... 28 elided

scala> ages.get("a1")
res40: Option[Int] = None
```

#### 12）对象

1. 对象，既可以使用 `new` 关键字创建。
2. 也可以直接定义，Scala 中叫做 object，相当于 `class` 的单例，此时通常在里面存放一些静态的 field 或者 method，相当于 Java 中的静态类、静态属性、和静态方法。
3. 注意，object 中不能定义带参数的构造函数，只允许空参的构造函数，也不能 `new`，但可以直接使用。
4. 只有在第一次调用 object 方法时，才会执行 object 的空参构造函数一次。

```scala
scala> :paste
// Entering paste mode (ctrl-D to finish)

object Person {

  var age = 1
  println("this Person is Object!")

  def getAge = age
}

// Exiting paste mode, now interpreting.

defined object Person
warning: previously defined class Person is not a companion to object Person.
Companions must be defined together; you may wish to use :paste mode for this.

scala> Person.age
this Person is Object!
res13: Int = 1

scala> Person.age
res14: Int = 1
```

##### 1、伴生对象

1. object 和某个 class 同名，那么就称这个 object 是那个 class 的伴生对象，那个 class 是这个 object 的伴生类。
2. 注意，伴生类和伴生对象，必须存放在一个 `.scala` 文件中。
3. 伴生类和伴生对象，最大的特点在于，可以相互访问 `private field` 私有属性。

```scala
scala> :paste
// Entering paste mode (ctrl-D to finish)

object Person {
	private val fdNum = 1
	def getFdNum = fdNum
}

class Person(val name : String, val age: Int) {
	def sayHello = println("name: " + name + ", age: " + age + ", fdNum: " + Person.fdNum)
}

// Exiting paste mode, now interpreting.

defined object Person
defined class Person

scala> Person.fdNum
<console>:13: error: value fdNum is not a member of object Person
       Person.fdNum
              ^

scala> Person.getFdNum
res24: Int = 1

scala> new Person("tom", 20).sayHello
name: tom, age: 20, fdNum: 1
```

##### 2、apply 方法

1. apply，是 object 中非常重要的一个特殊方法，通常是在伴生对象中实现 apply 方法，然后在 apply 方法体中实现构造伴生类对象的功能。
2. 这样，后期使用伴生类对象时，不需要再 `new class` 方法，而可以直接使用 `class` 的方式，隐式调用伴生对象的 apply 方法，从而使得创建对象更加简洁。

```scala
scala> :paste
// Entering paste mode (ctrl-D to finish)

class Person(val name: String) {
println("my name is: " + name)
}

object Person {
def apply(name: String) = {
println("apply exec...")
new Person(name)
}
}

// Exiting paste mode, now interpreting.

defined class Person
defined object Person

// 正常构造
scala> new Person("tom")
my name is: tom
res0: Person = Person@307cf964

// apply省略new关键字构造
scala> Person("tom")
apply exec...
my name is: tom
res1: Person = Person@c497a55
```

##### 3、main 方法

Scala#main 方法，必须定义在 object 里面，不能定义在 class 中。

```scala
/**
  * 测试main方法
  */
object MainDemo {
  def main(args: Array[String]) : Unit = {
    println("hello scala!")
  }
}
```

#### 13）接口

1. trait，接口，可以定义抽象方法，然后使用 `extends` 实现该接口（无论是类还是接口， 继承或者实现都是使用 `extends`），此时必须实现 trait 中所有抽象方法。
2. 类似于 Java，Scala 也是不支持类的多继承，但支持接口 trait 的多继承，使用 `with` 关键字即可。

```scala
package com.bigdata.demo

/**
  * 测试接口多继承
  * 
  * I can sing
  * I can dance
  * Hello jack
  * Hello tom
  * My name is tom, your name is jack 
  * My name is jack, your name is tom 
  * 
  */
object PersonDemo {
  def main(args: Array[String]): Unit = {
    val tom = new Person("tom")
    val jack = new Person("jack")

    tom.saySkill("sing")
    jack.saySkill("dance")

    tom.sayHello(jack.name)
    jack.sayHello(tom.name)

    tom.makeFriends(jack)
    jack.makeFriends(tom)
  }
}

trait HelloTrait {
  def sayHello(name: String)
}

trait MakeFriendsTrait {
  def makeFriends(p: Person)
}

trait SkillTrait {
  def saySkill(skill: String)
}

class Person(val name: String) extends HelloTrait with MakeFriendsTrait with SkillTrait {
  override def sayHello(name: String): Unit = {
    println("Hello " + name)
  }

  override def makeFriends(p: Person): Unit = {
    printf("My name is %s, your name is %s \n", name, p.name)
  }

  override def saySkill(skill: String): Unit = {
    println("I can " + skill)
  }
}
```

#### 14）高级特性

##### 1、模式匹配

1. Scala 没有 `switch case` 语法，但提供了 `match case` 模式匹配。
2. Java 中，`switch case` 只能匹配变量值，而 Scala 中，`match case` 可以匹配变量类型、集合元素、有值没值等多种情况。
3. Scala `match case`，分支使用完后，会自动退出，无需使用 `break` 关键字来退出。

```scala
// 1、match case + 变量值匹配
scala> :paste
// Entering paste mode (ctrl-D to finish)

def demo1(day: Int) {
  day match {
      case 1 => println("Mon")
      case 2 => println("Tue")
      case 3 => println("Wed")
      case _ => println("None")
  }
}

// Exiting paste mode, now interpreting.

demo1: (day: Int)Unit

scala> demo1(1)
Mon

scala> demo1(2)
Tue

scala> demo1(3)
Wed

scala> demo1(4)
None

// 2、match case + 类型匹配
scala> import java.io._
import java.io._

scala> :paste
// Entering paste mode (ctrl-D to finish)

def processException(e: Exception) {
	e match {
		case e1: FileNotFoundException => println("FileNotFoundException " + e1)
		case e2: IOException => println("IoException " + e2)
		case _: Exception => println("Exception " + _)
	}
}

// Exiting paste mode, now interpreting.

processException: (e: Exception)Unit

scala> processException(new FileNotFoundException())
FileNotFoundException java.io.FileNotFoundException

scala> processException(new IOException())
IoException java.io.IOException

scala> processException(new Exception())
$line20.$read$$iw$$iw$$iw$$iw$$$Lambda$1361/1173121971@219c32e3

// 3、match case + class类型匹配
class Person
class Teacher(val name: String, val sub: String) extends Person
class Student(name: String, cla: String) extends Person

def check(p: Person) {
	p match {
		case Teacher(name, sub) => println("Teacher: name is " + name + ", sub is " + sub)
		case Student(name, cla) => println("Student: name is " + name + ", cla is " + cla)
		case _ => println("None")
	}
}

// 4、match case + Option
scala> val ages = Map("jack" -> 18, "tom" -> 30, "jessic" -> 27)
ages: scala.collection.immutable.Map[String,Int] = Map(jack -> 18, tom -> 30, jessic -> 27)

scala> :paste
// Entering paste mode (ctrl-D to finish)

def getAge(name: String) {
val age = ages.get(name)
	age match {
		case Some(age) => println("your age is " + age)
		case None => println("none")
	}
}

// Exiting paste mode, now interpreting.

getAge: (name: String)Unit

scala> getAge("jack")
your age is 18

scala> getAge("hehe")
none
```

##### 2、隐式转换

1. 隐式转换，Scala 允许手动指定将某种类型的对象，转换成其他类型的对象，可以增强一个普通类的功能，本质上就是在上下文碰到隐式转换函数，则自动转换调用其中的代码，从而实现增强。
2. 隐式转换，其核心是定义 `implicit conversion function` 隐式转换函数，该语法与普通函数的区别是，用 `implicit` 开头，且最好要定义返回值类型。
3. Scala 会自动使用，伴生对象或者作用域中，的隐式转换函数，其他情况则需要使用 `import` 语句，引入某个包下的隐式转换函数，建议在需要时才进行隐式转换，以缩小隐式转换的作用域。

```scala
scala> :paste
// Entering paste mode (ctrl-D to finish)

class cat(val name: String) {
	def catchMouse() {
		println(name + " catch mouse")
	}
}

class dog(val name: String)

// Exiting paste mode, now interpreting.

defined class cat
defined class dog

// 执行使用，则报错
scala> val d = new dog("d1")
d: dog = dog@9b2dc56

scala> d.catchMouse()
<console>:13: error: value catchMouse is not a member of dog
       d.catchMouse()
         ^

// 上下文添加隐式转换函数后，则成功调用
scala> :paste
// Entering paste mode (ctrl-D to finish)

implicit def object2Cat(obj: Object): cat = {
    if(obj.getClass == classOf[dog]) {
    	val dog = obj.asInstanceOf[dog]
    	new cat(dog.name)
    }
    else Nil
}

// Exiting paste mode, now interpreting.

scala> d.catchMouse()
d1 catch mouse
```

#### 15）异常捕捉

```scala
scala> :paste
// Entering paste mode (ctrl-D to finish)

try {
	val lines = scala.io.Source.fromFile("D://notFile.txt").mkString
} catch {
    // catch块中，match case无需再写match
	case ex: FileNotFoundException => println("FileNotFoundException " + ex)
	case ex: IOException => println("IoException " + ex)
	case ex: Exception => println("Exception " + ex)
}

// Exiting paste mode, now interpreting.

FileNotFoundException java.io.FileNotFoundException: D:\notFile.txt (系统找不到指定的文件。)
```

