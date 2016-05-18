---
layout: post
title:  怎么在Caffe中添加Python Layer
categories: 技术
tags: Caffe python


---


由于Python的便捷性和灵活性，同时具有无比丰富的第三方库，能快速实现迭代，在实验中非常常用。Caffe是有C++实现的，用C++ 不利于快速验证算法。但是caffe提供了Python的接口口以及自定义Python layer的接口。


## 配置Makefile.config

首先要在`Makefile.config`中 把`WITH_PYTHON_LAYER`置为`1`。加入你之前已经吧Caffe编译完成，就要现`make clean`所有编译产生的二进制文件。然后，
{% highlight  PowerShell%}
make -j8 && make pycaffe 
{% endhighlight %}
`-j8` 代表采用8核编译，具体的数字看机器本身的CPU核数。由于Caffe的编译过程比较慢，建议采用`-j`选项。

## 配置Python Layer路径

添加自定义python layer的路径是为了让Caffe能找到这个自定义的layer。有两种方法可以实现添加路径。
第一种：直接在bash环境中添加。
```
export PYTHONPATH=$PYTHONPATH:/the/path/to/PythonLayer
```
第二种：动态添加。只在运行是添加。可以在调用这个Python Layer时，预先把它的路径通过`sys.path.append`函数添加。
{% highlight  Python%}
sys.path.append('path/to/your/layer');
{% endhighlight %}

## 实现layer

实现layer有主要的五个要素

 - class name  ---这是layer的ID标识 
 - setup 函数  
 - reshape 函数 
 - forward 函数 
 - backward 函数
 
举个栗子如下：
{% highlight  Python%}
import caffe
import numpy as np

class EuclideanLossLayer(caffe.Layer):

    def setup(self, bottom, top):
        # check input pair
        if len(bottom) != 2:
            raise Exception("Need two inputs to compute distance.")

    def reshape(self, bottom, top):
        # check input dimensions match
        if bottom[0].count != bottom[1].count:
            raise Exception("Inputs must have the same dimension.")
        # difference is shape of inputs
        self.diff = np.zeros_like(bottom[0].data, dtype=np.float32)
        # loss output is scalar
        top[0].reshape(1)

    def forward(self, bottom, top):
        self.diff[...] = bottom[0].data - bottom[1].data
        top[0].data[...] = np.sum(self.diff**2) / bottom[0].num / 2.

    def backward(self, top, propagate_down, bottom):
        for i in range(2):
            if not propagate_down[i]:
                continue
            if i == 0:
                sign = 1
            else:
                sign = -1
            bottom[i].diff[...] = sign * self.diff / bottom[i].num
{% endhighlight %}

## 在网络配置中定义Python Layer

在网络配置（.prototxt）中定义自定义的python layer 需要注意是是`type：Python`，caffe把所有的Python Layer 都看做一类。但是怎么标识本层是那个python layer呢？ 在 `python_param` 中的 `layer`参数中。
{% highlight  Python%}
...
layer {
  type: 'Python'
  name: 'loss'
  top: 'loss'
  bottom: 'ipx'
  bottom: 'ipy'
  python_param {
  # the module name -- usually the filename -- that needs to be in    $PYTHONPATH
    module: 'pyloss'
    # the layer name -- the class name in the module
    layer: 'EuclideanLossLayer'
  }
  # set loss weight so Caffe knows this is a loss layer
  loss_weight: 1
}
{% endhighlight %}

## Python Layer的优缺点
在Caffe 中定义Python Layer，易于实现，定义灵活。比起C++定义新的层-简单。但是由于自定义的新层不支撑GPU前馈和后馈不能充分利用GPU的硬件资源。

总之在追求程序速度和实现速度上找到平衡点。



### 参考文献：

http://chrischoy.github.io/research/caffe-python-layer/




