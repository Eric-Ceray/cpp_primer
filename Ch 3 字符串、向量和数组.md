# Ch 3 字符串、向量和数组

内容：

- 3.1 命名空间的`using`声明
- 3.2 标准库类型`string`
- 3.3 标准库类型`vector`
- 3.4 迭代器介绍
- 3.5 数组
- 3.6 多维数组

## 3.1 命名空间的using声明

例如`std::cin`表示从标准输入中读取内容。作用域操作符`::`的含义是：编译器应该从操作符左边名字所示的作用域寻找右边那个名字。

也可以使用`using`声明，有了`using`声明就无须专门的前缀（`::`）也能使用所需要的名字了；

```cc
using namespace::name;
```

####  每个名字都需要独立的`using`声明

### 头文件不应该包含`using`声明

位于头文件的代码一般来说不应该使用`using`声明；

## 3.2 标准库类型`string`

### 3.2.1 定义和初始化`string`对象

#### 直接初始化和拷贝初始化

C++有几种不同的初始化方式；

拷贝初始化、直接初始化；

### 3.2.2 `string`对象上的操作

#### 读写`string`对象

#### 读取未知数量的`string`类

#### 使用`getline`读取一整行

#### `string`的`empty`和`size`操作

#### `string::size_type`类型

#### 比较`string`对象

#### 为`string`对象赋值

#### 两个`string`对象相加

标准库允许把字符字面量和字符串字面量转换成`string`对象。

但是不能把字面值直接相加。

### 3.2.3 处理`string`对象中的字符

## 3.3 标准库类型`vector`

C++既有类模板，也有函数模板，其中`vector`是一个类模板。

### 3.3.1 定义和初始化`vector`对象

### 3.3.2 向`vector`对象中添加元素

关键概念：`vector`对象能高效增长

#### 向`vector`对象添加元素蕴含的编程假定

如果循环体内部包含有向`vector`对象添加元素的语句，则不能使用==**范围for循环**==；

### 3.3.3 其他`vector`操作

#### 计算`vector`内对象的索引

#### 不能用下标形式添加元素

## 3.4 迭代器介绍

所有标准容器都支持迭代器，但是只有其中少数几种才同时支持下标运算符。

### 3.4.1 使用迭代器

#### 迭代器运算符

#### 将迭代器从一个元素移动到另一个元素

要养成使用迭代器和`!=`的习惯；

#### 迭代器类型

就像不知道`string`和`vector`的`size_type`成员到底是什么类型一样，一般来说我们也不知道迭代器的精确类型。实际上，那些拥有迭代器的标准库类型使用`iterator`和`const_iterator`来表示迭代器的类型；

#### `begin`和`end`运算符

#### 结合解引用和成员访问操作

#### 某些对`vector`对象的操作会使迭代器失效

任何一种可能改变`vector`对象容量的操作，比如`push_back`，都会使该`vector`对象的迭代器失效。

### 3.4.2 迭代器运算

#### 迭代器的算数运算

#### 使用迭代器运算

## 3.5 数组

### 3.5.1 定义和初始化内置数组

数组的维度必须是常量表达式；

和内置变量的类型一样，如果在函数内部设定了某种内置类型的数组，那么默认初始化会令数组含有未定义的值；

### 3.5.2 访问数组元素

### 3.5.3 指针和数组

#### 指针也是迭代器

#### 标准库函数begin和end

#### 指针运算

#### 解引用和指针运算的交互

#### 下标和指针

### 3.5.4 C风格字符串

#### 3.5.5 与旧代码的接口

#### 混用`string`对象和`C`风格字符串

#### 使用数组初始化`vector`对象

## 3.6 多维数组



