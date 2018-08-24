---
title: Xgboost
date: 2017-09-10 18:26:26
categories: deeplearning
type: "tags"
tags: 
  - 机器学习
---

#### Windows下安装Xgboost

##### 所需的必要软件：
- Git
- MinGW
- Python

<!-- more -->

##### 安装：
1. Git安装及使用省略，需注意，Windows下要安装一个git bash，为使用Linux命令。

2. Windows下Python的安装自行官网。

3. MinGW安装

    [MinGW安装器下载地址](http://iweb.dl.sourceforge.net/project/mingw-w64/Toolchains%20targetting%20Win32/Personal%20Builds/mingw-builds/installer/mingw-w64-install.exe)
    
    安装时注意：

- Architecture 选择x86_64
- Threads 选择posix
- Exception 选择seh

完成安装后将MinGW的bin添加到PATH环境变量中。
    
例如，我的安装地址是

```
D:\MinGW\mingw64
```

将 D:\MinGW\mingw64\bin 添加到环境变量PATH中。

```
$ which mingw32-make
```

可查看环境变量是否添加成功。成功则MinGW安装完成。

4. 下载Xgboost并编译

```
$ git clone --recursive https://github.com/dmlc/xgboost
$ cd xgboost
$ git submodule init
$ git submodule update

$ alias make='mingw32-make' # 为了在windows上使用MinGW进行编译
# 以下使用MinGW在windows进行编译
$ cp make/mingw64.mk config.mk
$ make -j4  #不报错则编译成功

# 最后安装成为python包
$ cd python-package
$ python setup.py install  # (或者) sudo python setup.py install
```

至此Xgboost安装完成。


#### Linux/MacOS下安装Xgboost

##### 所需的必要软件：
- Git
- Python

##### 参考Xgboost官网即可
1. Linux下

```
$ git clone --recursive https://github.com/dmlc/xgboost
$ cd xgboost
$ make -j4
```

2. MacOS下

```
$ git clone --recursive https://github.com/dmlc/xgboost
$ cd xgboost
$ cp make/minimum.mk ./config.mk
$ make -j4
```

3. 两个环境下都安装为Python包

```
# path/to/xgboost
$ cd python-package
$ sudo python setup.py install
```

至此Xgboost安装完成。


#### 测试Xgboost的程序

```
# import os
import xgboost as xgb
import numpy as np

# mingw_path = 'D:\MinGW\mingw64\bin'
# os.environ['PATH'] = mingw_path + ';' + os.environ['PATH']

if __name__ == '__main__':
    data = np.random.rand(5, 10)
    print('data', data)

    label = np.random.randint(2, size=5)
    dtrain = xgb.DMatrix(data, label=label)

    dtest = dtrain

    param = {'bst:max_depth': 2, 'bst:eta': 1, 'silent': 1, 'objective': 'binary:logistic'}
    param['nthread'] = 4
    param['eval_metric'] = 'auc'

    evallist = [(dtest, 'eval'), (dtrain, 'train')]

    num_round = 10
    bst = xgb.train(param, dtrain, num_round, evallist)

    bst.dump_model('dump.raw.txt')
```

#### 参考文献
- https://xgboost.readthedocs.io/en/latest/build.html
- http://www.jianshu.com/p/5b3e0489f1a8
