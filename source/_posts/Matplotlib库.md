---
title: Matplotlib库的基本使用
date: 2024-07-22
aliases:
  - Python 绘图
tags:
  - Python
---
# 2.1 基本用法

```python
import matplotlib.pyplot as plt
import numpy as np

x = np.linspace(-1,1,50)
y = 2*x+1
plt.plot(x,y)
plt.show()
```
# 2.2 figure图像

- 绘制两张figure
```python
x = np.linspace(-1,1,50)
y1 = 2*x+1
y2 = x**2

plt.figure(num=1)
plt.plot(x,y1)

plt.figure(num=2,figsize=(4,4))
plt.plot(x,y2)

plt.show()
```

- 绘制两个曲线在一张figure中
```python
x = np.linspace(-1,1,50)
y1 = 2*x+1
y2 = x**2

plt.figure(num=1,figsize=(8,5))
plt.plot(x,y1)
plt.plot(x,y2，color='red',linewidth=1.0,linestyle='--')

plt.show()
```
# 2.3设置坐标轴
```python
# 设置取值范围
plt.xlim（（-1,1））
plt.ylim（（-2,3））

# 设置lable
plt.xlabel("X")
plt.ylable("Y")

# ticks = np.linspace
```
