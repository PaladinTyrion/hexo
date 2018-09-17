---
title: Pandas_Note
date: 2018-09-17 19:10:21
categories: python
type: "tags"
tags:
  - python
---

#### Pandas知识点

- Series
- DataFrame
- 索引
- pandas基本操作
    - 重索引
    - 丢弃一行或者一列
    - 数据选取
    - 数据对齐
- pandas函数简单介绍
    - apply和applymap函数
    - 排序函数: sort_index, sort_values
    - 汇总计算函数: corr, cov
- 缺失值的处理
- 整数索引

<!-- more -->

#### Pandas Tips

- Series = 一组数据 + 数据标签
- 两个Series对象在做运算时, 会自动对齐
- Index标签可以理解为，不可变的hashset
- loc按照标签查找, 两头取闭; iloc按照index查找, 左闭右开; ix已废弃, 不要使用
- 0是对axis=0上做, 即每一列; 1是对axis=1上做, 即每一行