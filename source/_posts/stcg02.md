---
title: Stanford CS148 Course02 Light and Color
date: 2020-02-18
tag: [Computer Graphics]
categories: [Stanford CS148]
---

### 电磁波谱 (Electromagnetic Spectrum)

依照波长的长短、频率以及波源的不同，电磁波谱可大致分为：无线电波、微波、红外线、可见光、紫外线、x射线和伽马射线。
![](/images/stcg/electro.png)

波长($\lambda $)和能量($\Delta E$)的关系：$\lambda \Delta E = 1239.9$

### 光谱 (Spectrum)
光谱可以看成是光子数目和波长的关系，又因为同样波长的光子，数目越多能量越大，所以也可以看成是波长和能量的关系。
![](/images/stcg/dayavgspectrum.jpg)

然而基于上述关系构建的光谱渲染器(spectral renderer)代价太过高昂，因此考虑到使用红绿蓝的组合来模拟光(暂不考虑光的偏振，衍射和干涉等)。


### 人眼 (Human eye)

人的视网膜(retina)上的光度感应器(photodetector)是`视杆细胞(rods)`和`视锥细胞(cones)`。

![](/images/stcg/retina.jpeg)

`视锥细胞(cones)`在视网膜中央凹处(`fovea`)密度最大，而此处几乎没有`视杆细胞(rods)`。`视杆细胞(rods)`和`视锥细胞(cones)`的分布如下图所示。
![](/images/stcg/rc.jpeg)

视杆细胞是感受弱光刺激的细胞，对光线的强弱反应非常敏感，主要在弱光下发挥作用，不能辨别颜色。比如，猫头鹰视网膜中视杆细胞较多，故夜间活动视觉灵敏。视锥细胞则可以在正常明度下，为我们提供视觉信息。

实际上在视网膜上存在着三种用来感知颜色的视锥细胞，他们分别叫做`S(Short)-视锥细胞`，`M(Middle)-视锥细胞`，`L(Long)-视锥细胞`，对应着波长的短，中，长。

每种视锥细胞都对某一个特定波长的颜色异常敏感。一般来说，`S-视锥细胞`对波长`420nm`的光线最为敏感，`M-视锥细胞`对`530nm`的波长最为敏感，而`L-视锥细胞`对于`560nm`的波长最为敏感。

![](/images/stcg/conesrespond.jpg)



### 颜色 (Color)

#### 色度学 (Colorimetry)


#### 格拉斯曼颜色定律 (Grassmann’s laws)

- Symmetry law
- Transitive law
- Proportionality law
- Additivity law


#### 配色实验 (Color Matching Experiments)
> 使用该实验确定的假想观察者，可将任意的可见光谱功率分布转换为一组三刺激值，从而量化地描述人类色觉。

简单来说，就是认为调整三种不同波长的单色光(波长为分别为435.8, 546.1, 700 nm)的组合，使其和另一种颜色匹配，然后记录下这个波段的光对应的三原色的系数。得到下面的曲线。

![](/images/stcg/matchexp.jpg)



#### 色彩空间 (Color Space)

![](/images/stcg/Colorspace.png)



![RGB色彩空间](/images/stcg/rgbcs.png)

由于每个显示设备有自己的原色(primaries)，所以即使有一张图片在RGB空间中定义，在不同设备上显示也可能不同。为此需要RGB空间的转换。


![TransformationsforstandardRGBcolorspaces](/images/stcg/trans.jpg)


### Light



### 参考资料

1. [Stanford CS148 Introduction to Computer Graphics and Imaging (Fall 2019)](http://web.stanford.edu/class/cs148/lectures.html)

2. Fundamentals of Computer Graphics, 4th Edition. 
Steve Marschner and Peter Shirley, A K Peters/CRC Press, 2015. (Ch21, Ch22, Ch23)

3. OpenGL Programming Guide: The Official Guide to Learning OpenGL, Version 4.3 (8th Edition) Dave Shreiner, Addison-Wesley Professional, 2013. (Ch7)

4. [Introduction to Light, Color and Color Space](https://www.scratchapixel.com/lessons/digital-imaging/colors/introduction)