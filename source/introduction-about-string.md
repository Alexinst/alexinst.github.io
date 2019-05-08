**简述**：类`string`声明于`C++ STL（Standard Template Library)`中的一个头文件`<string>`中，与`char`类型字符串相比较，它显得方便而强大。

<!-- more -->
<br />

# string简介
　　`<string>`是`C++`标准程序库中的一个头文件，定义了C++标准中的字符串的基本模板类`std::basic_string`及相关的模板类实例，这里不进行详细介绍。详细可查看相关[wiki百科](https://zh.wikipedia.org/wiki/String_(C%2B%2B%E6%A0%87%E5%87%86%E5%BA%93))。
　　其中的`string`是以`char`作为模板参数的模板类实例，把字符串的内存管理责任由`string负责`而不是由用户负责，大大减轻了`C`语言风格的字符串的麻烦。
　　`std::basic_string`提供了大量的字符串操作函数，如比较、连接、搜索、替换、获得子串等。并可与`C`语言风格字符串双向转换。`std::basic_string`属于`C++ STL`容器类，用户自定义的类也可以作为它的模板参数，因此也适用`C++ STL Algorithm`库。
<br />

# 声明实例
```C++
......
#include <string>

......
    std::string str;
......
```

# 成员函数
- 构造与析构
    - `string::string`（构造）
    - `string::~string`（析构）
    - **`string::operator =`** - 赋值
    - `string::assign` – 赋值
    - `string::get_allocator` – 获得内存分配器
- 字符访问
    - `string::at` – 访问特定字符，带边界检查
    - **`string::operator []`** – 访问特定字符
    - `string::front` – 访问第一个字符
    - `string::back` – 访问最后一个字符
    - `string::data` – 访问基础数组，`C++11`后与`c_str()`完全相同
    - `string::c_str` – 返回对应于字符串内容的`C`风格零结尾的只读字符串
    - `string::substr` – 以子串构造一个新串；参数为空时取全部源串
- 迭代器
    - **`string::begin`** – 获得指向开始位置的迭代器
    - **`string::end`** – 获得指向末尾的迭代器
    - `string::rbegin` – 获得指向末尾的逆向迭代器
    - `string::rend` – 获得指向开始位置的逆向迭代器
    - `string::cbegin` – 获得指向开始位置的只读迭代器
    - `string::cend` – 获得指向末尾的只读迭代器
    - `string::crbegin` – 获得指向末尾的逆向只读迭代器
    - `string::crend` – 获得指向开始位置的逆向只读迭代器
- 容量
    - `string::empty` – 检查是否为空
    - `string::size` – 返回数据的字符长度
    - **`string::length`** – 返回数据的字符长度，与`size()`完全相同
    - `string::max_size` – 返回可存储的最大的字节容量，在32位`Windows`上大概为 43 亿字节。
    - `string::reserve` – 改变`string`的字符存储容量，实际获得的存储容量不小于`reserve`的参数值。
    - `string::capacity` – 返回当前的字符存储容量
    - `string::shrink_to_fit（C++11 新增）` – 降低内存容量到刚好
- 修改器
    - `string::clear` – 清空内容
    - **`string::insert`** – 插入字符或字符串。目标`string`中的插入位置可用整数值或迭代器表示。如果参数仅为一个迭代器，则在其所指位置插入0 值。
    - `string::erase` – 删除 1 个或 1 段字符
    - **`string::push_back`** – 追加 1 个字符
    - `string::pop_back` – 删除最后 1 个字符，`C++11`标准引入
    - `string::append` – 追加字符或字符串
    - **`string::operator+=`** – 追加，只有一个参数——字符指针、字符或字符串；不像`append()`一样可以追加参数的子串或若干相同字符
    - `string::copy` – 拷贝出一段字符到`C`风格字符数组；有溢出危险
    - `string::resize` – 改变（增加或减少）字符串长度；如果增加了字符串长度，新字符缺省为`0`值
    - `string::swap` – 与另一个`string`交换内容
    - `string::replace` – 替换子串；如果替换源数据与被替换数据的长度不等，则结果字符串的长度发生改变
- 搜索
    - `string::find` – 前向搜索特定子串的第一次出现
    - `string::rfind` – 从尾部开始，后向搜索特定子串的第一次出现
    - `string::find_first_of` – 搜索指定字符集合中任意字符在`*this`中的第一次出现
    - `string::find_last_of` – 搜索指定字符集合中任意字符在`*this`中的最后一次出现
    - `string::find_first_not_of` – `*this`中的不属于指定字符集合的首个字符
    - `string::find_last_not_of` – `*this`中的不属于指定字符集合的末个字符
    - `string::compare` – 与参数字符串比较
