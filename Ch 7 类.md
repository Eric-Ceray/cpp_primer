# Ch 7 类

C++中，使用类定义自己的数据类型。

数据抽象的重要性。

数据抽象能帮我们将对象的具体实现与对象所能执行的操作分离开来。

13章讨论如何控制对象拷贝。

14章学习如何自定义运算符。

类的基本思想是数据抽象（data abstraction）和封装（encapsulation）。

数据抽象是一种依赖于接口（interface）和实现（implementation）分离的编程（设计）技术。

类的接口包括用户所能执行的操作。

类的实现包括类的数据成员、负责接口实现的函数体以及类所需的各种私有函数。

抽象数据类型。

## 7.1 定义类抽象数据类型

第一章定义的Sales_item类和第二章定义的Sales_data类。

`Sales_item.h`

```cpp
#ifndef SALESITEM_H
// we're here only if SALESITEM_H has not yet been defined 
#define SALESITEM_H

#include <iostream>
#include <string>

class Sales_item
{
friend std::istream& operator>>(std::istream&, Sales_item&);
friend std::ostream& operator<<(std::ostream&, const Sales_item&);
friend bool operator<(const Sales_item&, const Sales_item&);
friend bool operator==(const Sales_item&, const Sales_item&);
public:
    Sales_item() = default;
    Sales_item(const std::string &book): bookNo(book) { }
    Sales_item(std::istream &is) { is >> *this; }
public:
    Sales_item& operator+=(const Sales_item&);
    std::string isbn() const { return bookNo; }
    double avg_price() const;
private:
    std::string bookNo;      // implicitly initialized to the empty string
    unsigned units_sold = 0; // explicitly initialized
    double revenue = 0.0;
};

inline
bool compareIsbn(const Sales_item &lhs, const Sales_item &rhs) 
{ return lhs.isbn() == rhs.isbn(); }

// nonmember binary operator: must declare a parameter for each operand
Sales_item operator+(const Sales_item&, const Sales_item&);

inline bool 
operator==(const Sales_item &lhs, const Sales_item &rhs)
{
    // must be made a friend of Sales_item
    return lhs.units_sold == rhs.units_sold &&
           lhs.revenue == rhs.revenue &&
           lhs.isbn() == rhs.isbn();
}

inline bool 
operator!=(const Sales_item &lhs, const Sales_item &rhs)
{
    return !(lhs == rhs); // != defined in terms of operator==
}

Sales_item& Sales_item::operator+=(const Sales_item& rhs) 
{
    units_sold += rhs.units_sold; 
    revenue += rhs.revenue; 
    return *this;
}

Sales_item 
operator+(const Sales_item& lhs, const Sales_item& rhs) 
{
    Sales_item ret(lhs);  // copy (|lhs|) into a local object that we'll return
    ret += rhs;           // add in the contents of (|rhs|) 
    return ret;           // return (|ret|) by value
}

std::istream& 
operator>>(std::istream& in, Sales_item& s)
{
    double price;
    in >> s.bookNo >> s.units_sold >> price;
    // check that the inputs succeeded
    if (in)
        s.revenue = s.units_sold * price;
    else 
        s = Sales_item();  // input failed: reset object to default state
    return in;
}

std::ostream& 
operator<<(std::ostream& out, const Sales_item& s)
{
    out << s.isbn() << " " << s.units_sold << " "
        << s.revenue << " " << s.avg_price();
    return out;
}

double Sales_item::avg_price() const
{
    if (units_sold) 
        return revenue/units_sold; 
    else 
        return 0;
}
#endif
```

另外是Sales_data的初步定义：

```cpp
struct Sales_data
{
    std::string bookNo;
    unsigned units_sold=0;
    double revenue=0.0;
}
```

只要一个类定义了它自己的操作，那么就可以封装（隐藏）它的数据成员了。

### 7.1.1 设计Sales_data类

最终目的是令Sales_data支持与Sales_item类完全一样的操作集合。

Sales_item类有一个名为isbn的成员函数（member function），并且支持+、=、+=、<<和>>运算符。

Sales_data的接口应该包含以下操作：

- 一个isbn成员函数，用于返回ISBN编号；
- 一个combine成员函数，用于将一个Sales_data对象加到另一个对象上；
- 一个名为add的函数，执行两个Sales_data对象的加法；
- 一个read函数，将数据从istream读入到Sales_data对象中；
- 一个print函数，将Sales_data对象的值输出到ostream；

#### 使用改进的Sales_data类

### 7.1.2 定义改进的Sales_data类

改进后类的数据成员：

- bookNo (string);
- units_sold (unsigned)
- revenue (double)

成员函数：

- combine
- isbn
- avg_price

```cpp
struct Sales_data
{
    //成员函数：关于Sales_data对象的操作
    std::string isbn() const {return bookNo;}
    Sales_data& combine(const Sales_data&);
    double avg_price() const;
    //数据成员
    std::string bookNo;
    unsigned units_sold=0;
    double revenue=0.0;
};
//Sales_data的非成员接口函数
Sales_data add(const Sales_data&, const Sales_data&);
std::ostream& print(std::ostream&, const Sales_data&);
std::istream& read(std::istream&, Sales_data&);
```

成员函数的声明必须在类的内部，它的定义则既可以在类的内部也可以在类的外部；

非成员函数的声明和定义都在类的外部；

==**定义在类内部的函数是隐式的inline函数；**==

#### 定义成员函数

#### 引入this

再一次观察对isbn成员函数的引用；

```cpp
total.isbn();
```

使用点运算符来访问total对象的isbn成员，然后调用；

成员函数通过一个名为this的额外的隐式参数来访问调用它的那个对象。当我们调用一个成员函数的时候，用请求该函数的对象地址初始化this。例如，如果调用：

```cpp
total.isbn()
```

则编译器负责把total的地址传递给isbn的隐式形参this，可以等价地认为编译器将该调用重新写成了如下形式：

```cpp
Sales_data::isbn(&total)
```

在成员函数的内部，我们可以直接使用调用该函数的对象的成员，而无须通过成员访问运算符来做到这一点，因为this所指的正是这个对象。任何对类成员的直接访问都被看作this的隐式引用，也就是说，当isbn使用bookNo时，它隐式地使用this指向的成员，就像我们书写了this->bookNo一样。

this是一个常量指针，不允许改变this中保存的地址；

#### 引入const成员函数

isbn函数的另外一个关键之处是紧随参数列表之后的const关键字，这里const的作用是修改隐式this指针的类型；

默认情况下，this的类型是Sales_data* const。即

```cpp
Sales_data* const this=&total;
```

因此不能将this绑定到常量对象上。这一情况也就使得我们不能在一个常量对象上调用普通的成员函数。

所以如果isbn是一个普通函数而且this是一个普通的指针参数，则this应该声明成：

```cpp
const Sales_data* const this;
```

因为在isbn的函数体中不会改变this所指的对象。

所以就有==**常量成员函**==数的的定义方式：

```cpp
std::string isbn() const {return bookNo;}
```

可以想象成：

```cpp
std::string Sales_data::isbn(const Sales_data* const this) {return this->bookNo;}
```

常量对象，以及常量常量对象的引用或指针都只能调用常量成员函数。

#### 类作用域和成员函数

编译器分两步处理：先编译成员的声明，然后才轮到函数体；

#### 在类的外部定义成员函数

类外部定义的成员函数的名字必须包含它所属的类名；

```cpp
double Sales_data::avg_price() const 
{
    if(units_sold) return revenue/units_sold;
    else return 0;
}
```

#### 定义一个返回this对象的函数

```cpp
Sales_data& Sales_data::combine(const Sales_data& rhs)
{
    units_sold += rhs.units_sold;
    revenue += rhs.revenue;
    return *this;
}
```

```cpp
total.combine(trans);
```

### 7.1.3 定义类相关的非成员函数

如果函数在概念上属于类但是不定义在类中，则一般应该和类声明在同一个头文件中。

#### 定义read和print函数

```cpp
istream& read(istream& is, Sales_data& item)
{
    double price=0;
    is>>item.bookNo>>item.units_sold>>price;
    item.revenue=price*item.units_sold;
    return is;
}
ostream& print(ostream& os, const Sales_data& item)
{
    os<<item.isbn()<<" "<<item.units_sold<<" "<<item.revenue<<" "<<item.avg_price();
    return os;
}
```

IO类属于不能被拷贝的类型，所以只能通过引用来传递它们；

#### 定义add函数

```cpp
Sales_data add(const Sales_data& lhs, const Sales_data rhs)
{
    Sales_data sum=lhs;
    sum.combine(rhs);
    return sum;
}
```

### 7.1.4 构造函数

每个类都分别定义了它的对象的被初始化方式，类通过一个或几个特殊的成员函数来控制其对象的初始化过程。这些函数叫做构造函数（constructor）。

构造函数的任务是初始化类对象的数据成员，无论何时只要类的对象被创建，就会执行构造函数。

- 构造函数的名字和类的名字相同；
- 构造函数没有返回类型；
- 类可以包含多个构造函数（重载）；
- 构造函数不能被声明成const；

#### 合成的默认构造函数

默认构造函数；

#### 某些类不能依赖于合成的默认构造函数

- 只有当类没有声明任何构造函数时，编译器才会自动地生成默认构造函数；
- 如果类包含有内置类型或者复合类型的成员，则只有当这些成员全部都被赋予了类内的初始值时，才适合于使用合成的默认构造函数；

#### 定义Sales_data的构造函数

```cpp
struct Sales_data
{
    Sales_data()=default;
    Sales_data(const std::string& s): bookNo(s) {}
    Sales_data(const std::string& s, unsigned n,, double p):bookNo(s), units_sold(n), revenue(p*n) {};
    Sales_data(std::istream&);
    //
    std::string isbn() const {return bookNo;}
    Sales_data& combine(const Sales_data&);
    double avg_price() const;
    std::string bookNo;
    unsigned units_sold=0;
    double revenue=0.0;
}
```

##### =default的含义

```cpp
Sales_data()=default;
```

#### 构造函数初始值列表

```cpp
Sales_data(const std::string& s): bookNo(s) {}
Sales_data(const std::string& s, unsigned n, double p): bookNo(s), units_sold(n), revenue(p*n) {}
```

构造函数初始值列表；

#### 在类的外部定义构造函数

```cpp
Sales_data::Sales_data(std::istream& is)
{
    read(is, *this);
}
```

### 7.1.5 拷贝、赋值和析构

除了定义类的对象如何初始化之外，类还需要控制拷贝、赋值和销毁对象的时候发生的行为。

#### 某些类不能依赖于合成的版本

比如动态内存管理；

值得注意的是，很多需要动态内存的类能够使用vector对象或者string对象来管理必要的存储空间，使用vector或者string的类能避免分配和释放内存带来的复杂性；

进一步将，如果类包含vector或者string对象，则其拷贝、赋值和销毁的合成版本能够正常工作。当我们对含有vector成员的对象执行拷贝或者赋值操作的时候，vector类会设法拷贝或者复制成员中的元素。当这样的对象被销毁时，将销毁vector对象，也就是依次销毁vector中的每个元素。这一点与string类似；

## 7.2 访问控制与封装

C++中使用访问说明符（access specifier）来加强类的封装性；

- 定义在public说明符之后的成员在整个程序内可被访问，public成员定义类的接口；
- 定义在private说明符之后的成员可以被类的成员函数访问，但是不能被使用该类的代码访问，private部分封装了类的实现细节；

```cpp
class Sales_data
{
public:
    Sales_data()=default;
    Sales_data(const std::string& s, unsigned n, double p):bookNo(s), units_sold(n), revenue(p*n) {}
    Sales_data(const std::string& s): bookNo(s) {}
    Sales_data(std::istream&);
    std::string isbn() const {return bookNo};
    Sales_data& combine(const Sales_data&);
private:
    double avg_price() const {return units_sold ? revenue/units_sold : 0;}
    std::string bookNo;
    unsigned units_sold=0;
    double revenue=0.0;
};
```

#### 使用class或者struct关键字

struct和class的默认访问权限不同；

类可以在它的第一个访问说明符之前定义成员，对这种成员的访问权限依赖于类定义的方式。如果我们使用struct关键字，则定义在第一个访问说明符之前的成员是public的；相反，如果我们使用class关键字，则这些成员是private的；

### 7.2.1 友元

类可以允许其他类或者函数访问它的非公有成员，方法是令其他类或者函数称为他的友元。如果类想把一个函数作为它的友元，则只需要增加一条以friend关键字开始的函数声明语句即可：

```cpp
class Sales_data
{
friend Sales_data add(const Sales_data&, const Sales_data&);
friend std::istream& read(std::istream&, Sales_data&);
friend std::ostream& print(std::ostream&, const Sales_data&);
public:
    Sales_data()=default;
    Sales_data(const std::string& s, unsigned n, double p):bookNo(s), units_sold(n), revenue(p*n) {}
    Sales_data(const std::string& s):bookNo(s) {}
    Sales_data(std::istream&);
    std::string isbn() const {return bookNo;}
    Sales_data& combine(const Sales_data&);
private:
    std::string bookNo;
    unsigned units_sold=0;
    double revenue=0.0;
};
Sales_data add(const Sales_data&, const Sales_data&);
std::istream& read(std::istream&, Sales_data&);
std::ostream& print(std::ostream&, const Sales_data&);
```

封装的益处：

- 确保用户代码不会无意间破坏封装对象的状态；
- 被封装的具体实现细节可以随时改变，而无须调整用户级别的代码；

#### 友元的声明

友元的声明仅仅指定了访问权限，并不是一个普通意义上的函数声明。如果希望类的用户能够调用某个友元函数，那么必须在友元声明外再专门对函数进行一次声明；

通常把友元的声明与类本身放置在同一个头文件中；

## 7.3 类的其他特征

类型成员、类的成员和类内初始值、可变数据成员、内联成员函数、从成员函数返回\*this、关于如何定义并使用类类型以及友元类；

#### 7.3.1 类成员再探

定义一堆相互关联的类，分别是Screen和Window_mgr；

#### 定义一个类型成员

除了定义数据和函数成员之外，类还可以自定义某种类型在类中的别名。由类定义的类型名字和其他成员一样存在访问限制；

```cpp
class Screen
{
public:
    typdef std::string::size_type pos;
private:
    pos cursor=0;
    pos height=0, width=0;
    std::string contents;
}
```

#### Screen类的成员函数

```cpp
class Screen
{
public:
    typedef std::string::size_type pos;
    Screen()=default;//因为Screen有另一个构造函数，所以必须有这个函数；
    //cursor被其类内初始值初始化为0；
    Screen(pos ht, pos wd, char c): height(ht), width(wd), contents(ht*wd, c) {}
    char get() const {return contents[cursor];} //读取光标处的字符，隐式内联
    inline char get(pos ht, pos wd) const;//显式内联
    Screen& move(pos r, pos c);//能在之后被设为内联
private:
    pos cursor=0;
    pos height=0,width=0;
    std::string contents;
};
```

#### 令成员作为内联函数

我们可以在类的内部把inline作为声明的一部分显式地声明成员函数，同样的，也可能在类的外部用inline关键字修饰函数的定义；

最好只在类外部定义的地方说明inline，这样可以使类更容易理解；

#### 重载成员函数

#### 可变数据成员

有时我们希望能够修改类的某个数据成员。即使是在一个const成员函数内，可以通过在变量的声明中加入mutable关键字做到。

一个可变函数成员（mutable data member）永远不会是const，即使它是const对象的成员。因此，一个const成员函数可以改变一个可变成员的值。

```cpp
class Screen()
{
public:
    void some_member() const;
private:
    mutable size_t access_ctr; //即使在一个const对象内也能被修改
};
void Screen::some_member() const
{
    ++access_ctr; //保存一个计数值，用于记录成员函数被调用的次数
}
```

#### 类数据成员的初始值

```cpp
class Window_mgr
{
private:
	std::vector<Screen> screens{Screen(24,80,' ')};  
};
```

当我们提供一个类内初始值的时候，必须以符号=或者花括号表示；

### 7.3.2 返回\*this的成员函数

```cpp
class Screen()
{
public:
    Screen& set(char);
    Screen& set(pos, pos, char);
    //其他成员和之前的版本一致
};
inline Screen& Screen::set(char c)
{
    contents[cursor]=c;
    return *this;
}
inline Screen& Screen::set(pos r, por col, char ch)
{
    contents[r*width+col]=ch;//设置给定位置的新值
    return *this;
}
```

```cpp
myScreen.move(4,0).set('#');
```

#### 从const成员函数返回\*this

一个const成员函数如果以引用的形式返回\*this，那么它的返回类型将是常量引用；

#### 基于const的重载

当一个成员调用另一个成员的时候，this指针在其中隐式地从指向非常量的指针转换成指向常量的指针；

#### 7.3.3 类类型

#### 类的声明

```cpp
class Screen;
```

前向声明，不完全类型；

不完全类型只能在非常有限的情况下使用：可以定义指向这种类型的指针或引用，也可以声明（但是不能定义）以不完全类型作为参数或者返回类型的函数；

一个类的成员类型不能是类自己；

一旦一个类的名字出现后，它就被认为是声明过了，因此类允许包含指向它自身类型的引用或指针：

```cpp
class Link_screen
{
    Screen window;
    Link_screen* next;
    Link_screen* prev;
}
```

### 7.3.4 友元再探

类还可以把其他的类定义成友元。也可以把其他类的成员函数定义成友元；

#### 类之间的友元关系

例如：Window_mgr类的某些成员函数可能需要访问它管理的Screen类的内部数据；那么Screen需要把Window_mgr指定成它的友元；

```cpp
class Screen
{
    friend class Window_mgr;
};
```

如果一个类制定了友元类，则友元类的成员函数可以访问此类包括非公有成员函数在内的所有成员；

```cpp
class Window_mgr
{
public:
    using ScreenIndex=std::vector<Screen>::size_type;
    void clear(ScreenIndex);
private:
    std::vector<Screen> screens{Screen(24, 80, ' ')};
};

void Window_mgr::clear(ScreenIndex i)
{
    Screen& s=screens[i];
    s.contents=string(s.height*s.width, ' ');
}
```

友元关系不存在传递性，每个类负责控制自己的友元类或者友元函数；

#### 令成员函数作为友元

```cpp
class Screen
{
    friend void Window_mgr::clear(ScreenIndex);
};
```

想要令某个成员函数作为友元，必须仔细组织程序的结构；

- 首先定义Window_mgr类，在其中声明clear函数，但是不能定义它。在clear使用Screen的成员之前必须先声明Screen；
- 接下来定义Screen，包括对于clear的友元声明；
- 最后定义clear，此时它才可以使用Screen的成员；

#### 函数重载和友元

#### 友元声明和作用域

友元声明的作用是影响访问权限，它本身并非普通意义上的声明；

## 7.4 类的作用域

#### 作用域和定义在类外部的成员

返回类型中使用的名字都位于类的作用域外。这时，返回类型必须指明它是哪个类的成员；

```cpp
class Window_mgr
{
public:
    ScreenIndex addScreen(const Screen&);
};
Window_mgr::ScreenIndex
Window_mgr::addScreen(const Screen& s)
{
    screen.push_back(s);
    return screens.size()-1;
}
```

### 7.4.1 名字查找与类的作用域

名字查找；

编译器处理完类中的全部声明之后才会处理成员函数的定义；

#### 用于类成员声明的名字查找

#### 类型名要特殊处理

类型名的定义通常出现在类的开始处，这样就能确保所有使用该类型的成员都出现在类名的定义之后；

#### 成员定义中的普通块作用域的名字查找

尽管类的成员被隐藏了，但是我们仍然可以通过加上类的名字或显式地使用this指针来强制访问函数；

#### 类作用域之后，在外围的作用域中查找

#### 在文件中名字的出现处对其进行解析

## 7.5 构造函数再探

### 7.5.1 构造函数初始值列表

就对象的数据成员而言，如果没有在构造函数的初始值列表中显式地初始化成员，则该成员将在构造函数体之前执行默认初始化；

#### 构造函数的初始值有时必不可少

- 如果成员是const或者是引用的话，必须将其初始化；
- 当成员属于某种类类型并且该类没有定义默认构造函数，也必须将这个成员初始化；

#### 成员初始化的顺序

最好令构造函数初始值的顺序与成员声明的顺序保持一致。而且如果可能的话，尽量避免使用某些成员初始化其他成员；

#### 默认实参和构造函数

如果一个构造函数为所有参数都提供了默认实参，则它实际上也定义了默认构造函数；

### 7.5.2 委托构造函数

委托构造函数（delegating constructor）：一个委托构造函数使用它所属类的其他构造函数执行它自己的初始化过程，或者说它把它自己的一些职责委托给了其他构造函数；

和其他构造函数一样，一个委托构造函数也有一个成员初始值的列表和一个函数体。在委托构造函数内，成员初始值列表只有一个唯一的入口，就是类名本身；

```cpp
class Sales_data
{
public:
    Sales_data(std::string s, unsigned cnt, double price):bookNo(s), units_sold(cnt), revenue(cnt*price) {}
    Sales_data(): Sales_data("",0,0) {}
    Sales_data(std::string s): Sales_data(s,0,0) {}
    Sales_data(std::istream& is): Sales_data() {read(is, *this);}
};
```

### 7.5.3 默认构造函数的作用

#### 使用默认构造函数

### 7.5.4 隐式的类类型转换

C++语言在内置类型之间定义了集中自动转换规则；

同样的，也能为类定义隐式转换规则。如果构造函数只能接受一个实参，则它实际上定义了转换为此类类型的隐式转换机制，有时我们把这种构造函数称为转换构造函数（converting constructor）。

能通过一个实参调用的构造函数定义了一条从构造函数的参数类型向类类型隐式转换的规则；

#### 只允许一步类类型转换

#### 类类型转换并不总是有效

#### 抑制构造函数定义的隐式转换

在要求隐式转换的程序上下文中，我们可以通过将构造函数声明为explicit加以阻止：

#### explicit构造函数只能用于直接初始化

当我们使用explicit关键字声明构造函数时，它将只能以直接初始化的形式使用。而且，编译器将不会在自动转换过程中使用该构造函数；

即不能将explicit构造函数用于拷贝形式的初始化过程；

#### 为转换显式使用构造函数

```cpp
item.combine(Sales_data(null_book));
item.combine(static_cast<Sales_data>(cin));
```

#### 标准库中含有显式构造函数的类

我们用过的一些标准库中的类含有单参数的构造函数：

- 接受一个单参数的`const char*`的`string`构造函数，不是explicit的；
- 接受一个容量参数的`vector`构造函数是explicit的；

### 7.5.5 聚合类

聚合类（aggregate class）使得用户可以直接访问其成员，并且具有特殊的初始化语法形式；

当一个类满足如下条件时，我们说它是聚合的：

- 所有成员都是public的；
- 没有定义任何构造函数；
- 没有类内初始值；
- 没有基类，也没有virtual函数；

可以用一个花括号括起来的成员初始值列表，并用它初始化聚合类的数据成员；

### 7.5.6 字面值常量类

除了算数类型、引用和指针外，某些类也是字面值类型。和其他类不同，字面值类型的类可能含有constexpr函数成员，这样的成员必须符合constexpr函数的所有要求，它们是隐式const的；

数据成员都是字面值类型的聚合类是字面值常量类；

如果一个类不是聚合类，但它符合下述要求，则它也是一个字面值常量类；

- 数据成员都必须是字面值类型；
- 类必须至少含有一个constexpr构造函数；
- 如果一个数据成员含有类内初始值，则内置类型成员的初始值必须是一条常量表达式；如果成员属于某种类类型，则初始值必须使用成员自己的constexpr构造函数；
- 类必须使用析构函数的默认定义，该成员负责销毁类的对象；

#### constexpr构造函数

构造函数不能是const的，但是字面值常量类的构造函数可以是constexpr函数，事实上，一个字面值常量类必须至少提供一个constexpr构造函数；

constexpr构造函数可以声明成=default的形式；

## 7.6 类的静态成员

#### 声明静态成员

静态成员函数不与任何对象绑定在一起，它们不包含this指针。作为结果，静态成员函数不能声明称const的，而且我们也不能在static函数体内使用this指针；

#### 使用类的静态成员

#### 定义静态成员

#### 静态成员的类内初始化

通常情况下，类的静态成员不应该在类的内部初始化。然而，我们可以为静态成员提供const整数类型的类内初始值，不过要求静态成员必须是字面值常量类型的constexpr。

即使一个常量静态数据成员在类内部被初始化了，通常情况下也应该在类的外部定义一下该成员；

#### 静态成员能用于某些场景，而普通成员不能





