---
title: Neural style神经元网络深度学习皮克斯风美化头像
date: 2017-09-09 00:00:00
author: robin zheng
cover: true
img: https://raw.githubusercontent.com/zhengr/zhengr.github.io/master/assets/images/20170909163407.jpg
categories: AI
tags:
  - neural style
  - tensorflow
  - Pixar
---
## 用Neural style 神经元网络学习Pixar把你画的美美的

有画家将普通的人物生活照改画成皮克斯动画风，看起来非常惊艳。周末开始训练一下“神经病网络”，让机器学习一下皮克斯的画风，看能达到什么效果。

![20170909163407](https://raw.githubusercontent.com/zhengr/zhengr.github.io/master/assets/images/20170909163407.jpg)

![20170909163219](https://raw.githubusercontent.com/zhengr/zhengr.github.io/master/assets/images/20170909163219.jpg)

![20170909163244](https://raw.githubusercontent.com/zhengr/zhengr.github.io/master/assets/images/20170909163244.jpg)

![20170909163255](https://raw.githubusercontent.com/zhengr/zhengr.github.io/master/assets/images/20170909163255.jpg)

![20170909163305](https://raw.githubusercontent.com/zhengr/zhengr.github.io/master/assets/images/20170909163305.jpg)

![20170909163314](https://raw.githubusercontent.com/zhengr/zhengr.github.io/master/assets/images/20170909163314.jpg)

看起来是不是美美滴! 闲话少说， 直接上neural style开始深度学习，试着皮克斯的画风把下面的高圆圆重画一下， 看能画出啥样子，原谅我吧！要更多了解neural style关于画风的请参看论文 https://arxiv.org/pdf/1508.06576v2.pdf

![20170909163324](https://raw.githubusercontent.com/zhengr/zhengr.github.io/master/assets/images/20170909163324.jpg)
手头没有GPU服务器，AWS账户也没钱了， 就弄个普通机器跑吧， 一分钟一个迭代， 跑一千圈，得16个小时，用GeForce GTX Titan X GPU 之类都是按照分钟来的， 深度学习一直是壕的荣耀。

![20170909Capture1](https://raw.githubusercontent.com/zhengr/zhengr.github.io/master/assets/images/20170909Capture1.PNG)

![20170909Capture2](https://raw.githubusercontent.com/zhengr/zhengr.github.io/master/assets/images/20170909Capture2.PNG)

希望能画出一个不要太丑的高圆圆，天知道会成什么样子。等待吧。。。。。。。。

----------------------------------------------------1000次迭代分割线----------------------------------------------------

经过10几个小时的迭代后，运行终于结束

![20170909Capture3](https://raw.githubusercontent.com/zhengr/zhengr.github.io/master/assets/images/20170909Capture3.PNG)![img](https://raw.githubusercontent.com/zhengr/zhengr.github.io/master/assets/images/20170909Capture3.PNG)

我们看到的高圆圆成了什么样呢？

![3output1](https://raw.githubusercontent.com/zhengr/zhengr.github.io/master/assets/images/3output1.jpg)

额滴神哪， 这是啥玩意！吓死俺了，所以在学习库和硬件不够强大情况下效果还是很差强人意。其实这类深度学习搞搞莫奈，梵高那样的意识流的印象派翻到看不出什么， 这类非常精细的效果往往欠妥。

等下次有GPU系统后再测测看， 这次是失败了， 算是高圆圆进猪八戒出。

----------------------------------------------------3000次迭代分割线----------------------------------------------------

![20170909Capture4](https://raw.githubusercontent.com/zhengr/zhengr.github.io/master/assets/images/20170909Capture4.PNG)

![3output2](https://raw.githubusercontent.com/zhengr/zhengr.github.io/master/assets/images/3output2.jpg)

经过3000次迭代，效果好了一些，学习库的局限性和硬件的薄弱，做太精细的事情还是不行的。