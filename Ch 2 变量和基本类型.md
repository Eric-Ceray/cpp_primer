# Ch2 变量和基本类型

## 2.4 const限定符

有时候我们希望定义这样一种变量，它的值不能被改变。为了满足这一要求，可以用关键字`const`来对变量的类型加以限定。

```cpp
const int bufSize = 512;//输入缓冲区大小
```

因为const对象一旦创建之后其值就不能再改变，所以==**const对象必须初始化**==。一如即往，初始值可以是任意复杂的表达式。

#### 初始化和const

#### 默认状态下，const对象仅在文件内有效

一般变量的多文件分享：

定义在一个文件：

```cpp
int a = 1;
```

在要使用的文件里面声明：

```cpp
extern int a;
```

const变量的多文件分享：

因为默认情况下，const对象被设定在仅在文件内有效。当多个文件中出现了同名的const变量时，其实等同于在不同文件中分别定义了独立的变量。

因此const变量的多文件分享：

定义在一个文件：

```cpp
extern const int a=1;
```

在要使用的文件里面声明：

```cpp
extern const int a;
```

### 2.4.1 const的引用

reference to const

可以吧引用绑定到const对象上，就像绑定到其他对象上一样，我们称之为对常量的引用。

```cpp
const int ci = 1024;
const int& r1 = ci;
```

#### 初始化和对const的引用

一般来说，引用的类型必须与其所引用的对象的类型一样。但是有两种例外；

第一种例外就是**初始化常量引用**的时候允许用任意表达式作为初始值，只要该表达式的结果能够转换成引用的类型即可；

```cpp
int i = 42;
const int& r1 = i; //允许将const int&绑定到一个普通int对象上
const int& r2 = 42; //正确：r2是一个常量引用
const int& r3 = r1*2; //正确：r3是一个常量引用
int& r4 = r1*2; //错误
```

#### 对const的引用可能引用一个并非const的对象

常量引用仅仅对引用可参与的操作做出了限定，对于应用的对象本身是不是一个常量未做限定。因为对象也可能是个非常量，所以允许通过其他途径改变它的值；

```cpp
int i=42;
int& r1 = i;
const int& r2 = i;
r1 = 0;
r2 = 0; //错误
```

### 2.4.2 指针和const

```cpp
const double pi=3.14;
const double* cptr = &pi;
```

同样的，指针的类型必须与其所指的对象的类型一致，但是依旧有两个例外；

第一种例外就是允许一个指向常量的指针指向一个非常量对象：

```cpp
double dval = 3.14;
const double* cptr = &dval;
```

和常量引用一样，指向常量的指针也没有规定其所指向的对象不许是一个常量；

所谓的指向常量的指针或者引用，不过时指针或者引用“自以为是”，它们觉得自己指向了常量，因此自觉地不去改变所指向的对象的值；

#### const指针

允许把指针本身设定为常量。常量指针必须初始化。

```cpp
int errNumb = 0;
int *const curErr = &errNumb;
const double pi = 3.14159;
const double *const pip = &pi;
```

### 2.4.3 顶层const

### 2.4.4 constexpr和常量表达式

常量表达式是指值不会改变并且在编译过程就能得到计算结果的表达式。

```cpp
const int max_files = 20; //yes
const int limit = max_files + 1; //yes
int staff_size = 27; //no
const int sz = get_size(); //no
```

尽管sz本身是一个常量，但它的具体值直到运行的时候才能获取到，所以也不是常量表达式。

#### constexpr变量

在一个复杂系统中，很难分辨一个初始值到底是不是常量表达式。 

C++11新标准规定，允许将变量声明为constexpr类型以便由编译器来验证==变量的值==是否是一个常量表达式。

声明为constexpr的变量一定是一个变量，并且必须用常量表达式初始化。

一般来说，如果你认定变量是一个常量表达式，那就把它声明称constexpr。

#### 字面值类型

算数类型、引用、指针都属于字面量类型。

常量表达式的值需要在编译的时候就得到计算，因此对声明constexpr时用到的类型必须有所限制。必须是字面值类型。

尽管指针和引用都能定义成constexpr，但是它们的处世之却受到严格限制。一个constexpr指针的初始值必须是nullptr或者0，或者是存储于某个固定地址中的对象。

#### 指针和constexpr

### 2.5 处理类型

### 2.5.1 类型别名

### 2.5.2 auto类型说明符

#### 符合类型、常量和auto

编译器推断出来的auto类型有时候和初始值的类型并不完全一样。编译器会适当地改变结果类型使其更加符合初始化规则。

auto一般会忽略掉顶层const，同时底层const则会保留下来。例如：

```cc
int i = 0;
int& r = i;
auto a = r;
const int ci = i;
const int& cr = ci;
auto b = ci; //b是整数
auto c = cr; //c是整数
auto d = &i; //d是一个整形指针
auto e = &ci; //e是一个指向整形常量的指针
```

如果希望推断出的auto类型是一个顶层const，需要明确指出：

```cc
const auto f = ci; //f是const int
```

还可以将引用的类型设为auto，此时原来的初始化规则仍然适用：

```cc
auto& g = ci; // g是一个整形常量引用
auto& h = 42; //错误
const auto& j = 42; //正确，可以为常量引用绑定字面值
```

### 2.5.3 decltype类型指示符

#### decltype和引用

## 自定义数据结构

### 2.6.1 定义Sales_data类型

### 2.6.2 使用Sales_data类

### 2.6.3 编写自己的头文件

