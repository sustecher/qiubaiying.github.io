---
layout:     post
title:      How to to set up a deep learning demo
subtitle:   Learning note
date:       2017-06-20
author:     ZY
header-img: img/avatar_m.jpg
catalog: 	 true
tags:
    - Ubuntu
    - Pycharm
---

## How to to set up a deep learning demo - Deepmedic##

 This is a record of online tutorial about how to run the a deep learning demo efficiently based on Ubuntu.

### Python environment ###

Anaconda is the leading open data science platform powered by Python. Anaconda will make it easy to install python. This tutorial also taught us how to setup python 2 and python 3 simultaneous.

[link](http://blog.csdn.net/hua_bei/article/details/72683190)

Note

The code to create python 2 environment should be 
`conda create -n python_2 biopython`

The author lost `-n`

### Python editor - Pycharm ###

#### [Get student license](https://sales.jetbrains.com/hc/zh-cn/articles/207154369--学生授权申请方式)

#### [Offical installation guide](https://www.jetbrains.com/help/pycharm/installation-and-launching.html)

One thing shoule be noted. Setting JAVA JDK firstly.
 
[Tutoial](http://www.jianshu.com/p/e9b2369154dc)

#### [Third party method](http://www.linuxdiyf.com/linux/26442.html)

Note

Actually, it only need 3 lines code 

`sudo add-apt-repository ppa:mystic-mirage/pycharm`

`sudo apt update`

`sudo apt install pycharm`

#### [Connect Pycharm with github](http://www.jianshu.com/p/f58e38f38594)

#### Deep learning demo - [Deepmedic](https://github.com/Kamnitsask/deepmedic) 

Note:
