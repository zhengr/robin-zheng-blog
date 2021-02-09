---
title: 利用KNIME Deeplearning4J Integration分析吴秀波事件女主cici
date: 2019-02-15 00:00:00
author: robin zheng
img: https://raw.githubusercontent.com/zhengr/zhengr.github.io/master/assets/images/Jietu20190215-165212.jpg
categories: AI
tags:
  - Knime
  - Deeplearning4J
---

#### 获取关键图片数据
##### 先写一个爬虫程序在Ins上扒下来1000多张ici的照片，具体的爬虫程序不在这里赘述

![Illustration](https://raw.githubusercontent.com/zhengr/zhengr.github.io/master/assets/images/Jietu20190215-163955.jpg)

##### 在这1000多cici照片集中混入其他名人，例如angelina_jolie，avril_lavigne，barack_obama等名人照片。

#### 构建分析工作流，Celebrity Detection Using Alex Net

##### Celebrity Recognition Using AlexNet
This wokflow shows an example of image classification of celebrity faces using AlexNet.
Note: Due to the size of the network, this workflow has a very long execution time.

##### Workflow Requirements
KNIME Analytics Platform 3.3.0
KNIME Deeplearning4J Integration
Deeplearning4J Integration - Image Processing (Needs to be installed separately from the KNIP Stable Community

![Illustration](https://raw.githubusercontent.com/zhengr/zhengr.github.io/master/assets/images/Jietu20190215-165212.jpg)

其中包括 30 nodes(s):

- IconConvolution Layer (5) Other
- IconPooling Layer (3) Other
- IconImage Viewer (2) 
- IconLRN Layer (2) Other
- IconDense Layer (2) Other
- IconScorer (1) Other
- IconShuffle (1) Manipulator
- IconProjector (1) 
- IconList Files (1) Source
- IconJoiner (1) Manipulator
- IconJava Snippet (1) ManipulatorStreamable
- IconImage Resizer (1) 
- IconImage Reader (Table) (1) 
- IconImage Normalizer (1) 
- IconImage Converter (1) 
- IconImage Calculator (1) 
- IconFeature Calculator (BETA) (1) Manipulator
- IconPartitioning (1) Manipulator
- IconDL4J Feedforward Predictor (Classification) (1) Predictor
- IconDL4J Feedforward Learner (Classification) (1) Learner
- IconDL4J Model Initializer (1) Other

