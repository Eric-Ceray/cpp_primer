# Ch 1 开始

## 1.5  类简介

### 1.5.1 Sales_item 类

关键概念：类定义了行为；

#### 读写Sales_item

```cpp
#include <iostream>
#include "Sales_item.h"
int main()
{
    Sales_item book;
    // 读入ISBN号、售出的册数以及销售价格
    std::cin >> book;
    // 写入ISBN、售出的册数、总销售额和平均价格
    std::cout<<book<<std::endl;
    return 0;
}
```

#### Sales_item 对象的加法

可以使用文件重定向来测试程序。

```shell
addItems <infile >outfile
```

### 1.5.2 初识成员函数

成员函数是定义为类的一部分的函数，也叫做方法。

点运算符；

调用运算符；
