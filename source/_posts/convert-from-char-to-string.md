---
title: 【C++】数据类型 char 转化为 string
date: 2018-09-30 22:00:26
tags:
    - C++
    - conversion
    - char
    - string
categories:
    - C++
---


**简述**：`C++`中数据类型`char`和`string`都可以存放字符，所以存在将`char`类型转化为类型`string`的需求。

<!-- more -->
<br />

# 具体方法
　　话不多说，先上代码。

```C++
#include <iostream>
#include <string>
#include <sstream>  //stringstream出处

int main()
{
    char c = 'a';
    std::string str;
    std::stringstream ss;

    ss << c;
    str = ss.str();
    std::cout << str << std::endl;
    return 0;
}
```

　　这里借助`stringstream`，存储变量`c`中的值，再赋给变量`str`，完成转化。

　　以上！
