--
layout:     post
title:      Install MatCovNet
subtitle:   Learning note
date:       2017-07-27
author:     ZY
header-img: img/avatar_m.jpg
catalog: 	 true
tags:
    - MATLAB
    - Deep learning
    
---

# 写在前面
折腾了一段时间的Python，感觉对于我这种编程基础不好的小白来说，直接上python搞深度学习，有难度。再加上老板总是想要结果，所以，还算换回MATLAB来做初期的工作吧。
MatConvNet听说是个不错的工具箱，安装整整花了一天，其中有深坑。
## 软件硬件环境
>Ubuntu 16.04.2
>Dell OptiPlex 7040
>GTX 750ti
>MATLAB 2016a
>CUDA 8.0
>gcc 5.4.0 
>g++ 4.7.0
>GTX 750ti

## 与C++编译器的兼容性
------
  windows平台就是考虑VS版本，Ubuntu系统就是考虑gcc和g++的版本. 详细怎么搭配可以看code，也可以看文档，附录也摘录了一部分。

##  安装流程
以下操作代码均在MATLAB中实现。

- [x] 下载
到 [官网](http://www.vlfeat.org/matconvnet) 下载，解压缩。
也可以用以下代码实现
```matlab
% Install and compile MatConvNet (needed once).
untar('http://www.vlfeat.org/matconvnet/download/matconvnet-1.0-beta24.tar.gz') ;
cd matconvnet-1.0-beta24
run matlab/vl_compilenn ;
```
- [x] 安装
```matlab
addpath matlab
run matlab/vl_compilenn ;
```
- [x] 测试
```matlab
% Download a pre-trained CNN from the web (needed once).
urlwrite(...
  'http://www.vlfeat.org/matconvnet/models/imagenet-vgg-f.mat', ...
  'imagenet-vgg-f.mat') ;

% Setup MatConvNet.
run matlab/vl_setupnn ;

% Load a model and upgrade it to MatConvNet current version.
net = load('imagenet-vgg-f.mat') ;
net = vl_simplenn_tidy(net) ;

% Obtain and preprocess an image.
im = imread('peppers.png') ;
im_ = single(im) ; % note: 255 range
im_ = imresize(im_, net.meta.normalization.imageSize(1:2)) ;
im_ = im_ - net.meta.normalization.averageImage ;

% Run the CNN.
res = vl_simplenn(net, im_) ;

% Show the classification result.
scores = squeeze(gather(res(end).x)) ;
[bestScore, best] = max(scores) ;
figure(1) ; clf ; imagesc(im) ;
title(sprintf('%s (%d), score %.3f',...
   net.meta.classes.description{best}, best, bestScore)) ;
```

### 查错Debug
安装这一步中，产生了非常恶心得一个错误。
```matlab
Error using mex
[You path]/matconvnet/matlab/mex/.build/vl_imreadjpeg.o: In function `Batch::Item::Item(Batch const&)':
vl_imreadjpeg.cpp:(.text+0xbe3): undefined reference to `__warn_memset_zero_len'
collect2: error: ld returned 1 exit status

Error in vl_compilenn>mex_link (line 547)
mex(mopts{:}) ;

Error in vl_compilenn (line 498)
  mex_link(opts, objs, mex_dir, flags.mexlink) ;
```
而关于这个错误得解释[众说纷纭](https://github.com/vlfeat/matconvnet/issues/779)

总结一下，主要有以下几种。一下命令是在ubuntu 得terminal里。
#### 1）没有安装libjpeg包
解决方法 ：
```
sudo apt-get install build-essential libjpeg-turbo8-dev
```
#### gcc版本与MATLAB版本不匹配
虽然报warning，建议gcc 4.7，但是最后我用了gcc5.4,也能跑通。
在这里还是记录一下gcc版本切换的命令。
解决方法：
```
ls /usr/bin/gcc* %查看已安装得gcc版本清单
sudo apt-get install gcc-4.7 gcc-4.7-multilib g++-4.7 g++-4.7-multilib  % 安装4.7版本得gcc和 g++包
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-4.7 100
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-4.7 100
% 设置版本优先级，最后一个数字越大，优先级越高。
sudo update-alternatives --config gcc %查看设置
```

#### 64位得Ubuntu 16可能和这个版本不兼容
相关讨论在[这里](https://github.com/vlfeat/matconvnet/issues/770)
解决方法：
```
cd /usr/local/MATLAB/R2016a/sys/os/glnxa64  #where I installed my matlab
sudo mv libstdc++.so.6.0.20 bak-libstdc++.so.6.0.20
sudo mv libstdc++.so.6 bak-libstdc++.so.6
sudo ln -sf /usr/lib/x86_64-linux-gnu/libstdc++.so.6.0.21 ./
sudo ln -sf ./libstdc++.so.6.0.21 ./libstdc++.so.6
```

#### 需要关掉一个东西
悲哀的是，尝试了上述三种方法，我的还是报错。是的，明明已经安装好了libjpeg,可它还是提示这里有问题。陷入了苦苦的思索和乱尝试。最后在官方的[安装说明](http://www.vlfeat.org/matconvnet/install/)中找到了灵感，解决了Bug。

>Remark: The 'vl_imreadjpeg' tool uses an external image library to load images. In macOS and Windows, the default is to use the system libraries (Quartz and GDI+ respectively), so this dependency is immaterial. In Linux, this tool requires the LibJPEG library and the corresponding development files to be installed in the system. If needed, the ImageLibraryCompileFlags and ImageLibraryLinkFlags options can be used to adjust the compiler and linker flags to match a specific library installation. It is also possible to use the EnableImreadJpeg option of vl_compilenn to turn off this feature.

解决方法：
打开vl_compilenn.m文件。
将
```
opts.enableImreadJpeg = true;
```
改成
```
opts.enableImreadJpeg = false;
```


# 附录 代码中给得版本匹配表
这应该只是给的建议版，像我已经装好了Ubuntu 16，CUDA 8.0, MATLAB2016a, 并且搞了一些其他包了。然而CUDAU7.5只支持Ubuntu14 和15。总不能为了这个重装吧，
最后还是迎着头皮搞了，发现也没碍事儿。
## MATLAB 与CUDA版本匹配
| MATLAB version | Release | CUDA Devkit  |
|:----------------:|:---------:|:--------------:|
| 8.2            | 2013b   | 5.5          |
| 8.3            | 2014a   | 5.5          |
| 8.4            | 2014b   | 6.0          |
| 8.6            | 2015b   | 7.0          |
| 9.0            | 2016a   | 7.5          |

## C编译器版本兼容
作者在代码注释中，写到以下配置被检测成功运行。（没说其他搭配一定不成功）

| 系统版本 | RMATLAB版本 | C编译器版本  |CUDA版本|
|:----------------:|:---------:|:--------------:|
| Windows 7 x64           | MATLAB R2014a   | Visual C++ 2010, 2013    |6.5|
| Windows 7 x64           | MATLAB R2014a   | Visual C++ 2015   |不支持|
|Windows 8 x64          | MATLAB R2014a  | Visual C++ 2013 and CUDA     |6.5|
| Mac OS X 10.9, 10.10, 10.11          | MATLAB R2013a to R2016a   | 6.0          |5.5 to 7.5|
| GNU/Linux           |MATALB R2014a/R2015a/R2015b/R2016a   | 7.0          |gcc/g++|5.5/6.5/7.5|