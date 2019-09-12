---
title: 「Java」为什么“多态”
date: 2019-09-12 18:53:58
tags:
	- Java
	- polymorphism
	- advantage
	- disadvange
categories:	
	- Java
---

**简述**：多态（ `polymorphism` ）在 `OOP`（ `object-oriented programming`，面向对象编程）很常见，但却不好理解。窝以为窝懂了，后来发现自己原来不太懂。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这一刻，窝还在盲人摸象，希望通过分析多态的优缺点来了解它。我自己摸索着去理解它，所以这些内容不一定是对的。

<!-- more -->

<br />



# 1 多态

简单地说，多态就是：

1. **父类引用**指向**子类对象**。

```java
Animal a = new Cat();  // Animal 是 Cat 的父类。
```

2. 子类要重写（ `Override` ）父类的方法。



如果不了解多态，可以点[**这里**](https://www.zhihu.com/question/30082151)。

<br />

# 2 优点

鉴于参考内容的示例代码非常不错，窝就抄过来用了，见谅见谅。这里是[**出处**](https://www.zhihu.com/question/30082151/answer/120520568?utm_source=com.google.android.apps.docs&utm_medium=social)。

<br />

多态当然是有它的优点，才会被引入的嘛。

1. 通过继承和重写（ `Override` ）非静态方法，对父类进行特定的改写。

   虽然引用变量 `a` 引用的是子类 `Cat` 的对象，但 `a` 能访问的是
   - `Animal` 的成员变量/常量
   - `Animal`  的静态方法
   - `Animal`  中**未被子类 `Cat`重写**的非静态方法
   -  `Cat` 中**重写**的非静态方法。

   简单地说，就是不能访问 `Animal` 中那些被重写的非静态方法。
   
   但是，经过强制转型（ `((Cat) a)` ），就又变了个样，比较魔幻了。看不懂不要紧，看代码就好了。
   
   总结：**父类引用指向子类对象，等于指向一个改变了（在子类中重写的）某些非静态方法的父类的对象**。

```java
public class Test {
    public static void main(String[] args) {
        Animal a = new Cat();
        a.eat();  			// 猫吃饭（被 Cat 重写的非静态方法）
        a.sleep();			// 动物睡觉（ Animal 的静态方法)
        a.run();			// 动物奔跑（没有被 Cat 重写的静态方法）
        System.out.println(a.num);			// 10
        System.out.println(a.age);			// 10

        ((Cat) a).sleep();  // 猫睡觉
        System.out.println(((Cat) a).num);  // 100
        System.out.println(((Cat) a).age);  // 100
    }
}

class Animal {
    public int num = 10;
    public static int age = 10;

    public void eat() {
        System.out.println("动物吃饭");
    }
    public static void sleep() {
        System.out.println("动物睡觉");
    }

    public void run() {
        System.out.println("动物奔跑");
    }
}

class Cat extends Animal{
    public int num = 100;
    public static int age = 100;
    public String name = "Tom";

    @Override
    public void eat() {
        System.out.println("猫吃饭");
    }

    public static void sleep() {
        System.out.println("猫睡觉");
    }

    public void catchMouse() {
        System.out.println("猫抓老鼠");
    }
}
```

2. 复用代码

```java
public void test(Animal a) {
    do something with a.
}
```

调用 `test` 函数时，传入参数可以是 `Animal` 的对象，也可以是 `Animal` 的子类的对象。

不需要针对 `Animal` 子类的对象重载（ `Overload` ）`test` 函数，如 `public void test(Cat c)` 。

3. 其他。。。

   

代码撸得不够多，想不出来了。

<br />



# 3 缺点

- 父类引用指向子类对象，无法通过父类引用访问子类的成员变量与静态方法。

```java
public class Test {
    public static void main(String[] args) {
        Animal a = new Cat();
        
        a.catchMouse();
        System.out.print(a.name);
    }
}
```

输出： 找不到对象的方法 `catchMouse()` 和 成员变量 `name` 。

```
Error:(5, 10) java: cannot find symbol
                         symbol:   method catchMouse()
                         location: variable a of type Animal
Error:(6, 27) java: cannot find symbol
                         symbol:   variable name
                         location: variable a of type Animal
```

原因：父类 `Animal` 中没有方法 `catchMouse()`和成员变量 `name` 。

<br />



# 4 参考

- [JAVA的多态用几句话能直观的解释一下吗？ - zhihu](https://www.zhihu.com/question/30082151)
- [Creating object with reference to Interface -  StackOverflow](https://stackoverflow.com/questions/14997202/creating-object-with-reference-to-interface)
- [Diving Deeper into Polymorphism and its Benefits in Java - developer](https://www.developer.com/java/data/diving-deeper-into-polymorphism-and-its-benefits-in-java.html)

<br />



**以上**