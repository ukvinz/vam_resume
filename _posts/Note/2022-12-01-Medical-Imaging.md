---
layout: "post"
author: Lin Han
title: "医学影像笔记"
date: "2022-02-12 02:54"
math: true
published: false
categories:
  - Note
  - "Medical Imaging"
tags:
  - Note
  - "Medical Imaging"
---

## MRI

- 质子数或中子数为奇数，原子核有自旋，都是偶数没有

- 不严谨的理解上，一个带电的物质旋转就有磁矩，方向和旋转方向垂直。没有环境影像方向随机，变化随机。大量放在一起[互相抵消](https://tools.itali.uq.edu.au/bioimg/nuclear-spin/nuclear-spin.html)整体没有磁性

- 施加一个磁场，核自旋倾向于和外部磁场方向对齐。
  
  - 1/2自旋系统的磁矩在和外部磁场呈54或者126度夹角旋进 precessing，围绕外部磁场旋转，54度能量更低
  
  - 旋转的频率是Larmor frequency
    
    - Larmor equation: $\omega_0=\gamma B_0$，MR里一般是MHz
      
      - $\gamma$：gyromagnetic ratio，是原子核的性质
      
      - $B_0$：外加磁场强度

### <video src="https://user-images.githubusercontent.com/29757093/205147188-a3716fc9-16bb-41f4-a767-c152042ff8c1.mp4"></video>

- MR检测到的信号是在横断面旋进的净磁化向量转动时产生的电场，信号的频率是Larmor freq。

- dephasing：不同的$B_0$带来不同的Larmor freq，旋进的角速度不同磁矩逐渐互相抵消

- Free induction decay:
  
  - Lab frame：振幅逐渐变小，频率$\frac{1}{\delta}$，$\delta$应该是Larmor和RF频率的差
    
    - <video src="https://user-images.githubusercontent.com/29757093/205153585-ec296329-49e4-4b34-90a2-246719b2a8e3.mp4"></video>
  
  - Rotating frame：强度指数变小
    
    - <video src="https://user-images.githubusercontent.com/29757093/205154065-582611d2-ceee-44ed-95da-a598534ef918.mp4"></video>



$T_{1}\ge T_2$



[MR Physics 2 - The Source of Signal - YouTube](https://www.youtube.com/watch?v=4vEfWBkju6w&list=PLBkfUPj1TSRqmh2pXV2t5bY5Qr3Z8AeOi&index=2) 

[Introduction to Biomedical Imaging | edX](https://learning.edx.org/course/course-v1:UQx+BIOIMG101x+1T2022/home)
