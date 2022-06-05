---
title: 04 Makefile
date: 2022-05-27 21:02:46
tags: linux环境编程
---

# Makefile

[TOC]



### 文件命名：

Makefie的文件命名：makefile或者Makefile

### Makefile规则：

一个Makefile文件可以有一个或多个规则

```makefile
目标：依赖

	命令（shell）
	
# 例如
app:main.c add.c sub.c mult.c div.c
	gcc main.c add.c sub.c mult.c div.c -o app
```

Makefile中的其他规则都是为第一条规则服务的。当make解释器发现依赖中的文件不存在时，会向下执行执行其他的规则并找到所需的依赖。

```makefile
app:main.o add.o sub.o mult.o div.o
	gcc main.o add.o sub.o mult.o div.o -o app
main.o:main.c
	gcc main.c -o main.o
add.o:add.c
	gcc add.c -o add.o
sub.o:sub.c
	gcc sub.c -o sub.o
mult.o:mult.c
	gcc mult.c -o mult.o
div.o:div.c
	gcc div.c -o div.o
```

make解释器在执行规则时会比较依赖和目标的时间戳，若依赖的时间戳比目标的时间戳晚，那么make程序将会执行含有依赖的规则

### 变量

#### 自定义变量

变量名=变量值	var=hello

#### 预定义变量

`AR`:归档默认程序名称，默认`ar`

`CC`：C编译器名称，默认为`cc`

`CXX`：C++编译器的名称，默认为`g++`

`$@`：目标的完整名称

`$<`：第一个依赖文件的名称

`$^`：所有的依赖文件

获取变量的值 `$(变量名)`

#### 自动变量只能在规则的命令中使用

```makefile
app:main.c add.c sub.c mult.c div.c
	$(CC) $^ -o $@
```

```makefile
src=main.c add.c sub.c mult.c div.c
target=app
$(target):$(src)
	$(CC) $(src) -o $(target)
```

#### 模式匹配

```makefile
%.o:%.c
```

其中%是通配符，表示匹配一个字符串，两个`%`表示匹配同一个字符串，例如上述中冒号前匹配的字符串和冒号后匹配的字符串是同一个，可能匹配到`main.c:main.o`，但是不会匹配到`main.c:add.o`

```makefile
main.o:main.c
	gcc main.c -o main.o
add.o:add.c
	gcc add.c -o add.o
sub.o:sub.c
	gcc sub.c -o sub.o
mult.o:mult.c
	gcc mult.c -o mult.o
div.o:div.c
	gcc div.c -o div.o

#可采用通配符来重写
%.o:%.c
	$(CC) -c $< -o $@

```

#### 函数

##### wildcard

使用：

```makefile
# 获取当前目录下所有后缀以.c结尾的文件列表，并用空格隔开
src=$(wildcard ./*c)
```

##### patsubst

使用：

```makefile
# 字符串的替换，将src中以.c结尾的字符串替换为.o结尾的字符串
$(patsubst %.c, %.o,$(src))
```

```makefile
main.o:main.c
	gcc main.c -o main.o
add.o:add.c
	gcc add.c -o add.o
sub.o:sub.c
	gcc sub.c -o sub.o
mult.o:mult.c
	gcc mult.c -o mult.o
div.o:div.c
	gcc div.c -o div.o
	
```

若使用函数来写，可表示为：

```makefile
#获取当前目录下的.c文件
src=$(wildcard ./*.c)
#将.c的匹配模式替换成.o
objs=$(patsubst %.c, %.o, $(src))
target=app
$(target):$(objs)
	$(CC) $^ -o $@
%.o:%.c
	$(CC) -c $< -o $@

# clean是一个伪目标，就不会生成特定的clean文件，make工具不会进行更新检查
.PHONY:clean
clean:
	rm $(objs)
```