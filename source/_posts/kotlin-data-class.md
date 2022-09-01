---
title: Kotlin 中的 Data Class
date: '2022-07-29 19:08'
theme: smartblue
tags:  
    - Kotlin
comment: false
---

## 数据类

数据类(Data Class) 顾名思义，**用于存放数据的类**，Kotlin 设计者深知开发者需要的数据类应该具备哪些能力，甚至直接为其设计了一个关键字 **data**，于是通过 **data class** 定义的数据类从设计层面就支持自动生成一些例如 equals、hashCode 之类的方法，方便了开发者的同时其中也有一些值得注意的细节，我们把它反编译成 Java 代码便可一目了然
<!--more-->
## data 关键字带来的变化

如下是两个数据类，唯一区别就是一个有 data 关键字，一个没有

```kotlin
class Person(
    val name: String,
    var age: String,
    val isAdult: Boolean,
)

data class DataPerson(
    val name: String,
    var age: String,
    val isAdult: Boolean,
)
```

反编译成 Java 代码的区别对比

```kotlin
public final class Person {
   private final String name;
   private int age;
   private final boolean isAdult;

   public final String getName()
   public final int getAge()
   public final void setAge(int age)
   public final boolean isAdult()

   public Person(@NotNull String name, int age, boolean isAdult) {
      this.name = name;
      this.age = age;
      this.isAdult = isAdult;
   }
}

public final class DataPerson {

   // ... 省略相同部分
   
   public final <Type> componentN()
   public final DataPerson copy(@NotNull String name, int age, boolean isAdult)
   public String toString()
   public int hashCode()
   public boolean equals(@Nullable Object var1)
}
```

可以看到添加了 data 关键字之后编译器为我们自动生成了一些有用的方法

- componentN() [用于解构声明](https://www.kotlincn.net/docs/reference/multi-declarations.html)
- copy()
- toString()
- hashCode()
- equals()

## 派生属性

创建数据类时，有些属性可以通过其它属性计算得到，这时候要**根据情况选择是否使用 getter**

```kotlin
class Person(
    val name: String,
    var age: String,
) {
    val isAdult = age > 17      // ① ❌

    val adult get() = age > 17  // ② ✅
}
```

代码 ① 属性会**被声明为 final 类型并初始化在构造方法中**，代码 ② 属性则会**直接被编译成 get 方法**，如下是对应反编译的 Java 代码

```java
public final class Person {
   private final boolean isAdult;
   @NotNull
   private final String name;
   private int age;
   public final boolean isAdult() {
      return this.isAdult;
   }
   public final boolean getAdult() {
      return this.age > 17;
   }
   @NotNull
   public final String getName() {
      return this.name;
   }
   public final int getAge() {
      return this.age;
   }
   public final void setAge(int var1) {
      this.age = var1;
   }
   public Person(@NotNull String name, int age) {
      Intrinsics.checkNotNullParameter(name, "name");
      super();
      this.name = name;
      this.age = age;
      this.isAdult = this.age > 17;
   }
}
```

可见 Person 被创建之后，如果 age 被改变，isAdult 的结果不会受到影响，因为它在对象初始化的时候就计算出固定结果了。adult 的结果会对应发生预期的变化

```kotlin
fun main() {
    val person = Person(name = "", age = 17)

    person.age = 18
    println("isAdult ${person.isAdult}")  // false
    println("adult ${person.adult}")      // true
}
```

所以，**对于那些根据可变属性的变化而派生出来的属性，需要自定义 getter 而不是直接赋予表达式**。根据不可变属性派生出来的属性则不需要 getter

```kotlin
class Rectangle(val width: Int, val height: Int) {
    val area = this.width * this.height  // ✅
}
```