# MXNET 服务器环境搭建

## 搭建环境

已有环境

- Ubuntu 16.04.6 LTS

- CUDA-10

- python3.5



1. 下载mxnet源码

```
# clone源码
git clone https://github.com/dmlc/mxnet.git ~/mxnet --recursive
```

2. 编辑配置文件

```
# 编辑配置文件： mxnet/make/config.mk
USE_CUDA = 1 
USE_CUDA_PATH = /usr/local/cuda 
USE_OPENCV = 1
USE_MKLDNN = 0
```

3. 编译

```
# 使用全部cpu核编译（不报错就编译成功）
make -j$(nproc)
```

# 基于Anaconda 搭建pytorch

```shell
conda create -n pytorch1.0 python=3.6

conda install jupyter notebook

conda install cudnn
```







