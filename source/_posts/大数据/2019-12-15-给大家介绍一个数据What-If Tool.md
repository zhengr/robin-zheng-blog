---
title: 给大家介绍一个数据What-If Tool
date: 2019-12-15 00:00:00
author: robin zheng
img: https://github.com/PAIR-code/what-if-tool/raw/master/img/wit-smile-intro.png
categories: 大数据
tags:
  - 大数据
  - Google
  - What-if
---
机器学习世界的研究重点稍微从探索“模型 能够理解什么”，转向了研究“模型是如何理解的”。由于人们担心引入偏差，或者过度泛化模型的适用性，因此开发出了What-If Tool（WIT）这样的工具。这个工具可帮助数据科学家深入研究模型的行为，并将各种功能和数据集对输出的影响进行可视化。WIT由Google引入，简化了比较模型、切片数据集、可视化构面和编辑单个数据点等任务，并可以在Tensorboard 或Jupyter notebooks中使用。尽管WIT可以帮助执行分析，但研究者仍然需要对模型背后的数学 和理论有深刻的理解。WIT只是数据科学家 用来深入了解模型行为的工具，对于使用不当或缺乏训练不佳的算法，初级用户不应奢望有任何工具可以消除或减轻其风险或造成的损害。

在数据中我们对What-If充满了好奇，总是想从上下文关联推断中获得对未来假设的展望。

![wit-smile-intro.png](https://github.com/tensorflow/tensorboard/blob/master/tensorboard/plugins/interactive_inference/img/wit-smile-intro.png?raw=true)

What-If Tool(WIT)为扩展对黑箱分类或回归ML模型的理解提供了一个易于使用的接口。使用该插件，您可以对大量示例执行推理，并立即以各种方式可视化结果。此外，可以手动或以编程方式编辑示例并重新运行模型，以查看更改的结果。它包含用于研究模型性能和数据集子集公平性的工具。

To build the web demos yourself:

- Binary classifier for UCI Census dataset salary prediction
  - Dataset: [UCI Census](https://archive.ics.uci.edu/ml/datasets/census+income)
  - Task: Predict whether a person earns more or less than $50k based on their census information
  - To build and run the demo from code: `bazel run tensorboard/plugins/interactive_inference/tf_interactive_inference_dashboard/demo:demoserver` then navigate to `http://localhost:6006/tf-interactive-inference-dashboard/demo.html`
- Binary classifier for smile detection in images
  - Dataset: [CelebA](http://mmlab.ie.cuhk.edu.hk/projects/CelebA.html)
  - Task: Predict whether the person in an image is smiling
  - To build and run the demo from code: `bazel run tensorboard/plugins/interactive_inference/tf_interactive_inference_dashboard/demo:imagedemoserver` then navigate to `http://localhost:6006/tf-interactive-inference-dashboard/image_demo.html`
- Multiclass classifier for Iris dataset
  - Dataset: [UCI Iris](https://archive.ics.uci.edu/ml/datasets/iris)
  - Task: Predict which of three classes of iris flowers that a flower falls into based on 4 measurements of the flower
  - To build and run the demo from code: `bazel run tensorboard/plugins/interactive_inference/tf_interactive_inference_dashboard/demo:irisdemoserver` then navigate to `http://localhost:6006/tf-interactive-inference-dashboard/iris_demo.html`
- Regression model for UCI Census dataset age prediction
  - Dataset: [UCI Census](https://archive.ics.uci.edu/ml/datasets/census+income)
  - Task: Predict the age of a person based on their census information
  - To build and run the demo from code: `bazel run tensorboard/plugins/interactive_inference/tf_interactive_inference_dashboard/demo:agedemoserver` then navigate to `http://localhost:6006/tf-interactive-inference-dashboard/age_demo.html`
  - This demo model returns attribution values in addition to predictions (through the use of vanilla gradients) in order to demonstate how the tool can display attribution values from predictions.

https://pair-code.github.io/what-if-tool/index.html 洞悉数据的利器，Google PAIR（[People + AI Research initiative (PAIR)](https://www.google.ai/pair) ）出品。

