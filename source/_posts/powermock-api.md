---
title: 「Testing」测试框架 PowerMock 使用手册
date: 2020-06-16 14:19:10
tags:
	- testing
	- PowerMock
	- Android
categories:
	- Testing
---



**简述**：这篇文章主要是列出 Android 测试框架 `PowerMock` 的常用 `API` 和一些注解。其中某些 `API` 其实并没有用过，所以不能保证全文毫无错误。在此之前，建议先阅读一些入门文章，了解单元测试是什么和为什么做单元测试。

<!-- more -->

<br />



# 1 注解

## 1.1 [JUnit] RunWith

> When a class is annotated with `@RunWith` or extends a class annotated with `@RunWith`, JUnit will invoke the class it references to run the tests in that class instead of the runner built into JUnit. We added this feature late in development. While it seems powerful we expect the runner API to change as we learn how people really use it. Some of the classes that are currently internal will likely be refined and become public.

简单地说，JUnit 会使用 `@RunWith` 中的 Runner 类，代替 JUnit 的内建 Runner 来运行测试。



参考

- [Annotation Type RunWith](https://junit.org/junit4/javadoc/latest/org/junit/runner/RunWith.html)



## 1.2 [PowerMock] PrepareForTest

> Tells PowerMock to prepare certain classes for testing. Classes needed to be defined using this annotation are typically those that needs to be byte-code manipulated. This includes final classes, classes with final, private, static or native methods that should be mocked and also classes that should be return a mock object upon instantiation.

翻译过来，`PrepareForTest` 的作用就是提醒 `PowerMock` 对待测试的类做准备。这些待测试的类可以是：

1. final 类

2. 类中有需要 mock 的 final/private/static/native 方法

3. 希望实例后返回的是某个类的 mock 对象



参考

- [Annotation Type PrepareForTest](https://javadoc.io/doc/org.powermock/powermock-core/1.6.5/org/powermock/core/classloader/annotations/PrepareForTest.html)



## 1.3 [PowerMock] PowerMockIgnore

> This annotation tells PowerMock to defer the loading of classes with the names supplied to [`value()`](https://www.javadoc.io/static/org.powermock/powermock-core/1.7.1/org/powermock/core/classloader/annotations/PowerMockIgnore.html#value--) to the system classloader.
>
> This is useful in situations when you have e.g. a test/assertion utility framework (such as something similar to Hamcrest) whose classes must be loaded by the same classloader as EasyMock, JUnit and PowerMock etc.
>
> Note that the [`PrepareForTest`](https://www.javadoc.io/static/org.powermock/powermock-core/1.7.1/org/powermock/core/classloader/annotations/PrepareForTest.html) and [`PrepareOnlyThisForTest`](https://www.javadoc.io/static/org.powermock/powermock-core/1.7.1/org/powermock/core/classloader/annotations/PrepareOnlyThisForTest.html) will have precedence over this annotation. This annotation will have precedence over the [`PrepareEverythingForTest`](https://www.javadoc.io/static/org.powermock/powermock-core/1.7.1/org/powermock/core/classloader/annotations/PrepareEverythingForTest.html) annotation.

PowerMock 会延迟 system classloader 对在 `PowerMockIgnore` 中的类的加载。如果测试框架的类需要由 JUnit/EasyMock/PowerMock 的类加载器来加载，就可以用上 `PowerMockIgnore` 了。

不过，`PrepareForTest` 和 `PrepareOnlyThisForTest` 的优先级比 `PowerMockIgnore` 高。



参考

- [Annotation Type PowerMockIgnore](https://www.javadoc.io/doc/org.powermock/powermock-core/1.7.1/org/powermock/core/classloader/annotations/PowerMockIgnore.html)



<br />

# 2 构造对象

## 2.1 mock

```java
T object = PowerMockito.mock(Class<? extends Object> type);
```



## 2.2 spy

```java
T object = PowerMockito.spy(T object);
```



参考

- [Mockito – Using Spies](https://www.baeldung.com/mockito-spy)

- [What is the difference between mocking and spying when using Mockito - StackOverflow](https://stackoverflow.com/questions/15052984/what-is-the-difference-between-mocking-and-spying-when-using-mockito)

- [Mockito 的 mock 和 spy](https://www.cnblogs.com/softidea/p/4204389.html)
- [Mockito Mock 與 Spy 的差別](https://matthung0807.blogspot.com/2018/08/mockito-mockspy.html)




<br />

# 3 访问/修改非公有成员

## 3.1 访问非 public 成员

```java
ClassName obj = Whitebox.getInternalState(Object object, String fieldName);
```

**PS：假设类名为 Example，访问静态成员时，object 就是 Example.class。不止是访问非公有成员，其他与 static 有关的操作像是修改静态成员和调用静态方法等，object 都是 Example.class。**



## 3.2 修改非 public 成员

```java
Whiterbox.setInternalState(Object object, String fieldName, Object value);
```




<br />

# 4 调用/执行方法

## 4.1 public 方法

直接调用就行了鸭！



## 4.2 非 public 方法

````java
// 有返回值
ClassName toBeReturned = Whitebox.invokeMethod(Object object, String methodToExecute, Object... arguments);

// 无返回值
Whitebox.invokeMethod(Object object, String methodToExecute, Object... arguments);
````



## 4.3 调用 mock 对象真正的方法

**PS：先设置，再调用。设置 != 调用。另外，对象是通过 mock 得到的**

```java
// public 方法
Powermockito.doCallRealMethod().when(Object object).METHOD(Object... arguments);  // 设置
object.METHOD(Object... arguments);  // 调用

// 非 public 方法
Powermockito.doCallRealMethod().when(Object object, String methodToExecute, Object... arguments);
Whitebox.invokeMethod(Object object, String methodToExecute, Object... arguments);
```




<br />

# 5 设定方法返回值

**PS：此节的对象都是 mock / spy 对象。**

## 5.1 普通方法

1. public 方法

   ```java
   // 有返回值
   PowerMockito.doReturn(Object toBeReturned).when(mockedClass).METHOD(Object.. Arguments);
   
   // 无返回值
   PowerMockito.doNothing().when(T mock).METHOD(Obecject... arguments);
   ```



2. 非 public 方法

   ```java
   // 有返回值
   PowerMockito.doReturn(Object toBeReturned).when(T mock, String methodToExpect, Obecject... arguments);
   
   // 无返回值
   PowerMockito.doNothing().when(T mock, String methodToExpect, Obecject... arguments);
   ```



## 5.2 静态方法

1. public 方法

   ```java
   PowerMockito.mockStatic(Class<?> type, Class<?>... types)
   
   // 有两种方式，第一种更接近正常逻辑
   // 1)
   // 有返回值
   PowerMockito.when(T.STATIC_METHOD(Object... arguments)).thenReturn(Object toBeReturned);
   
   // 2)
   // 有返回值
   PowerMockito.doReturn(...).when(T mock);
   T.STATIC_METHOD(Object... arguments);
   ```
   
2. 非 public 方法

   ```java
   PowerMockito.mockStatic(Class<?> type, Class<?>... types)
   
   // 1)
   // 有返回值
   PowerMockito.when(T.class, String methodToExpect, Obecject... arguments).thenReturn(Object toBeReturned);
   
   // 2)
   // 有返回值
   PowerMockito.doReturn(...).when(T mock);
   Whitebox.invokeMethod(T.class, String methodToExpect, Obecject... arguments);
   
   // 无返回值
   PowerMock.doNothing().when(T.class, String methodToExpect, Object... arguments);
   ```

<br />
对于 spy 对象来说，`when().doReturn()` 仍然会执行指定方法的真实代码逻辑，虽然返回结果被修改了；而 `doReturn().when().METHOD()`，就确实屏蔽了指定方法。

对于 mock 对象来说，这两者就没有区别。



参考

- [Mock/Spy 能力 - zhihu](https://zhuanlan.zhihu.com/p/44239404)
- [Use doReturn to partially mock static method with PowerMockito - StackOverflow](https://stackoverflow.com/questions/7391806/use-doreturn-to-partially-mock-static-method-with-powermockito)
- [doNothing.When for private void method itself is invoking method - StackOverflow](https://stackoverflow.com/questions/55913409/donothing-when-for-private-void-method-itself-is-invoking-method)
- [Mockito: doReturn vs thenReturn](http://sangsoonam.github.io/2019/02/04/mockito-doreturn-vs-thenreturn.html)




<br />

# 6 验证方法

**PS: 先调用外层方法，再验证外层方法内调用过的方法**



## 6.1 普通方法

1. public 方法

   ```java
   // 验证方法的调用和相关参数
   Mockito.verify(? Extends Object mock).METHOD(Object... arguments); 
   
   // 只验证方法的调用，any()，any(T.class)，anyInt()，anyString()
   Mockito.verify(? Extends Object mock).METHOD(Mockito.any());  
   
   // 验证方法没有被调用
   Mockito.verify(? Extends Object mock, Mockito.never()).METHOD(Object... arguments); 
   ```

   PS: 

   - 以下的各类验证，对 Mockito.never() 的使用都可以参照这个例子。
   - [Mockito.ArgumentMatchers.]any() 是一个匹配器，它可以匹配 null 和任意对象。但它不能匹配 Java 的基础类型（int，double，boolean），可以使用 anyInt()，anyDouble()，anyBoolean() 来匹配。



2. 非 public 方法

   ```java
   PowerMockito.verifyPrivate(Object object).invoke("MethodName", Argument);
   ```



## 6.2 静态方法

**PS: 每个待验证的静态方法都需要一个 `verifyStatic()`。**

1. public 方法

   ```java
   PowerMockito.mockStatic(Class<?> type);
   
   ......  // 调用方法
   
   PowerMockito.verifyStatic(Class<T> mockedClass, VerificationMode mode);
   T.STATIC_METHOD(Object... arguments);  // 待验证的静态方法
   ```



2. 非 public 方法

   ```java
   PowerMockito.mockStatic(Class<?> type);
   
   ......  // 调用方法
   
   PowerMockito.verifyStatic(Class<T> mockedClass, VerificationMode mode);
   Whitebox.invokeMethod(Class<T> mockedClass, String methodToExecute, Object... arguments);  // 待验证的静态方法
   ```



参考

- [Mockito Verify Cookbook](https://www.baeldung.com/mockito-verify)
- [PowerMockito.verifyStatic() problems - StackOverflow](https://stackoverflow.com/questions/32976855/powermockito-verifystatic-problems)
- [MockStatic - PowerMockito Docs](https://github.com/powermock/powermock/wiki/MockStatic)




<br />

# 7 抑制方法

首先，对测试类使用类级注解 `@PrepareForTest(ClassWithEvilConstructor.class)`。

然后，分以下三种情况，实现对特定方法的抑制。

## 7.1 普通方法

   ```java
// 有参数
PowerMockito.suppress(MemberMatcher.method(Class<?> declaringClass, String methodName, Class<?>... parameterTypes));

// 无参数
PowerMockito.suppress(MemberMatcher.method(Class<?> declaringClass, String methodName);
   ```



## 7.2 静态方法

先 mock 后 suppress

```java
PowerMockito.mockStatic(Class.class);

// 有参数
PowerMockito.suppress(MemberMatcher.method(Class<?> declaringClass, String methodName, Class<?>... parameterTypes));

// 无参数
PowerMockito.suppress(MemberMatcher.method(Class<?> declaringClass, String methodName);
```



## 7.3 构造方法

```java
// 有参数
PowerMockito.suppress(MemberMatcher.constructor(Class<T> declaringClass, Class<?>... parameterTypes));

// 无参数
PowerMockito.suppress(MemberMatcher.constructor(Class<T> declaringClass));
```



参考
- [Suppress unwanted behavior - GitHub Wiki](https://github.com/powermock/powermock/wiki/Suppress-Unwanted-Behavior)
- [How to suppress and verify private static method calls?](https://stackoverflow.com/questions/16458981/how-to-suppress-and-verify-private-static-method-calls)




<br />

# 8 与 Robolectric 协同

- https://github.com/robolectric/robolectric/wiki/Using-PowerMock



<br />

**以上！**

<br />