---
title: 实战人工智能（TensorFlow）在证券量化交易中的突破
date: 2017-10-29 00:00:00
author: robin zheng
img: https://raw.githubusercontent.com/zhengr/zhengr.github.io/master/assets/images/20171029Capture2.PNG
categories: AI
tags:
  - 量化交易
  - tensorflow
  - Quant
---
近年来各类量化交易平台层出不穷， 多数是基于类似（anaconda + 证券数据）这样的数据建模+分析平台， 通过Python的强大数据处理能力来构建量化交易策略。近期测用了一个借助Tensorflow深度学习技术的量化工具，将人工智能概念与量化交易融合，回测结果非常惊人，远超沪深300。

## 可视化策略设计

![20171029Capture2](https://raw.githubusercontent.com/zhengr/zhengr.github.io/master/assets/images/20171029Capture2.PNG)

## 执行基于深度学习的量化交易策略 

```verilog
[2017-10-30 04:23:51.860846] INFO: quant: instruments.v2 开始运行..
[2017-10-30 04:23:51.900663] INFO: quant: instruments.v2 运行完成[0.039842s].
[2017-10-30 04:23:51.984478] INFO: quant: advanced_auto_labeler.v2 开始运行..
[2017-10-30 04:24:05.050702] INFO: 自动数据标注: 加载历史数据: 4451670 行
[2017-10-30 04:24:05.051990] INFO: 自动数据标注: 开始标注 ..
[2017-10-30 04:24:31.427443] INFO: quant: advanced_auto_labeler.v2 运行完成[39.442966s].
[2017-10-30 04:24:31.436329] INFO: quant: input_features.v1 开始运行..
[2017-10-30 04:24:31.445372] INFO: quant: 命中缓存
[2017-10-30 04:24:31.446037] INFO: quant: input_features.v1 运行完成[0.009731s].
[2017-10-30 04:24:31.470692] INFO: quant: general_feature_extractor.v6 开始运行..
[2017-10-30 04:24:43.465794] INFO: 基础特征抽取: 年份 2010, 特征行数=431567
[2017-10-30 04:24:57.637034] INFO: 基础特征抽取: 年份 2011, 特征行数=511455
[2017-10-30 04:25:13.418651] INFO: 基础特征抽取: 年份 2012, 特征行数=565675
[2017-10-30 04:25:28.853081] INFO: 基础特征抽取: 年份 2013, 特征行数=564168
[2017-10-30 04:25:45.124617] INFO: 基础特征抽取: 年份 2014, 特征行数=569948
[2017-10-30 04:25:59.190550] INFO: 基础特征抽取: 年份 2015, 特征行数=569698
[2017-10-30 04:26:15.774595] INFO: 基础特征抽取: 年份 2016, 特征行数=641546
[2017-10-30 04:26:30.497031] INFO: 基础特征抽取: 年份 2017, 特征行数=597613
[2017-10-30 04:26:30.527335] INFO: 基础特征抽取: 总行数: 4451670
[2017-10-30 04:26:30.537181] INFO: quant: general_feature_extractor.v6 运行完成[119.066492s].
[2017-10-30 04:26:30.547465] INFO: quant: derived_feature_extractor.v2 开始运行..
[2017-10-30 04:26:34.316964] INFO: derived_feature_extractor: 提取完成 avg_amount_0/avg_amount_5, 0.009s
[2017-10-30 04:26:34.329348] INFO: derived_feature_extractor: 提取完成 avg_amount_5/avg_amount_20, 0.011s
[2017-10-30 04:26:34.341731] INFO: derived_feature_extractor: 提取完成 rank_avg_amount_0/rank_avg_amount_5, 0.011s
[2017-10-30 04:26:34.352090] INFO: derived_feature_extractor: 提取完成 rank_avg_amount_5/rank_avg_amount_10, 0.009s
[2017-10-30 04:26:34.361827] INFO: derived_feature_extractor: 提取完成 rank_return_0/rank_return_5, 0.008s
[2017-10-30 04:26:34.371729] INFO: derived_feature_extractor: 提取完成 rank_return_5/rank_return_10, 0.009s
[2017-10-30 04:26:39.481508] INFO: derived_feature_extractor: /y_2010, 431567
[2017-10-30 04:26:39.912103] INFO: derived_feature_extractor: /y_2011, 511455
[2017-10-30 04:26:40.795991] INFO: derived_feature_extractor: /y_2012, 565675
[2017-10-30 04:26:41.290907] INFO: derived_feature_extractor: /y_2013, 564168
[2017-10-30 04:26:41.778812] INFO: derived_feature_extractor: /y_2014, 569948
[2017-10-30 04:26:42.304537] INFO: derived_feature_extractor: /y_2015, 569698
[2017-10-30 04:26:43.185396] INFO: derived_feature_extractor: /y_2016, 641546
[2017-10-30 04:26:43.997410] INFO: derived_feature_extractor: /y_2017, 597613
[2017-10-30 04:26:44.741970] INFO: quant: derived_feature_extractor.v2 运行完成[14.194475s].
[2017-10-30 04:26:44.753794] INFO: quant: join.v3 开始运行..
[2017-10-30 04:27:01.503429] INFO: join: /y_2010, 行数=431030/431567, 耗时=13.222894s
[2017-10-30 04:27:26.105056] INFO: join: /y_2011, 行数=510922/511455, 耗时=24.586824s
[2017-10-30 04:27:59.728027] INFO: join: /y_2012, 行数=564582/565675, 耗时=33.589178s
[2017-10-30 04:28:26.234889] INFO: join: /y_2013, 行数=563137/564168, 耗时=26.479275s
[2017-10-30 04:28:58.825867] INFO: join: /y_2014, 行数=567874/569948, 耗时=32.568128s
[2017-10-30 04:29:27.996655] INFO: join: /y_2015, 行数=560424/569698, 耗时=29.145789s
[2017-10-30 04:29:58.872081] INFO: join: /y_2016, 行数=637424/641546, 耗时=30.851773s
[2017-10-30 04:30:30.941011] INFO: join: /y_2017, 行数=576562/597613, 耗时=32.041855s
[2017-10-30 04:30:31.102396] INFO: join: 最终行数: 4411955
[2017-10-30 04:30:31.104637] INFO: quant: join.v3 运行完成[226.350825s].
[2017-10-30 04:30:31.119058] INFO: quant: dropnan.v1 开始运行..
[2017-10-30 04:30:31.776377] INFO: dropnan: /y_2010, 423741/431030
[2017-10-30 04:30:33.517425] INFO: dropnan: /y_2011, 504726/510922
[2017-10-30 04:30:34.449474] INFO: dropnan: /y_2012, 561109/564582
[2017-10-30 04:30:35.291505] INFO: dropnan: /y_2013, 563107/563137
[2017-10-30 04:30:37.202618] INFO: dropnan: /y_2014, 566035/567874
[2017-10-30 04:30:39.190745] INFO: dropnan: /y_2015, 558148/560424
[2017-10-30 04:30:42.708962] INFO: dropnan: /y_2016, 635564/637424
[2017-10-30 04:30:43.383097] INFO: dropnan: /y_2017, 571976/576562
[2017-10-30 04:30:43.415940] INFO: dropnan: 行数: 4384406/4411955
[2017-10-30 04:30:43.436363] INFO: quant: dropnan.v1 运行完成[12.317303s].
[2017-10-30 04:30:43.593560] INFO: quant: stock_ranker_train.v5 开始运行..
[2017-10-30 04:31:31.995534] INFO: df2bin: prepare bins ..
[2017-10-30 04:31:38.116864] INFO: df2bin: prepare data: training ..
[2017-10-30 04:32:22.274530] INFO: df2bin: sort ..
[2017-10-30 04:34:25.088080] INFO: stock_ranker_train: 09817828 准备训练: 4384406 行数
[2017-10-30 04:37:28.953616] INFO: quant: stock_ranker_train.v5 运行完成[405.360067s].
[2017-10-30 04:37:28.960640] INFO: quant: instruments.v2 开始运行..
[2017-10-30 04:37:28.963965] INFO: quant: 命中缓存
[2017-10-30 04:37:28.965369] INFO: quant: instruments.v2 运行完成[0.004703s].
[2017-10-30 04:37:28.983046] INFO: quant: general_feature_extractor.v6 开始运行..
[2017-10-30 04:37:28.987251] INFO: quant: 命中缓存
[2017-10-30 04:37:28.988026] INFO: quant: general_feature_extractor.v6 运行完成[0.00499s].
[2017-10-30 04:37:28.993648] INFO: quant: derived_feature_extractor.v2 开始运行..
[2017-10-30 04:37:28.996227] INFO: quant: 命中缓存
[2017-10-30 04:37:28.997090] INFO: quant: derived_feature_extractor.v2 运行完成[0.00344s].
[2017-10-30 04:37:29.002208] INFO: quant: dropnan.v1 开始运行..
[2017-10-30 04:37:29.007026] INFO: quant: 命中缓存
[2017-10-30 04:37:29.008029] INFO: quant: dropnan.v1 运行完成[0.005807s].
[2017-10-30 04:37:29.024938] INFO: quant: stock_ranker_predict.v5 开始运行..
[2017-10-30 04:37:32.514483] INFO: df2bin: prepare data: prediction ..
[2017-10-30 04:38:02.464189] INFO: stock_ranker_predict: 准备预测: 1202058 行
[2017-10-30 04:38:15.803604] INFO: quant: stock_ranker_predict.v5 运行完成[46.778661s].
[2017-10-30 04:38:15.864959] INFO: quant: backtest.v7 开始运行..
[2017-10-30 04:39:11.183389] INFO: Performance: Simulated 488 trading days out of 488.
[2017-10-30 04:39:11.184538] INFO: Performance: first open: 2015-01-05 14:30:00+00:00
[2017-10-30 04:39:11.185306] INFO: Performance: last close: 2016-12-30 20:00:00+00:00
[2017-10-30 04:39:13.489930] INFO: bigquant: backtest.v7 运行完成[57.624996s].
```

## 回测结果数据

![20171029Capture1](https://raw.githubusercontent.com/zhengr/zhengr.github.io/master/assets/images/20171029Capture1.PNG)

