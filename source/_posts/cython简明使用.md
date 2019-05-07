---
title: cython简明使用
date: 2019-01-04 14:11:32
categories: python
tags:
    - python
---

### Cython目标

Cython主要是针对python性能太差，解决尤其是cpu矩阵运算，循环运算的性能问题而诞生。

Cython将你的类python代码编译成c链接库，你可以自己定义哪些运算可以交给c代码去运行。如果你想使用Cython，代表你已经开始关注计算性能了，榨取性能不只是交给C来处理就应该满足的，你可能还需要关注python语句里可优化的代码。

<!-- more -->

### 安装

```
$ pip3 install Cython
```

### 配置Pycharm

Preferences -> Tools -> External Tools新添加一个用户配置

```
# $xxx$均是宏插入，本设置用于将.pyx代码编译为c时所用
Tool Setting:
- Program: /to/path/bin/python3
- Arguments: $FilePath$ build_ext --inplace
- Working directory: $FileDir$
```

### .pxd/.pyd/.pyx区别

- .pxd

.pxd是扩展模块头文件，类似于C语言的.h头文件，.pxd文件中有 Cython模块要包含的Cython声明。.pxd文件还可为.pyx文件模块提供 Cython接口，以便其它Cython模块可使用比Python更高效的协议与之进行通信。

可用cimport关键字将.pxd文件导入.pyx模块文件中。

- .pyx

.pyx是扩展模块源代码文件，类似于C语言的.c源代码文件，.pyx文件中有 Cython模块的源代码。

.pyx必须先被编译成.c文件，再编译成.pyd(Windows)或.so(Linux)文件，才可作为模块import导入使用。

- .pyd

.pyd是由其它编程语言"编写-编译"生成的Python扩展模块。Python要导入.pyd文件，把它当成module来用就可以了，"import ${path}.modulename"。

### 示例一. 使用c代码

```
+-- cfib.c
+-- cfib.h
+-- fib.pyx
+-- setup.py
+-- main.py
```

```
//cfib.h
#ifndef __CFIB_H__
#define __CFIB_H__
unsigned long fib(unsigned long n);
#endif
```

```
//cfib.c
#include "cfib.h"
unsigned long fib(unsigned long n) {
    unsigned long a=0, b=1, i, tmp;
    for (i=0; i<n; ++i) {
        tmp = a; a = a + b; b = tmp;
    }
    return a;
}
```

```
//fib.pyx
cdef extern from "cfib.h":
    unsigned long _fib "fib"(unsigned long n)

def fib_c(n):
    ''' Returns the nth Fibonacci number.'''
    return _fib(n)

def fib_cython(n):
    '''Returns the nth Fibonacci number.'''
    a, b = 0, 1
    for i in range(n):
        a, b = a + b, a
    return a

def fib_cython_optimized(unsigned long n):
    '''Returns the nth Fibonacci number.'''
    cdef unsigned long a=0, b=1, i
    for i in range(n):
        a, b = a + b, a
    return a
```

```
//setup.py
from distutils.core import setup, Extension
from Cython.Build import cythonize

setup(ext_modules = cythonize(Extension(name="fib", sources=["cfib.c", "fib.pyx"])))
```

右键setup.py，External Tools -> accountname编译

性能测试：

```
def fib_python(n):
    '''Returns the nth Fibonacci number.'''
    a, b = 0, 1
    for i in range(n):
        a, b = a + b, a
    return a

if __name__ == '__main__':
    import timeit
    python_setup = "from __main__ import fib_python"
    cython_setup = "import fib"
    print("Python code: ", timeit.timeit('fib_python(47)', setup=python_setup), "seconds")
    print("Cython code: ", timeit.timeit('fib.fib_cython(47)', setup=cython_setup), "seconds")
    print("Optimized Cython code: ", timeit.timeit('fib.fib_cython_optimized(47)', setup=cython_setup), "seconds")
    print("C code: ", timeit.timeit('fib.fib_c(47)', setup=cython_setup), "seconds")
    
##结果：
Python code:  2.9352622053858113 seconds
Cython code:  1.7331176511158422 seconds
Optimized Cython code:  0.14643933094340067 seconds
C code:  0.11884286952119272 seconds
```

### 示例二. 径向基函数的近似计算

- 公式：![径向基函数](formula.png)

```
//fastloop.pyx
#cython: language_level=3, boundscheck=False, wraparound=False, nonecheck=False
from libc.math cimport exp
def rbf_network(double[:, :] X, double[:] beta, double theta):
    from numpy import zeros
    cdef int N = X.shape[0]
    cdef int D = X.shape[1]
    cdef double[:] Y = zeros(N)
    cdef int i, j, d
    cdef double r = 0
    for i in range(N):
        for j in range(N):
            r = 0
            for d in range(D):
                r += (X[j, d]-X[i, d])**2
            r = r**0.5
            Y[i] += beta[j] * exp(-(r*theta)**2)

    return Y
```

```
//setup.py
from distutils.core import setup, Extension
from Cython.Build import cythonize
from Cython.Distutils import build_ext
import os

os.environ["CC"] = "gcc-8"
os.environ["CXX"] = "g++-8"

# setup(
#     ext_modules = cythonize(Extension(name="fib", sources=["cfib.c", "fib.pyx"]),
#                             language_level=3
#                             )
# )

ext_modules=[Extension(name="fastloop",
                       sources=["fastloop.pyx"],
                       libraries=["m"],
                       language='c',
                       extra_compile_args=['-O3', -ffast-math','-fopenmp'],
                       extra_link_args=['-fopenmp'],

setup(
    name='fastloop',
    cmdclass={"build_ext": build_ext},
    ext_modules=ext_modules
)
```

性能测试：

```
from math import exp
import numpy as np

setup = '''
import numpy as np;
from fastloop import rbf_network;
D = 5;
N = 1000;
X = np.array([np.random.rand(D) for d in range(N)]);
beta = np.random.rand(N);
theta = 10;'''

setup1 = '''
from __main__ import rbf_network;
import numpy as np;
from math import exp;
D = 5;
N = 1000;
X = np.array([np.random.rand(D) for d in range(N)]);
beta = np.random.rand(N);
theta = 10;'''

def rbf_network(X, beta, theta):
    N = X.shape[0]
    D = X.shape[1]
    Y = np.zeros(N)
    
    for i in range(N):
        for j in range(N):
            r = 0
            for d in range(D):
                r += (X[j, d] - X[i, d]) ** 2
            r = r ** 0.5
            Y[i] += beta[j] * exp(-(r * theta) ** 2)
    
    return Y

if __name__ == '__main__':
    import timeit
    print(timeit.timeit(stmt='rbf_network(X, beta, theta)',
                        setup=setup,
                        number=1))
    print(timeit.timeit(stmt='rbf_network(X, beta, theta)',
                        setup=setup1,
                        number=1))
```

```
##结果：
0.018067925004288554
4.64309394301381
```

### cython怎样找到性能优化点

```
$ cython xxxxx.pyx -a
```

生成的html黄色部分则是可以做性能优化的点。

如上例二fastloop.pyx，使用

```
from libc.math cimport exp
```

代替

```
from math import exp
```

### 其他

#### timeit

```
timeit.timeit(stmt='pass', setup='pass', timer=<default timer>, number=1000000, globals=None)
```

#### python调用c

使用ctypes

```
from ctypes import cdll,c_char_p,c_int,c_double
# libc.dylib必须在环境变量lib里，其他还有如libm.dylib等
cdll_names = 'libc.dylib'
clib = cdll.LoadLibrary(cdll_names)
clib.printf(c_char_p("Hello %d %f".encode('utf-8')),c_int(15),c_double(2.3))
```

#### python代码优化

- 减少变量公有，能__的就用__变为私有
- 尽量使用函数，减少全局声明和定义
- 尽可能去掉属性访问，减少.的使用次数，尤其是循环中
- 更多使用局部变量，尽量少的全局变量
- 避免不必要的抽象，可以直接访问，不要用get/set方法
- 使用内置的数据类型，自己封装的类更慢
- 避免创建不必要的数据结构或复制

### 参考文献

- [Cython入门教程](https://charlesnord.github.io/2017/03/11/cython-tuto/)
- [PyCharm & Cython](http://chingchuan-chen.github.io/posts/201804/2018-04-15-PyCharm-and-Cython.html)
- [Python调用C](https://coolshell.cn/articles/671.html)
- [知乎-Python调用C](https://zhuanlan.zhihu.com/p/20152309)
- [python基础小工具timeit](https://www.zybuluo.com/kingwhite/note/138504)
- [加速python运行](https://python3-cookbook.readthedocs.io/zh_CN/latest/c14/p14_make_your_program_run_faster.html)
- [python加速](http://www.idataskys.com/2018/04/14/python科学计算加速/)
- [stackoverflow_fix_openmp](https://stackoverflow.com/questions/16737260/how-to-tell-distutils-to-use-gcc/16737674)
- [distutils_for_gcc](https://codeday.me/bug/20171008/81374.html)
- [Parallel computing in Cython](http://nealhughes.net/parallelcomp2/)
- [Fast Python loops with Cython](http://nealhughes.net/cython1/)
