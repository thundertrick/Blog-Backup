---
title: Mac 下 Paddlepaddle 安装配置
date: 2016-10-24 15:37:04
tags: [paddlepaddle, mac, deep learning]
---

百度新推出了`paddlepaddle`深度学习工具，目前官方还未支持 Mac 下 binary 安装。本文参考[官方文档](http://deeplearning.baidu.com/doc/build/build_from_source.html)，尝试使用源码本地编译安装尝尝鲜 (CPU Only)。

<!-- more -->
## 依赖库
1. cmake: `brew install cmake`
2. BLAS: Mac 系统下面的 vecLib 框架（Accelerate Frame）本身就集成了 Atlas的实现，直接使用即可
3. protobuf: `brew install protobuf`
4. Python: Mac 自带 Python 2.7 和 numpy


官方推荐安装：
1. glog, gflags: `brew install glog gflags`
2. gtest:

~~~shell
# Install google test on Mac OS X
# Download gtest 1.7.0
wget https://github.com/google/googletest/archive/release-1.7.0.tar.gz
tar -xvf googletest-release-1.7.0.tar.gz && cd googletest-release-1.7.0
# Build gtest
mkdir build 
cd build
cmake ..
make
# Install gtest library
sudo cp -r ../include/gtest /usr/local/include/
sudo cp lib*.a /usr/local/lib
~~~

## 编译

todo: 编译提示缺失 BLAS：

~~~shell
...
-- Found GTest: /usr/local/lib/libgtest.a  
-- Found Sphinx: /Users/xuyang/anaconda/bin/sphinx-build  
-- Could NOT find Doxygen (missing:  DOXYGEN_EXECUTABLE) 
CMake Error at cmake/cblas.cmake:127 (message):
  CBlas must be set.  Paddle support MKL, ATLAS, OpenBlas, reference-cblas.
  Try set MKL_ROOT, ATLAS_ROOT, OPENBLAS_ROOT or REFERENCE_CBLAS_ROOT.
Call Stack (most recent call first):
  CMakeLists.txt:26 (include)


-- Configuring incomplete, errors occurred!
See also "/Users/xuyang/Desktop/work/paddle/build/CMakeFiles/CMakeOutput.log".
~~~

显然是没有引用默认路径，搜了一圈发现 vecLib.Framework 的路径在`/System/Library/Frameworks/Accelerate.framework/Versions/Current/Frameworks/vecLib.framework/Versions/A`。问题是怎么加到 ATLAS_ROOT 里呢？尝试添加`-DATLAS_ROOT=...`和设置环境变量`ATLAS_ROOT=...`均无效。官方建议的依赖安装是`brew install glog gflags cmake protobuf openblas`，看来是支持`openblas`的默认路径。所以索性替换为`openblas`。

参考一篇[caffe的配置教程](http://blog.csdn.net/surgewong/article/details/43708339)，使用`brew install  homebrew/science/openblas`安装。

// to be continued...

