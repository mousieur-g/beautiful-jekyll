---
layout: post
title: Logistic模型识别信用卡欺诈
subtitle: Python学习笔记
bigimg: /pics/20180527/01.jpg
tags: Python，Logistic
---

## 数据
 
&emsp;&emsp;该信用卡欺诈数据来源于[天善智能学院](https://edu.hellobi.com/course/144/lessons)，包含284807条样本数据，涉及30个特征变量（*Time*, *V1* ~ *V28*, *Amount*）及1个行为类标签（*Class*），部分截图如下，其中*V6* ~ *V24*已经被隐藏。
 
![](/pics/20180527/02.png)

## 预处理

&emsp;&emsp;这里不考虑缺失和异常，因为数据良好，并且已经做了脱敏处理。相关的预处理操作见如下代码（原来`pd.to_excel`只能向*xls*文件中写入65536行）。
 
![prepareAnalysis.py](/pics/20180527/03.png)

## 自定义函数

&emsp;&emsp;这里自定义两个函数，一个是用于训练*Logistic*模型时选取最佳*L1*惩罚系数的函数，另一个是绘制混淆矩阵的函数，具体见如下代码（😂Python代码果然不是那么好写的）。

![functions.py](/pics/20180527/04.png)

## LR模型

&emsp;&emsp;基于预处理的数据，直接进行LR模型的训练，见代码。

![originSampleAnalysis.py](/pics/20180527/05.png)

## 处理不平衡数据的问题

&emsp;&emsp;[欠采样](https://en.wikipedia.org/wiki/Undersampling)和[过采样](https://en.wikipedia.org/wiki/Oversampling)是两种不同的用于处理非平衡数据的方法。这里的过采样，选用*SMOTE*算法生成新的样本数据。

### 欠采样

![underSampleAnalysis.py](/pics/20180527/06.png)

#### 过采样

![overSampleAnalysis.py](/pics/20180527/07.png)
 