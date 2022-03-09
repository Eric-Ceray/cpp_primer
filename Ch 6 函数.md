# Ch 6 函数

## 6.1 函数基础

## 6.2 参数传递

### 6.2.4 数组形参

因为数组是以指针的形式传递给函数的切尺寸。管理指针形参有三种常见技术。

#### 使用标记指定数组长度

例如C风格字符串：

```cpp
void print(const char *p)
{
    if(cp)
        while (*cp)
            cout<<*cp++;
}
```

#### 使用标准库规范

```cpp
void print(const int *beg, const int *end)
{
    //输出beg到end之间（不含end）的所有元素
    while(beg!=end)
        cout<<*beg++<<endl;
}
```

```cpp
int j[2]={0,1};
print(begin(j), end(j));
```

#### 显式传递一个表示数组大小的形参

```cpp
void print(const int ia[], size_t size)
{
    for(size_t i=0;i!=size;i++)
    {
        cout<<ia[i]<<endl;
    }
}
```

```cpp
int j[]={0,1};
print(j, end(j)-begin(j))
```

#### 数组形参和const

#### 数组引用形参

```cpp
void print(int (&arr)[10])
{
    for(auto elem:arr)
        cout<<elem<<endl;
}
```

#### 传递多维数组

```cpp
void print(int (*matrix)[10], int rowSize);
void print(int matrix[][10], int rowSize);
```

### 6.2.5 main：处理命令行选项

```cpp
int main(int argc, char* argv[]) {...}
```

其中第一个形参$argc$​是参数的数量，第二个形参$argv$是一个数组，它的元素是C风格的字符串指针；由于第二个形参是数组，所以$main$函数也可以定义成如下形式：

```cpp
int main(int argc, char** argv) {...}
```

假定$main$​函数位于可执行文件$prog$内，可以向程序传递下面的选项：

```shell
prog -d -o ofile data0
```

实参传递给main函数之后，$argv$​数组的第一个元素指向程序的名字或者一个空字符串，接下来的元素依次传递命令行提供的实参，最后一个指针之后的值为0；

依照上例：

```cpp
argv[0]="prog";
argv[1]="-d";
argv[2]="-o";
argv[3]="ofile";
argv[4]="data0";
argv[5]=0;
```

### 6.2.6 含有可变形参的函数

C++11提供两种方式来编写能处理不同数量的实参的函数。

- 若所有实参类型相同：传递一个名为initializer_list的标准库类型；
- 若有实参类型不同：一种特殊的函数——可变参数模版；

如果要和C函数交互，C++还有一种特殊的形参类型（即省略符）；

#### initializer_list形参

如果函数的实参数量未知但是全部实参的类型都相同，我们可以使用initializer_list类型的形参。

initializer_list是一种标准库类型，用于表示某种特定类型的值的数组。

initializer_list类型定义在同名的头文件中。

initializer_list对象中的元素永远是常量值，无法改变initializer_list对象中元素的值。

如果想向initializer_list形参中传递一个值序列，必须把序列放在一堆花括号内：

```cpp
void eror_msg(initializer_list<string> il)
{
    for(auto beg=il.begin();beg!=il.end();++beg)
        cout<<*beg<<" ";
    cout<<endl;
}

//expected和actual是string对象
if(expected != actual)
    error_msg({"functionX", expected, actual});
else
    error_msg({"functionX", "okay"});
```

#### 省略符形参数

省略符形参是为了便于C++程序访问某些特殊的C代码而设置的。这些C代码使用了名为varargs的C标准库功能。

需要特别注意的是：大多数类型的对象在传递给省略符形参时都无法正确拷贝。

省略符形参只能出现在形参列表的最后一个位置；

```cpp
void foo(parm_list, ...);
void foo(...);
```

省略符形参对应的实参数无须类型检查。

## 6.3 返回类型和return语句

### 6.3.1 无返回值函数

return语句中止当前正在执行的函数并将控制权返回到调用该函数的地方。

```cpp
return;
return expression;
```

### 6.3.2 有返回值函数

#### 值是如何被返回的

#### 不要返回局部变量的引用或者指针

#### 返回类类型的函数和调用和运算符

#### 引用返回左值

#### 列表初始化返回值

C++11新标准规定，函数可以返回花括号包围的值的列表。

#### 主函数main的返回值

main函数的返回值可以看作状态指示器。

cstdlib头文件定义了两个预处理变量：EXIT_FAILURE和EXIT_SUCCESS

#### 递归

### 6.3.3 返回数组指针

数组不能被拷贝，因此函数不能返回数组。但是函数可以返回数组的指针或引用。可以使用类型别名来简化。

```cpp
typedef int arrT[10];//arrT是一个类型别名，表示的类型是含有10个整数的数组
using arrT=int[10];//arrT的等价声明
arrT* func(int i);//func返回一个指向含有10个整数的数组的指针
```

#### 声明一个返回数组指针的函数

```cpp
int arr[10];//arr是一个含有10个整数的数组
int *p1[10];//p1是一个含有10个指针的数组
int (*p2)[10] = &arr;//p2是一个指针，它指向含有10个整数的数组
```

同样，如果想定义一个返回数组指针的函数：

```cpp
Type (*function(parameter_list)) [dimension]
```

具体举例：

```cpp
int (*func(int i))[10];
```

#### 使用尾置返回类型

尾置返回类型（trailing return type），任何函数的定义都能使用尾置返回，但是这种形式对于返回类型比较复杂的函数最有效果。

```cpp
auto func(int i) -> int(*)[10];
```

#### 使用decltype

还有一种情况，如果我们知道函数返回的指针指向哪个数组，就可以使用decltype关键字声明返回类型。例：

```cpp
int odd[]={1,3,4,7,9};
int even[]={0,2,4,6,8};
decltype(odd) *arrPtr(int i)
{
    return (i%2) ? &odd : &even;//返回一个指向数组的指针
}
```

## 6.4 函数重载

如果同一作用域内的几个函数名字相同但是形参不同，我们称之为重新载函数（overloaded）。

#### 定义重载函数

不允许两个函数除了返回类型外其他所有的要素都相同。

#### 判断两个形参的类型是否相同

#### 重载和const形参

==**顶层const形参不影响传入函数的对象。**==一个拥有顶层const的形参无法和另一个没有顶层const的形参区分。例如：

```cpp
Record lookup(Phone);
Record lookup(const Phone); //声明重复了
Record lookup(Phone*);
Record lookup(Phone* const); //声明重复了
```

另一方面，如果==**形参是某种类型的指针或引用，则通过区分其指向的是常量对象还是非常量对象可以实现函数重载，此时const是底层的。**==

```cpp
//对于接受引用或指针的函数来说，对象是常量还是非常量对应的形式参数不同
//定义了4个独立的重载函数
Record lookup(Account&);
Record lookup(const Account&);
Record lookup(Account*);
Record lookup(const Account*);
```

#### const_cast和重载

==**const_cast只能改变对象的底层const。**==

例：

```cpp
const char* pc;
char* p=const_cast<char*> pc;
```

const_cast通常用于重载函数的情景。例如：

```cpp
const string &shorterString(const string& s1, const string& s2)
{
    return s1.size()<=s2.size() ? s1 : s2;
}
```

```cpp
string& shorterString(string& s1, string& s2)
{
    auto& r=shorterString(const_cast<const string&>(s1), const_cast<const string&>(s2));
    return const_cast<string&>(r);
}
```

#### 调用重载的函数

函数匹配（重载确定）；

### 6.4.1 重载与作用域

如果在内层作用域中声明名字，它将隐藏外层作用域中声明的同名实体。在不同的作用域中无法重载函数名。

C++中，名字查找发生在类型检查之前；

## 6.5 特殊用途语言特性

### 6.5.1 默认实参

一旦某个形参被赋予了默认值，它后面的所有形参都必须有默认值；

#### 使用默认实参调用函数

函数调用时实参按照其位置解析，默认实参负责填补函数调用缺少的尾部实参；

要注意默认实参函数的调用只能省略尾部实参数；

设计默认实参函数的时候，其中一项任务就是合理设置形式参数的顺序，尽量让不怎么使用默认值的形参出现在前面，而让那些经常使用默认值的形参出现在后面。

#### 默认实参声明

对于函数的声明来说，通常的习惯是将其放在头文件中，并且一个函数只声明一次；

多次声明同一个函数也是合法的；

但是多次声明不能修改一个之前声明的默认值；

#### 默认实参初始值

局部变量不能作为默认实参；



### 6.5.2 内联函数和constexpr函数

#### 内联函数可以避免函数调用的开销

一般来说，内联机制用于优化规模较小、流程直接、频繁调用的函数。

#### constexpr函数

constexpr函数是指能用于常量表达式的函数。

几项约定：

- 函数的==返回类型以及所有形参的类型都必须是字面值类型==；
- 函数体中必须==有且只有一条return语句==；

constexpr函数被隐式地指定为内联函数；

允许constexpr函数的返回值并非一个常量；

#### 把内联函数和constexpr函数放在头文件中

内联函数和constexpr函数可以在程序中多次定义，并且多个定义必须完全一样。

内联函数和constexpr函数通常定义在头文件中。

### 6.5.3 调试帮助

cpp有一种类似于头文件保护的技术，以便有选择地执行调试代码。

基本思想：程序包含一些用于调试的代码，但是这些代码只在开发程序的时候使用。当应用程序编写完成时，要先屏蔽掉测试代码。这种方法用到两项处理功能：assert和NDEBUG。

#### assert预处理宏（preprocessor macro）

就是一些预处理变量；

```cpp
assert(expr);
```

先对expr求值，如果表达式为假，assert输出信息并且终止程序的执行。如果表达式为真，assert什么都不做。

assert宏定义在cassert头文件中。预处理名字由预处理器而非编译器管理。因此可以直接使用预处理名字。即不需要using，也不需要写成std::assert。

assert宏用于检查“不能发生”的条件。

#### NDEGUG

assert的行为依赖一个名为NDEGUG的预处理变量的状态。如果定义了NDEBUG，则assert什么都不做，即关闭了调试状态。

默认状态下没有定义NDEBUG。

可以：

```cpp
#define NDEBUG
```

从而关闭调试状态；

很多编译器都提供了一个命令行选项来定义预处理变量：

```bash
CC -D NDEBUG main.c
```

定义NDEBUG可以避免检查各种条件所需要的运行时开销。

除了用于assert之外，也可以使用NDEBUG编写自己的条件调试代码。如果NDEBUG未定义，将执行`#ifnedf`和`#endif`之间的代码；否则这些代码被忽略；

```cpp
void print(const int ia[], size_t size)
{
    #ifndef NDEBUG
    	//__func__是编译器定义的一个局部静态变量，用于存放函数的名字；
    	cerr<<__func__<<": array size is"<<size<<endl;
    #endif
    //...
}
```

使用变量`__func__`输出当前调试的函数的名字。编译器为每个函数都定义了`__func__`，它是const char的一个静态数组，用于存放函数的名字。

除了C++编译器定义的`__func__`之外，预处理起还定义了另外4个对于程序调试很有用的名字：

```cpp
__FILE__
__LINE__
__TIME__
__DATE__
```

## 6.6 函数匹配

#### 确定候选函数和可行函数

#### 寻找最佳匹配

#### 含有多个形式参数的函数匹配

### 6.6.1 实参类型转换

## 6.7 函数指针

函数指针指向的是函数类型而非对象。函数的类型由它的返回类型和形参类型共同决定；

```cpp
bool lengthCompare(const string&, const string&);
bool (*pf)(const string&, const string&);
```

#### 使用函数指针

#### 重载函数的指针

当我们使用重载函数的时候，上下文必须清晰地界定到底应该选用哪个函数。如果定义了指向重载函数的指针，指针类型必须与重载函数中的某一个精确匹配；

#### 函数指针形参

形参可以是指向函数的指针。

```cpp
void useBigger(const string& s1, const string& s2, bool pf(const string&, const string&));
void useBigger(const string& s1, const string& s2, bool (*pf)(const string&, const string&));
```

可以直接把函数作为实参使用，此时它会总动转换成指针：

```cpp
useBigger(s1,s2,lengthCompare);
```

直接使用函数指针类型显得冗长繁琐。可以用类型别名和decltype来简化代码；

```cpp
typedef bool Func(const string&, const string&);
typedef decltype(lengthCompare) Func2;
typedef bool (*FuncP)(const string&, const string&);
typedef decltype(lengthCompare) (*FuncP2);
```

```cpp
void useBigger(const string&, const string&, Func);
void useBigger(const string&, const string&, FuncP2);
```

#### 返回指向函数的指针

我们必须把返回类型写成指针形式，编译器不会自动地将函数返回类型当成对应的指针型处理。

使用类型别名；

```cpp
using F=int(int*, int);
using PF=int(*)(int*, int);
PF f1(int);
F* f1(int);
```

也可以直接声明：

```cpp
int (*f1(int))(int*, int);
```

还可以用尾置返回类型：

```cpp
auto f1(int)->int(*)(int*, int);
```

#### 将auto和decltype用于函数指针类型

当我们将decltype作用于某个函数的时候，它返回函数类型而非指针类型；



