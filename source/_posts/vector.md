---
title: 「C++」vector 用法
date: 2018-09-29 19:25:45
tags:
    - C++
    - vector
categories:
    - C++
---

**简述**：`vector`是`C++`标准程序库中的一个类，众多容器（`container`）之一，可视为会自动扩展容量的数组，以循序(`sequential`)的方式维护变量集合，是一种非常实用的容器。

<!-- more -->

<br />
# 1 vector介绍
　　`vector`以模板(泛型)方式实现，可以保存任意类型的变量，包括用户自定义的数据类型，例如：它可以是放置整数（`Int`）类型的 `vector`、也可以是放置字符串（`string`）类型的 `vector`、或者放置用户自定类别（`user-defined class`）的`vector`。
　　`vector`的特色有支持随机存取，在集合尾端增删元素很快，但是在集合中间增删元素比较费时。
　　`vector`类定义于`<vector>`头文件中。
<br />

<br />
# 2 vector类的方法

```C++
#include <vector> 
std:: vector<TYPE> vec;
```
- 访问元素
    - `vec[i]` - 访问索引值为`i`的元素引用。 (索引值从零起算，故第一个元素是`vec[0]`。)
    - `vec.at(i)` - 访问索引值为`i`的元素的引用，以`at()`访问会做数组边界检查，如果访问越界将会抛出一个异常，这是与`operator[]`的唯一差异。
    - `vec.front()` - 回传`vector`第一个元素的引用。
    - `vec.back()` - 回传`vector`最尾端元素的引用。
- 新增或移除元素
    - `vec.push_back()` - 新增元素至`vector`的尾端，必要时会进行存储器配置。
    - `vec.pop_back()` - 删除`vector`最尾端的元素。
    - `vec.insert()` - 插入一个或多个元素至`vector`内的任意位置。
    - `vec.erase()` - 删除`vector`中一个或多个元素。
    - `vec.clear()` - 清空所有元素。
- 获取长度/容量
    - `vec.size()` - 获取`vector`目前持有的元素个数。
    - `vec.empty()` - 如果`vector`内部为空，则传回`true`值。
    - `vec.capacity()` - 获取`vector`目前可容纳的最大元素个数。这个方法与存储器的配置有关，它通常只会增加，不会因为元素被删减而随之减少。
- 重新配置／重置长度
    - `vec.reserve()` - 如有必要，可改变`vector`的容量大小（配置更多的存储器）。在众多的`STL`实例，容量只能增加，不可以减少。
    - `vec.resize()` - 改变`vector`目前持有的元素个数。
- 迭代 (Iterator)
    - `vec.begin()` - 回传一个`iterator`，它指向`vector`第一个元素。
    - `vec.end()` - 回传一个`iterator`，它指向`vector`最尾端元素的下一个位置（请注意：它不是最末元素）。
    - `vec.rbegin()` - 回传一个反向`iterator`，它指向`vector`最尾端元素的。
    - `vec.rend()` - 回传一个`iterator`，它指向`vector`的第一个元素的前一个位置。

<br />
# 3 二维vector

## 3.1 声明

```C++
std::vector<vector<TYPE>> VEC_NAME;  //VEC_NAME: 容器名
```

## 3.2 增添元素

```C++
std::vector<vector<TYPE> VEC_NAME;
VEC_NAME[0].push_back(element); //在容器内的第一子容器末添加值为element的数据，容量size变大
```

## 3.3 删减元素

```C++
std::vector<vector<TYPE>>　VEC_NAME;
VEC_NAME[0].pop_back();  //删减容器内第一子容器的末元素，不返回元素
```

以上！