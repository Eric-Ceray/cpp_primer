# Make命令学习笔记

（教程：[阮一峰的教程](https://www.ruanyifeng.com/blog/2015/02/make.html)）

编译（compile）：将代码变成可执行文件

构建（build）：编译顺序的安排

Make是最常用的构建工具，诞生于1977年，主要用于C/C++的项目。

## 一、Make的概念

make只是一个根据指定的Shell命令进行构建的工具。它的规则很简单，你规定要构建哪个文件、它依赖哪些源文件、当那些文件有变动时，如何重新构建它。

## 二、Makefile文件的格式

构建规则都写在Makefile文件里，要学会如何Make命令，就必须学会如何编写Makefile文件。

### 2.1 概述

Makefile文件由一系列规则（rules）构成。每条规则的形式如下。

```makefile
<target>: <prerequisites>
[tab] <commands>
```

### 2.2 目标（target）

一个目标就构成一条规则。目标通常是文件名，指明Make命令所要构建的对象。

除了文件名，目标还可以是某个操作的名字，这称为“伪目标”（phony target）。

```makefile
clean:
	rm *.o
```

但是，如果当前目录中，正好有一个文件叫做clean，那么这个命令不会执行。因为Make发现clean文件已经存在，认为就没有必要重新构建了，就不会执行指定的rm命令。

为了避免这种情况，可以明确声明clean是“伪目标”：

```makefile
.PHONY: clean
clean:
	rm *.o
```

声明clean是“伪目标”之后，make就不会去检查是否存在一个叫clean的文件，而是每次运行都执行对应的命令。

如果Make命令执行时没有指定目标，默认会执行Makefile文件的第一个目标。

### 2.3 前置条件（prerequisites）

前置条件通常是一组文件名，之间用空格分割。前置条件指定了“目标”是否重新构建的判断标准：只有有一个前置文件不存在，或者有过更新（前置文件的last-modification时间戳比目标的时间戳新），“目标”就需要重新构建。

如果需要生成多个文件，往往采用下面的写法。

```makefile
source: file1 file2 file3
```

上面代码中，source是一个伪目标，只有三个前置文件，没有任何对应的命令。

执行`make source`之后，就会一次性生成file1，file2，file3三个文件。这比下面的写法要方便很多：

```makefile
$ make file1
$ make file2
$ make file3
```

### 2.4 命令（commands）

命令（commands）表示如何更新目标问问你家，由一行或多行的Shell命令组成。它是构建“目标”的具体指令，它的运行结果通常就是生成目标文件。

每行命令之前必须有一个tab键。

需要注意的是，每行命令在一个单独的shell中执行。这些Shell之间没有继承关系。

一个解决办法是将两行命令写在一起，中间用分号分隔。例如：

```makefile
var-kept:
	export foo=bar; echo "foo=[$$foo]"
```

另一个解决办法是在换行符之前加反斜杠转义。例如

```makefile
var-kept:
	export foo=bar; \
	echo "foo=[$$foo]"
```

最后一个方法是加上`.ONESHELL`命令。例如：

```makefile
.ONESHELL：
var-kept:
	export foo=bar;
	echo "foo=[$$foo]"
```

## 三、Makefile文件的语法

### 3.1 注释

#在Makefile中表示注释。

### 3.2 回声（echoing）

正常情况下，make会打印每条命令，然后再执行，这就叫做回声（echoing）。

在命令的前面加上@，就可以关闭回声。

由于在构建过程中，需要了解当前在执行哪条命令，所以通常只在注释和纯显示的echo命令前面加上@。

### 3.3 通配符

通配符（wildcard）用来指定一组符合条件的文件名。Makefile的通配符与bash一致。

### 3.4 模式匹配

Make命令允许对文件名进行类似正则运算的匹配，主要用到的匹配符是%。比如，假定当前目录下有f~1~.c和f~2~.c两个源码文件，需要将它们编译为对应的对象文件。则可以这样写：

```makefile
%.o: %.c
```

等同于下面的写法。

```makefile
f1.o: f1.c
f2.o: f2.c
```

使用匹配符%，可以将大量同类型的文件，只用一条规则就完成构建。

### 3.5 变量和赋值符

Makefile允许使用等号自定义变量。例如：

```makefile
txt = Hello World
test:
	@echo $(txt)
```

上面代码中，变量txt等于Hello World。调用时，变量需要放在$()之中。

调用Shell变量，需要在美元符号前，再加一个美元符号，这是因为Make命令会对美元符号转义。

```makefile
test:
	@echo $$HOME
```

有时，变量的值可能指向另一个变量。例如：

```makefile
v1 = $(v2)
```

Makefile一共提供了四个赋值运算符（=、:=、?=、+=）

### 3.6 内置变量（Implicit  Variables）

Make命令提供了一系列的内置变量，比如，\$(CC)指向当前使用的编译器，\$(MAKE)指向当前使用的MAKE工具。这主要是为了跨平台的兼容性。

### 3.7 自动变量（Automatic Variables）

Make命令还提供了一些自动变量，它们的值与当前规则有关。主要有以下几个：

1. \$@

\$@指代当前目标，即Make命令当前构建的那个目标。比如，`make foo`的、\$@就指代foo。

2. \$<

\$@指代第一个前置条件

3. \$?

\$?指代比目标更新的所有前置条件

4. \$^

\$^指代所有前置条件，之间以空格分隔。

5. \$*

\$*指代匹配符%匹配的部分。

6. \$(@D)和\$(@F)

\$(@D)和\$(@F)分别指向\$@的目录名和文件名。比如，\$@是/src/input.c，那么\$(@D)的值为src，\$(@F)的值为input.c。

7. \$(<D)和\$(<F)

\$(<D)和\$(<F)分别指向\$<的目录名和文件名。

下面是自动变量的一个例子：

```makefile
dest/%.txt: src/%.txt
	@[ -d dest ] || mkdir test
	cp $< $@ 
```

### 3.8 判断和循环

Makefile使用Bash语法，完成判断和循环。下面是两个例子

```makefile
ifeq ($(CC),gcc)
	libs=$(libs_for_gcc)
else
	libs=$(normal_libs)
endif
```

```makefile
LIST = one two three
all:
	for i in $(LIST); do \
		echo $i; \
	done
```

### 3.9 函数

Makefile还可以使用函数，格式如下。

```makefile
$(function arguments)
# 或者
${function arguments}
```

(1) shell函数

shell函数用来执行shell命令

```makefile
srcfiles := $(shell echo src/{00..99}.txt)
```

(2) wildcard函数

wildcard函数用来在Makefile中，替换bash的通配符。

```makefile
srcfiles := $(wildcard src/*.txt)
```

(3) subst函数

subst将函数用来文本替换，格式 如下。

```makefile
\$(subst from,to,text)
```

(4) patsubst函数

patsubst函数用于模式匹配的替换，格式如下：

```makefile
$(patsubst pattern,replacement,text)
```

(5) 替换后缀名

替换后缀名函数的写法是：变量名+冒号+后缀名替换规则。它实际上是patsubst函数的一种简写形式。

```makefile
min: $(OUTPUT:.js=.min.js)
```

## 四、Makefile实例





