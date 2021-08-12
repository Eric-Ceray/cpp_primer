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
    Sales_data()
}
```



## 7.2 访问控制与封装

## 7.3 类的其他特征

## 7.4 类的作用域

## 7.5 构造函数再探

## 7.6 类的静态成员

