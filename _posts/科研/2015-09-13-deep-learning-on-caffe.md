# 欢迎使用 Cmd Markdown 编辑阅读器

---
layout: post
title: "手势识别-DeepLearning"
categories: 科研 ML DeepLearning
- 
tags: DeepLearning Caffe Gesture Recognition
- 
## 安装Caffe
网上安装Caffe的博客有很多，但是有很多并不是很正确。本文主要是参考了[这篇博客](https://gist.github.com/bearpaw/c38ef18ec45ba6548ec0) 
## 准备数据集
本文主要采用是paper *“In-air Gestures Around Unmodified Mobile Devices”*的数据集。这个数据集可以在 [这里](http://ait.inf.ethz.ch/projects/2014/InAirGesture/) 下载到。
这个数据集有**7个手势**， **16k testing images, 16k traning images**. 

##转换数据格式
把普通的图像格式转化为Caffe支持的leve DB 或者 LMDB 格式。这是两个不同的“key-v”数据库。 
1. 把下载好的数据放到`CaffeROOT/data/handgesture` 中
``` shell
#！/usr/bin/env sh
##file conver_iamges.sh
TOOLS=./tools
DATA_DIR=./data/handgesture
$TOOLS/convert_imageset.bin \ 
    -shuffle=true \                       #random
    -backend=leveldb \                    #formate
    -resize_width=64 -resize_height=64 \  # resize image 
	$DATA_DIR/ \                          #path of data
	#training data's file list and lable
	$DATA_DIR/trainVal_l.txt  \ 
	$DATA_DIR/traindb
$TOOLS/convert_imageset.bin    -shuffle=true \
    -backend=leveldb \
    -resize_width=64 -resize_height=64  \
	$DATA_DIR/ \
	$DATA_DIR/testVal_l.txt   \
	$DATA_DIR/testdb 
	
```
在linux下，数据的的文件列表和label可以用awk脚本生成,最后生成的文件如下
```python

Half_Training/gesture1/RGB/data929.jpg 0
Half_Training/gesture1/RGB/data167.jpg 0
Half_Training/gesture1/RGB/data336.jpg 0
Half_Training/gesture1/RGB/data2498.jpg 0
Half_Training/gesture1/RGB/data2164.jpg 0
Half_Training/gesture1/RGB/data1333.jpg 0
.............
```
在convert dataset 的数据文件在`traindb` 和 `testdb` 两个文件夹中。

##计算mean
```shell
#!/usr/bin/env sh 
##file computerMean.sh
TOOLS=./build/tools

DATA_DIR=./data/handgesture
$TOOLS/compute_image_mean.bin \
-backend=leveldb \
$DATA_DIR/traindb_64  \
$DATA_DIR/img_mean_64.binaryproto

```
运行`computerMean.sh` 之后在`DATA_DIR`下有 `img_mean_64.binaryproto`文件生成。
## 构建NN的模型
Caffe采用的是用protobuf 来定义网络。本文是参考cifar10 的网络架构。

1 把`./example/cifar10/cifar10_full_train_test.prototxt`   拷贝到 `./example/handgesture/gesture_train_test.prototxt`。
2. 把`./example/cifar10/cifar10_full_solver.prototxt ` 拷贝到 `./example/handgesture/gesture_solver.prototxt` 
3.修改`gesture_train_test.prototxt`中 
``` shell
name: "Gesture"
layer {
  name: "cifar"
  type: "Data"
  top: "data"
  top: "label"
  include {
    phase: TRAIN
  }
  transform_param {
    mean_file: "examples/handgesture/img_mean_64.binaryproto"
  }
  data_param {
    source: "examples/handgesture/traindb"
    batch_size: 100
    backend: LEVELDB
  }
}
layer {
  name: "cifar"
  type: "Data"
  top: "data"
  top: "label"
  include {
    phase: TEST
  }
  transform_param {
    mean_file: "examples/handgesture/img_mean_64.binaryproto"
  }
  data_param {
    source: "examples/handgesture/traindb"
    batch_size: 100
    backend: LEVELDB
  }
}
........
```
  4  #修改最后输出层的num_output 为要分类数据的类数
```shell

  inner_product_param {
    num_output: 7 
    weight_filler {
      type: "gaussian"
      std: 0.1
    }
    bias_filler {
      type: "constant"
    }
  }
}
..........
```
5. 修改`gesture_solver.prototxt` 中的参数
```shell
# reduce learning rate after 120 epochs (60000 iters) by factor 0f 10
# then another factor of 10 after 10 more epochs (5000 iters)

# The train/test net protocol buffer definition
net: "examples/handrecognition/gesture_train_test.prototxt"
# test_iter specifies how many forward passes the test should carry out.
# In the case of CIFAR10, we have test batch size 100 and 100 test iterations,
# covering the full 10,000 testing images.
test_iter: 100
# Carry out testing every 1000 training iterations.
test_interval: 1000
# The base learning rate, momentum and the weight decay of the network.
base_lr: 0.001
momentum: 0.9
weight_decay: 0.004
# The learning rate policy
lr_policy: "fixed"
# Display every 200 iterations
display: 200
# The maximum number of iterations
max_iter: 10000
# snapshot intermediate results
snapshot: 10000
snapshot_prefix: "examples/handrecognition/handRecognition_full"
# solver mode: CPU or GPU
solver_mode: GPU

```

##最后

在`./caffe`运行 
```shell
#!/usr/bin/env sh

TOOLS=./build/tools
$TOOLS/caffe train \
    --solver=examples/handrecognition/gesture_solver.prototxt
```
所有的就ok了，就等着结果吧。在本文中的参数下，这个数据集的测试准确率是`99.04%`


---

