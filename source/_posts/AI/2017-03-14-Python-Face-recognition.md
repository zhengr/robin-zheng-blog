---
title: 人脸识别——Python Face_recognition 安装
date: 2017-03-14 00:00:00
author: robin zheng
categories: AI
tags:
  - face_recognition
  - dlib
  - faceid
---

### 安装git， cmake，python-pip

sudo apt-get install  -y git

sudo apt-get install  -y cmake

sudo apt-get install  -y python-pip

### 安装dlib

```
sudo apt-get install libboost-all-dev

#编译dlib

git clone https://github.com/davisking/dlib.git

cd dlib

mkdir build

cd build

cmake .. -DDLIB_USE_CUDA=0 -DUSE_AVX_INSTRUCTIONS=1

cmake --build .

cd ..

python setup.py install --yes USE_AVX_INSTRUCTION --no DLIB_USE_CUDA
```



### 安装face_recognition

```
pip3 install face_recognition
```