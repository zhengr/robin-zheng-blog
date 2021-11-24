---
title: Google 开源一个通用编码-解码框架（tf-seq2seq）
date: 2017-03-14 22:48:00
author: robin zheng
img: https://3.bp.blogspot.com/-3Pbj_dvt0Vo/V-qe-Nl6P5I/AAAAAAAABQc/z0_6WtVWtvARtMk0i9_AtLeyyGyV6AI4wCLcB/s1600/nmt-model-fast.gif
categories: 大数据
tags:
  - 大数据
  - Google
  - 通用编码
---

### 这个框架基于Tensorflow的更具体实践， 但并非是一个官方的产品。 ###

A general-purpose encoder-decoder framework for Tensorflow that can be used for Machine Translation, Text Summarization, Conversational Modeling, Image Captioning, and more.

- 项目介绍：https://google.github.io/seq2seq
- 代码地址：https://github.com/google/seq2seq 
- 相关论文：https://arxiv.org/pdf/1703.01619.pdf



![](https://3.bp.blogspot.com/-3Pbj_dvt0Vo/V-qe-Nl6P5I/AAAAAAAABQc/z0_6WtVWtvARtMk0i9_AtLeyyGyV6AI4wCLcB/s1600/nmt-model-fast.gif)

当前能看到的示例只有Machine Translation（机器翻译），Text Summarization,Image Captioning示例还未推出。

### 模型 ###

- ModelBase
- Seq2SeqModel
- > ***BasicSeq2Seq***
- > ***AttentionSeq2Seq***
- > ***Image2Seq***


