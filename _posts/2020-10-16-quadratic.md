---
layout: post
title: "[论文阅读]Quadratic Video Interpolation"
subtitle: ""
date: 2020-10-16
author: Aether
category: coding
tags: CODING PAPER CV VIDEO-FRAME-INTERPOLATION
finished: true
---

# Quadratic Video Interpolation

###### Li, Siyao, et al.  - In *ICCVW*, 2019



论文使⽤⾼阶模型对光流建模，可反映物体运动的加速度和⾮线性轨迹，打破了传统光流方法使用的线形模型的限制，可以对现实世界中物体的复杂运动进⾏了更为准确的描述。

### Quadratic flow prediction

传统方法中默认物体匀速运动，加速度为 0，$\boldsymbol f_{0 \rightarrow t}$ 可由 $\boldsymbol f_{0 \rightarrow 1}$ 线形表示


$$
\boldsymbol f_{0 \rightarrow t} = t\boldsymbol f_{0 \rightarrow 1}
$$


论文中利用多个相邻帧提取高阶信息，建立加速度模型，将 $\boldsymbol f_{0 \rightarrow t}$ 表示为


$$
\boldsymbol f_{0 \rightarrow t} = (\boldsymbol f_{0 \rightarrow 1}+\boldsymbol f_{0 \rightarrow -1})/2 \times t^2 + (\boldsymbol f_{0 \rightarrow 1}-\boldsymbol f_{0 \rightarrow -1})/2 \times t
$$



即


$$
x=v_0t+\frac{1}{2}at^2
$$


线性模型限制像素匀速&沿直线运动，只用最近的两帧信息。此公式引入加速度和曲线运动，能够利用更多的临近帧，反映物体运动的加速度和⾮线性轨迹。

### Flow reversal layer

由于线性组合 $\boldsymbol f_{0 \rightarrow 1}$ 和 $\boldsymbol f_{1 \rightarrow 0}$ 来表示 backward flow 的方法具有运动边界周围表现不佳、无法应用加速度的缺点，作者提出新的计算方法：


$$
\boldsymbol f_{t \rightarrow 0}(\boldsymbol u)=
	\frac{\sum_{\boldsymbol x + \boldsymbol f_{0 \rightarrow t}(\boldsymbol x)\in \mathcal N(\boldsymbol u)}w(||\boldsymbol x + \boldsymbol f_{0 \rightarrow t}(\boldsymbol x)-\boldsymbol u||_2)
(-\boldsymbol f_{0 \rightarrow t}(\boldsymbol x))}{\sum_{\boldsymbol x + \boldsymbol f_{0 \rightarrow t}(\boldsymbol x)\in \mathcal N(\boldsymbol u)}w(||\boldsymbol x + \boldsymbol f_{0 \rightarrow t}(\boldsymbol x)-\boldsymbol u||_2)}
$$



- $w(d)=e^{-d^2/\sigma ^2}$  每个光流的高斯权重
- $\mathcal N(\boldsymbol u)$ ：像素 $\boldsymbol u$ 的周边像素

由于此计算公式可微，所以求梯度反向传播到光流估计部分可以作为一个模块加入到系统中，实现端到端训练；同时此方法可能导致空洞（物体在 t 时可见，0 时被遮挡，将被填充为变形的 1 时图像）

### Frame synthesis

#### Adaptive flow filtering

由于使用 PWC-Net，backward flow 的边缘可能出现 ringing artifacts。作者使用 resCNN 策略表现不佳，它们主要来源于尖锐边缘的离群值，无法简单地移除这些立群值，因为卷积的权重平均运算受其影响。为避免权重平均的影响，只从相邻的一个像素采样


$$
\boldsymbol{f}_{t \rightarrow 0}^{\prime}(\boldsymbol{u})=\boldsymbol{f}_{t \rightarrow 0}(\boldsymbol{u}+\boldsymbol{\delta}(\boldsymbol{u}))+\boldsymbol{r}(\boldsymbol{u}) \\
$$



- $\boldsymbol \delta (\boldsymbol u) = k\times \tanh(·)∈[−k,k]$

此滤波器使backward flow可以空间变形且非线性微调

#### Warping and fusing source frames

基于$I_0$ $ I_1$ $ \boldsymbol f_{t \rightarrow 0}$ $\boldsymbol f_{t \rightarrow 1}$，合成目标中间帧 $\hat I_t$ ：


$$
\hat{I}_{t}(\boldsymbol{u})=\frac{(1-t) m(\boldsymbol{u}) I_{0}\left(\boldsymbol{u}+\boldsymbol{f}_{t \rightarrow 0}^{\prime}(\boldsymbol{u})\right)+t(1-m(\boldsymbol{u})) I_{1}\left(\boldsymbol{u}+\boldsymbol{f}_{t \rightarrow 1}^{\prime}(\boldsymbol{u})\right)}{(1-t) m(\boldsymbol{u})+t(1-m(\boldsymbol{u}))} \\
$$



- 图片合成中不直接使用$I_{-1}$，$I_2$，这两帧只用于挖掘加速度信息

定义 loss 为 1 loss 和 perceptual loss 的结合：


$$
\left\|\hat{I}_{t}-I_{t}\right\|_{1}+\lambda\left\|\phi\left(\hat{I}_{t}\right)-\phi\left(I_{t}\right)\right\|_{2}
$$



- $\phi$ 是 VGG16 模型的特征提取器

### Algorithm Frame Overview

![image-20201021033212451](../img/image-20201021033212451.png)

1. **光流估计**：输入 $I_0$ $I_1$，使用 PWC-Net 计算 $\boldsymbol f_{0 \rightarrow 1}$ $\boldsymbol f_{0 \rightarrow -1}$
2. **平方光流预测**：输入 $\boldsymbol f_{0 \rightarrow 1}$ $\boldsymbol f_{0 \rightarrow -1}$, 计算 $\boldsymbol f_{0 \rightarrow t}$ 
3. **逆转光流**：输入 $\boldsymbol f_{0 \rightarrow t}$ ，计算反向光流 $\boldsymbol f_{t \rightarrow 0}$ 
4. **合成中间帧**：输入$I_0$ $ I_1$ $ \boldsymbol f_{t \rightarrow 0}$ $\boldsymbol f_{t \rightarrow 1}$，合成目标中间帧 $\hat I_t$ 

### Experiments 

- Data
  - 数据集：GOPRO、Adobe240、UCF101、DAVIS。视频既包含镜头抖动，也包含物体的动态运动
  - 数据增强：改变大小、随机裁剪、翻转图片、倒放、960fps降采样为240fps和480fps
- Hyper-parameters
  - iteration: 200+40 epochs
  - optimizer: Adam
  - initial learning rate $\alpha = 10^{-4}$
  - $\lambda = 0.005$
  - $k = 10$
  - $\sigma = 1$ 
- Metricss
  - PSNR、SSIM、IE（RMS）
  - ASFP  average shift of feature points