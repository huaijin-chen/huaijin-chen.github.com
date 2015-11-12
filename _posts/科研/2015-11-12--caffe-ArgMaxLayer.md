---
layout: post
title: Caffe源码分析之ArgMaxLayer
categories: 技术
tags: DeepLearning Caffe
---

ArgMaxLayer 是多SoftMaxLayer输出的对图像预测概率进行再一部的处理，从而得出top K的预测结果和相对应的概率值（if out_mal_val==ture ）。

##主要函数
- `Reshape`:  初始化输出Blob top 如果out_max_val_为true，第一通道放预测top k的label，第二通道存储top 1 的概率值。
{% highlight  C++%}
template <typename Dtype>
void ArgMaxLayer<Dtype>::Reshape(const vector<Blob<Dtype>*>& bottom,
      const vector<Blob<Dtype>*>& top) {
  if (out_max_val_) {
    // Produces max_ind and max_val
    top[0]->Reshape(bottom[0]->num(), 2, top_k_, 1); 
  } else {
    // Produces only max_ind
    top[0]->Reshape(bottom[0]->num(), 1, top_k_, 1);
  }
}
{% endhighlight %}

-`Forward_cpu`：在得到top K的label和相对的概率至存储在top 中。采用的是pair和局部排序算法。

{% highlight  C++%}
template <typename Dtype>
void ArgMaxLayer<Dtype>::Forward_cpu(const vector<Blob<Dtype>*>& bottom,
    const vector<Blob<Dtype>*>& top) {
  const Dtype* bottom_data = bottom[0]->cpu_data();
  Dtype* top_data = top[0]->mutable_cpu_data(); //输出blob的数据指针
  int num = bottom[0]->num();
  //dim 每个出入数据的一维长度（实际为N---数据的种类数）
  int dim = bottom[0]->count() / bottom[0]->num(); 
  for (int i = 0; i < num; ++i) { //每一个输入图像，num是batch的值
    std::vector<std::pair<Dtype, int> > bottom_data_vector;
    for (int j = 0; j < dim; ++j) {
      bottom_data_vector.push_back(
          std::make_pair(bottom_data[i * dim + j], j));
    }
    //求top k 存在bottom_data_vector的前k元素中
    std::partial_sort(
        bottom_data_vector.begin(), bottom_data_vector.begin() + top_k_,
        bottom_data_vector.end(), std::greater<std::pair<Dtype, int> >());
    for (int j = 0; j < top_k_; ++j) { //预测的label存在0#通道
      top_data[top[0]->offset(i, 0, j)] = bottom_data_vector[j].second;
    }
    if (out_max_val_) {
      for (int j = 0; j < top_k_; ++j) { //最大的概率值存在1#通道
        top_data[top[0]->offset(i, 1, j)] = bottom_data_vector[j].first;
      }
    }
  }
}
{% endhighlight %}
