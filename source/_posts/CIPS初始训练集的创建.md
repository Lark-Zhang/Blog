---
title: CIPS初始训练集的创建
date: 2024-07-23
tags:
  - VASP
---

## 1. 准备工作

准备一组数据需要已经优化好的电子结构`COUNTCAR` 、赝势`POTCAR`以及输入参数 `INCAR`，并将`COUNTCAR`更名为`POSCAR`以便进行下一步运算。
### POSCAR文件的准备
为了让初始数据集的数据足够丰富，需要对体系进行一些应变操作。对于我要运算的CIPS二维体系，需要对 (001) 晶面进行拉伸而不对(001)方向进行任何操作，此时需要通过一个脚本来实现。~~使用MS进行操作然后导出文件也可以，不过太麻烦orz~~

但是众所周知的是，bash有一个很奇葩的特性，就是没办法直接对数字进行数学运算，因为默认的变量都会被识别为字符串。此时将引入了`bc`命令

"|" 是bash所谓的管道符，其意义是连接命令，命令 1 的正确输出作为命令 2 的操作对象。

**举例**
```bash
$ echo "15+5" | bc
20

# 可以通过scale设定小数位
$ echo 'scale=2; (2.777 - 1.4744)/1' | bc
1.30
```

了解了`bc`以后，思路就很简单了。我的目录下有一个POSCAR文件，我希望能够通过这个脚本来生成10个应变以后的POSCAR并放在10个不同的文件夹里

==一定要记住声明变量的赋值语句等号前往不能加空格，一个空格的报错害得我找了半天！！！==

strain代码感觉不是很简洁，不晓得bash的列表好用不，要不可以适当优化一下
```bash
#!/bin/bash 

# 首先读取POSCAR文件中的3、4行数据，即a轴和b轴

a1=$(sed -n '3p' POSCAR | awk -F' *' '{print $2}')
a2=$(sed -n '3p' POSCAR | awk -F' *' '{print $3}')
a3=$(sed -n '3p' POSCAR | awk -F' *' '{print $4}')
a4=$(sed -n '4p' POSCAR | awk -F' *' '{print $2}')
a5=$(sed -n '4p' POSCAR | awk -F' *' '{print $3}')
a6=$(sed -n '4p' POSCAR | awk -F' *' '{print $4}')

# 分别将变形以后的
for i in {-5..5}
do
	mkdir "layer_strain_$i%"
	cp POSCAR "layer_strain_$i%"
	cd "layer_strain_$i%"
	n=$(echo "100+$i" |bc)
	rat=$(echo "scale=2; $n/100" |bc)
	# 3-1
	new_a1=$(echo "scale=16; $a1*$rat" |bc)
	sed -i "3s/$a1/$new_a1/g" POSCAR
	# 3-2
	new_a2=$(echo "scale=16; $a2*$rat" |bc)
	sed -i "3s/$a2/$new_a2/g" POSCAR
	# 3-3
	new_a3=$(echo "scale=16; $a3*$rat" |bc)
	sed -i "3s/$a3/$new_a3/g" POSCAR
	# 4-1
	new_a4=$(echo "scale=16; $a4*$rat" |bc)
	sed -i "4s/$a4/$new_a4/g" POSCAR
	# 4-2
	new_a5=$(echo "scale=16; $a5*$rat" |bc)
	sed -i "4s/$a5/$new_a5/g" POSCAR
	# 4-3
	new_a6=$(echo "scale=16; $a6*$rat" |bc)
	sed -i "4s/$a6/$new_a6/g" POSCAR
	cd ..
done
```

由此所有的11个POSCAR文件就准备好了
这个没有对齐，感觉有点丑...好在不影响使用?
![[POSCAR_strain.png]]
在社区逛了一圈，发现这个小数点前没有0之类的是`bc`输出的问题，好在可以加`awk`来解决
如下：
```bash
new_a6=$(echo "scale=16; $a6*$rat" |bc| awk '{printf "%.2f", $0}')
```

### INCAR文件的准备

这里进行的是分子动力学模拟，因此将BRION设置为0[[INCAR参数详解#^73e8ee]]

```bash 
IBRION = 0
```

设置不同的温度进行计算
```bash
TEBUG = 50 # /100
```

对于双层材料需要考虑到Van der Waals 修正
```bash
IVDW = 12
```
双层的结构除了点问题就是...之前的滑移结构没有算完，但是师兄的意思是把滑移的结构都放进去算。
