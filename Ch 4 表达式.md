# Ch 4 表达式

## 4.1 基础

### 4.1.1 基本概念

#### 组合运算符和运算对象

#### 运算对象转换

#### 重载运算符

#### 左值和右值

两篇文章明白左值和右值：

https://www.internalpointers.com/post/understanding-meaning-lvalues-and-rvalues-c

https://www.internalpointers.com/post/c-rvalue-references-and-move-semantics-beginners

一些注意：

- 取地址符作用域一个左值运算对象，返回一个指向该运算对象的指针，这个指针是一个右值。
- 内置解引用运算符、下标运算符、迭代器解引用运算符、string和vector的下表运算符的求职结果都是左值。
- 赋值运算符需要一个非常量的左值作为其左侧运算对象，得到的结果也仍然是一个左值。
- 内置类型和迭代器的递增递减运算符作用于左值运算对象。

使用decltype的时候，如果作用于求值结果是左值的表达式，则得到一个引用类型。如果作用于求值结果为右值的表达式吗，则得到此值的类型。

## 4.2 算数运算符

## 4.3 逻辑和关系运算符

## 4.4 赋值运算符

## 4.5 递增和递减运算符

## 4.6 成员访问运算符

## 4.7 条件运算符

## 4.8 位运算符

## 4.9 sizeof运算符

## 4.10 逗号运算符

## 4.11 类型转换

#### 何时发生类型转换：

- 在大多数表达式中，比`int`类型小的整型值首先提升为较大的整数类型。
- 在条件中，非bool值转换成bool类型。
- 初始化过程中，初始值转换成变量的类型；在赋值语句中，右侧运算对象转换成左侧运算对象的类型。
- 如果算数运算或者关系运算的运算对象有多重类型，那么需要转换成同一种类型。
- 函数调用的时候也会发生类型转换。

### 4.11.1 算数转换

###  4.11.2 其他隐式类型转换

- 数组转换成指针；
- 指针的转换：常量数值0或者字面值nullptr能转换成任意指针类型；指向任意非常量的指针能转换成void\*；指向任意对象的指针能转换成const void\*；
- 转换成布尔类型；
- 转换成常量：允许将指向非常量类型的指针转换成指向相应的常量类型的指针，对于引用也是这样。也就是说，如果T是一种类型，那么我们就可以将T\*隐式转换成const T\*，也可以将T&隐式转换成const T&。
- 类类型定义的转换；

### 4.11.3 显式转换

#### 命名的强制类型转换

```cc
cast-name<type>(expression);
```

其中，type是转换的目标类型而expression是要转换的值。如果type是引用类型，则结果是左值。

cast-name是static_cast，dynamic_cast，const_cast，reinterpret_cast中的一种。

dynamic_cast支持运行时类型识别。

## static_cast

任何具有名且定义的类型转换，只有不包含底层const，即可用static_const。

例如：

```cc
double slope = static_cast<double>(j) / i;
```

例如：

```cc
double d = 0;
void* p = &d; // 正确，任何非常量对象的地址都能存入void*
double* dp = static_cast<double*>(p);
```

#### const_cast

const_cast只能改变运算对象的底层const；

```cc
const char c = 'a';
const char* pc = &c;
char* p = const_cast<char*>(pc); // 正确，但是通过p写值是未定义的行为
```

对于将常量对象转换成非常量对象的行为，我们一般称其为“去掉const性质”（cast away the const）。

const_cast通常用于有函数重载的上下文中。

#### reinterpret_cast

reinterpret_cast通常为运算对象的位模式提供较低层次上的重新解释。

例如：

```cc
int* ip;
char* pc = reinterpret_cast<char*> (ip);
```

## 4.12 运算符优先级表

