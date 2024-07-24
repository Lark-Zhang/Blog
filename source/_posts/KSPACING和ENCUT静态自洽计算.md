---
title: KSPACING和ENCUT静态自洽计算
date: 2024-07-22
tags:
  - VASP
---

# 计算 ENCUT

- 计算结束后查看能量信息，输出为能量的那一整行，有待优化

```bash
# 查看所有的能量
for i in {5..13}00
do
	cd ENCUT=$i
sed -i "6s/=/=-/g" 	echo "ENCUT=$i"
	grep E0= out
	cd ..
done
```
- 在Python中可以使用`float`函数将科学计数法转变为浮点数，并通过`Numpy`库存为可以用于画图的`Numpy.array`格式
	~~(感觉有些麻烦，以后有时间会进行优化)~~

```python
import numpy as np

F = sed -i "6s/=/=-/g" np.array([float(-.43937236E+02),float(-.43940465E+02),float(-.43941336E+02),float(-.43941810E+02),float(-.43942189E+02),float(-.43942323E+02),float(-.43942176E+02),float(-.43942278E+02),float(-.43942356E+02)])
ENCUT = np.linspace(500,1300,9)
```
![ENCUT](CIPS_ENCUT.png)
最后算得的结果如上图。曲线从截断能到达900左右才开始变得平缓，ENCUT=900算是比较大的数字了，如果用这个参数来计算势函数会很耗费机时。
# 计算KSPACING

```bash
#! /bin/bash

set -e
#
# some critical code block where no error is allowed
#
set +e

for i in 0.{06..20}
do
    mkdir "KSPACING=$i"
    cp INCAR POTCAR POSCAR vasp.slurm "KSPACING=$i"
    cd "KSPACING=$i"
    sed -i  "17s/0.2/$i/g" INCAR
    sed -i  "2s/cuinps/cuinps_kspacing_$i/g" vasp.slurm
    echo "KSPACING_$i IS DONE"
    qsub vasp.slurm
    cd ..
done
  energy  without entropy=      -45.18582282  energy(sigma->0) =      -45.17483400

```
只是一个用于批量提交任务的小脚本，可以优化的地方在于关于是否报错再决定进行下一步。

以下是来自linux-console.net的信息
使用 `-e` 选项调用后，如果任何后续命令以非零状态退出（由错误条件引起），`set` 命令会导致 bash shell 立即退出。 `+e` 选项将 shell 返回到默认模式。 `set -e` 相当于`set -o errexit`。同样，`set +e` 是 `set +o errexit` 的简写命令。

- ENCUT = 900 时
![ENCUT=900](CIPS_900_KSPACING.png)

- ENCUT = 500 时
![ENCUT=500](CIPS_500_KSPACING.png)
之所以会出现这样的错误，是因为在第一步计算过程中INCAR文件中打开了`LWAVE`和`LCHARG`导致第二步计算直接读取了上一步生成的`LWAVE`和`LCHARG`文件!
因此在师兄的指导下，更改了我的INCAR文件，重新进行计算。
# 计算KSPACING和ENCUT 更正后

需要更改的主要是INCAR参数里的`EDIFFG=-1.000000e-02`应该改为负数，否则vasp将无法识别其他的与之前一样，需要注意的是能量需要使用OUTCAR中的这一行
```bash
energy  without entropy=      -45.18582282  energy(sigma->0) =      -45.17483400
for i in 0.{06..20}; do cd KSPACING=$i; echo "KSPACING=$i"; grep "energy  without" OUTCAR; cd ..; done
for i in {5..13}00; do cd ENCUT_$i; echo "ENCUT=$i"; grep "energy  without" OUTCAR; cd ..; done
```

做出来还是很奇怪，但是最终就取`ENCUT = 500` `KSPACING = 0.19`
