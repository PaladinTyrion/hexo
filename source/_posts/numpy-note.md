---
title: numpy_note
date: 2018-09-13 15:38:26
categories: python
type: "tags"
tags:
  - python
---

#### numpy知识点

- 构建numpy对象
- 从序列型对象中构造
- 用numpy自带函数构建
- ndarray的数据类型
- 索引和切片
- 切片
- 多维索引
- 花式索引
- 布尔索引
<!-- more -->
- ufunc通用函数
- ufunc函数的定义和优势
- 自定义ufunc
- numpy常用的函数: sum, mean, std, var, min, max, argmin, argmax, cumsum, cumprod
- 数学统计函数
- 改变numpy数据结构的函数
- where函数
- 线性代数函数
- 排序操作
- 随机数
- 广播机制
- matrix对象
- 数据的持久化

##### numpy tips

- astype默认是copy的, 不会改变原来的数组
- 切片是视图, 也就是原来数据的引用, 我们对视图的修改会体现在原来的数据上
- 高维数组的切片也是一个视图
- 花式索引: 用整数数组进行索引
- 花式索引不是视图, 他会拷贝数据
- bool索引也是copy的, 不是视图
- ufunc通用函数: 对ndarray执行元素级运算的函数
- reshape维度中出现-1可以表示自适应, 另外reshape也是视图
- ravel不产生副本, flattern产生副本。换句话说, ravel返回的是视图, flatten返回的是副本
- data.sort()是视图, np.sort()是副本
- 广播机制: 如果两个数组的trailing dimension相同, 或者其中一方的为1, 那么广播会在缺失和长度为1的维度上进行
- trailing dimension就是从尾部开始计算对齐
- <class 'numpy.matrixlib.defmatrix.matrix'>可以完成类似<class 'numpy.ndarray'>的高级计算