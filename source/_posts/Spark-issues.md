---
title: Spark_issues
date: 2017-09-10 18:17:55
categories: issues
tags: 
  - spark
---

#### Issue1: 代码中spark报序列化问题。  
**原因**: spark是需要通过master分发计算的，序列化问题是因为代码中还有不能序列化的变量或函数。  
**解决方案**: 变量需要能够序列化，像pool等不可序列化的对象，可建立类去专门包装，实现Serializable接口，在分发后分别worker运行任务时再完成构造对象。尽量少使用transient等关键字。

#### Issue2: 代码运行时，提交spark://master:7077，报类找不到类的问题。  
**原因**: spark是需要通过master分发计算的,如果是spark-submit提交任务，只要jar包代码序列化正确则没什么问题，但IDE执行，需要sparkContext启动前调用setJar()设置任务jar包。  
**解决方案**: 程序测试用，可以把mvn生成的jar设置到程序中运行。一般都会使用spark-submit去提交任务。

<!-- more -->

#### Issue3: 运行jar正常，但运行依赖不能分发执行的问题。   
**原因**: 原因如问题。  
**解决方案**: mvn打包时把依赖jar使用shade方式打包，不要把依赖的资源文件打在外面。


未完待续……
