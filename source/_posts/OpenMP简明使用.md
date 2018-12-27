---
title: OpenMP简明使用
date: 2018-12-27 11:16:32
categories: parallel
type: "tags"
tags:
    - parallel
---

### OpenMP安装

```
$ brew install libomp     #最新版MacOSX只需要安装即可使用
$ brew install gcc      #gcc是必须安装的
```

### 配置Clion

Preferences -> Build,Execution,Deployment -> Toolchains中新建一个用户配置，设置对应的cmake，make，C/C++ compiler

Preferences -> Build,Execution,Deployment -> CMake中Toolchain选择自己所配置的新用户

<!-- more -->

### CMakeLists.txt如何配置

```
cmake_minimum_required(VERSION 3.13.2)
project(projName CXX)
find_package(OpenMP REQUIRED)
if(OPENMP_FOUND)
    message("OPENMP FOUND")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
endif()
set(ignoreMe "${CMAKE_C_COMPILER}")

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++17")
set(SOURCE_FILES main.cpp)
add_executable(projName ${SOURCE_FILES})
```

如果只使用C++，则只配置了项目的CXX属性，并忽略C编译器的配置

### OpenMP使用

#### 常用函数

```
#返回当前可用的处理器个数
int omp_get_num_procs(void) 

#返回当前并行区域中的活动线程个数，如果在并行区域外部调用，返回1
int omp_get_num_threads(void)

#返回当前线程号
int omp_get_thread_num(void) 

#设置进入并行区域时，将要创建的线程个数
int omp_set_num_threads(int)
```

#### parallel和for循环

##### 声明

第二种声明方式简洁，但第一种声明方式可以在for循环以外写其他并行代码

```
//声明方式一
#pragma omp parallel
{
    #pragma omp for
    for (int i = 0; i < 100; ++i){
        ...
    }
}

//声明方式二
#pragma omp parallel for
for (int j = 0; j < 100; ++j){
    ...
}
```

##### for循环并行化约束条件

- for循环中的循环变量必须是有符号整形
- for循环中比较操作符必须是<, <=, >, >=
- for循环中的第三个表达式，必须是整数加减，并且加减值必须是一个循环不变量，且只能是自操作，如++,--,+=,-=
- 如果for循环中的比较操作为<或<=，那么循环变量只能增加；反之亦然
- 循环必须是单入口、单出口，也就是说循环内部不允许能够达到循环以外的跳转语句，exit除外. 异常的处理也必须在循环体内处理

#### 数据共享与私有化

除了以下三种情况外，并行区域中的所有变量都是共享的：

- 并行区域中定义的变量
- 多个线程用来完成循环的循环变量
- private、firstprivate、lastprivate或reduction字句修饰的变量

```
int share_to_private_b = 1;
#pragma omp parallel
{
    ...
    #pragma omp for private(share_to_private_b)
    for (int i = 0; i < 10; ++i)
    {
        ...
    }
}
```

```
#如果使用private，无论该变量在并行区域外是否初始化，在进入并行区域后，该变量均不会初始化
private(val1, val2, ...)
#与private不同的是，每个线程在开始的时候都会对该变量进行一次初始化
firstprivate(val1, val2, ...)
#与private不同的是，并发执行的最后一次循环的私有变量将会拷贝到val
lastprivate(val1, val2, ...)
#共享变量，等同于加锁
shared(val1, val2, ...) 
```

#### reduction

```
#pragma omp parallel for reduction(+:sum)
```

reduction(operator: var1, val2, ...)

其中operator以及约定变量的初始值如下：

运算符      |      数据类型       |     默认初始值
---        |        ---        |       ---
  +        |      整数、浮点     |        0
  -        |      整数、浮点     |        0
  *        |      整数、浮点     |        1
  &        |        整数        |     所有位均为1
  &#124;    |        整数        |        0
  ^        |        整数        |        0
  &&       |        整数        |        1
  &#124;&#124;   |        整数        |        0

#### 线程同步atomic

```
#pragma omp atomic
```

atomic只能用于两种情况(单变量的自操作，跟for中介绍的一样)：

- 自加自减：x++, x--, --x, ++x
- x< + or &#42; or - or / or & or | or << or >> >=expr，(例如x <<= 1; or x &#42;=2;)

#### 线程同步critical

critical用途类似java中的synchronized

```
#pragma omp critical[(name)] //[]表示名字可选
{
    //并行程序块，同时只能有一个线程能访问该并行程序块
}

#pragma omp parallel for
for (int i = 0; i < 100; ++i){
    #pragma omp critical(a)
    {
        sum = sum + i;
        sum = sum + i * 2;
    }
}
```

#### 线程事件同步机制

##### 隐式屏障

barrier为隐式栅障，即并行区域中所有线程执行完毕之后，主线程才继续执行。可以显示调用

```
//在两个for循环中添加
#pragma omp barrier
```

##### nowait取消栅障

```
#pragma omp for nowait //不能用#pragma omp parallel for nowait
or
#pragma omp single nowait
```

##### master

通过#pragma omp mater来声明对应的并行程序块只由主线程完成

```
#pragma omp master
{
    //只有一个主线程执行这个for循环
    for (int j = 0; j < 10; ++j){
        printf("%d-\n", j);
    }
}
```

##### section

```
#pragma omp parallel sections
{
    #pragma omp section //由一个线程单独完成
    for (int i = 0; i < 5; ++i){
        ...
    }

    #pragma omp section //由另一个线程单独完成
    for (int j = 0; j < 5; ++j){
        ...
    }
}
```

### 参考文献
- [小土刀-OpenMP入门指南](https://wdxtub.com/2016/03/20/openmp-guide/)