---
title: 'TensorFlow Datasets: The Bad Parts'
date: 2020-08-02 13:38:08
tags:
  - TensorFlow
  - Deep Learning
---

本文是[https://determined.ai/blog/tf-dataset-the-bad-parts/](https://determined.ai/blog/tf-dataset-the-bad-parts/)的译文。我在尝试用TensorFlow参照PyTorch代码复现一些模型的时候也产生了同样的想法。本文仅供个人研究使用。

***

**TLDR**: TensorFlow的`tf.data` API是一个常用的将数据加载进深度模型的方法。虽然`tf.data`有很多强大的功能，但是它是围绕顺序读取数据集设计的。这个设计导致它无法高效实现一些功能：shuffle大数据集，分布式训练时对数据进行分片，实现容错训练（fault-tolerant training）。我们认为，在构建深度学习数据读取API时，随机访问应该是需要考虑的关键因素。

TODO

