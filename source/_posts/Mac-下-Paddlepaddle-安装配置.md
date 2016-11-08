---
title: Mac 下 Paddlepaddle 安装配置
date: 2016-10-24 15:37:04
tags: [paddlepaddle, mac, deep learning]
---

百度新推出了`paddlepaddle`深度学习工具，目前官方还未支持 Mac 下 binary 安装。参考[最新官方文档](http://www.paddlepaddle.org/doc/build/index.html)，笔者先基于源码编译安装，发现 BLAS 依赖是个非常麻烦的坑，最后使用官方推荐的 docker 镜像方式进行安装。

<!--more-->

1. 参照官网安装 [Docker for Mac](https://docs.docker.com/docker-for-mac/)，傻瓜式安装非常简单，不再累述；
2. 判断本地 CPU 是否支持 `AVX`指令集：
```
if cat /proc/cpuinfo | grep -q avx ; then echo "Support AVX"; else echo "Not support AVX"; fi
```
3. 根据是否支持 `AVX` 选择镜像下载：

||normal    |devel  |demo|
|----|----|----|
|CPU    |cpu-latest |cpu-devel-latest   |cpu-demo-latest|
|GPU    |gpu-latest |gpu-devel-latest   |gpu-demo-latest|
|CPU WITHOUT AVX    |cpu-noavx-latest   |cpu-noavx-devel-latest |cpu-noavx-demo-latest|
|GPU WITHOUT AVX    |gpu-noavx-latest   |gpu-noavx-devel-latest |gpu-noavx-demo-latest|

虽然新版的文档只有6个镜像，但实际上上述表格中所有镜像都在同步更新（[地址](https://hub.docker.com/r/paddledev/paddle/tags/)）。 没有本文选择`cpu-noavx-latest`下载：
```
$ docker pull paddledev/paddle:cpu-noavx-latest
$ docker run -it paddledev/paddle:cpu-noavx-latest 
root@c9eea5b52828:/# paddle version
PaddlePaddle 0.8.0b3, compiled with
    with_avx: OFF
    with_gpu: OFF
    with_double: OFF
    with_python: ON
    with_rdma: OFF
    with_glog: ON
    with_gflags: ON
    with_metric_learning: 
    with_timer: OFF
    with_predict_sdk: 
```

安装成功！

##注意：
[旧版官网](http://deeplearning.baidu.com/doc/build/build_from_source.html)上提到的镜像下载语句：
```
$ docker run -it paddledev/paddlepaddle:cpu-latest
```
是**错的！！！**。仓库路径应该为`paddledev/paddle`，而非`paddledev/paddlepaddle`。这个小问题折腾了我很久，希望对读者有所帮助。

