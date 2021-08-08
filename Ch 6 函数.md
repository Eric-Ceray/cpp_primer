# Ch 6 函数

## 6.1 函数基础

## 6.2 参数传递

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
typedef int arrT[10];
using arrT=int[10];
arrT* func(int i);
```

#### 声明一个返回数组指针的函数

```cpp
Type (*function(parameter_list)) [dimension]
```



## 6.4 函数重载

## 6.5 特殊用途语言特性

## 6.6 函数匹配

## 6.7 函数指针

