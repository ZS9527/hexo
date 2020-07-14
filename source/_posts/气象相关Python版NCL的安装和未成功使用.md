---
title: 气象相关Python版NCL的安装和未成功使用
date: 2020-04-30 18:23:26
tags: 气象
---

# 气象相关Python版NCL的安装和未成功使用
> 业务需要将气象服务文件类型画图，去找了找有关的东西，发现了这个视频简单一些[Python版NCL安装及快速入门](https://www.bilibili.com/video/av73159125/) 。仔细研究了一下发现还是没能入门。但是还是记录一下，之后看看有没有后续研究。
> 这里卡住了之后要去尝试看一下 MeteoInfoLab，暂时停止这个的研究。

<!--more-->

## 环境安装
首先需要一台 linux 机器，本人用的是腾讯云。
### 1. 安装miniconda
百度 minicaonda，打开官网。[https://docs.conda.io/en/latest/miniconda.html](https://docs.conda.io/en/latest/miniconda.html) 找到相对应的安装包。视频中下载的是32位，因为云服务器系统的原因，我使用64位。下载速度有点慢，所以我从本机下载完成后传到准备好的 linux 文件夹里。
![1](b1.png)

下载完成后，给安装包赋予执行权限
```
chmod 777 Miniconda3-latest-Linux-x86_64.sh
```

执行安装程序
```
bash Miniconda3-latest-Linux-x86_64.sh
```
安装时需要同意协议，接着按回车以安装在默认位置
![2](b2.png)

![3](b3.png)

输入yes同意安装包初始化 Miniconda。
![4](b4.png)

到这里 Miniconda 安装完毕，请注意输出的提示信息 ..bashsrc 被更改。防止一些莫名其妙的错误，重启一下终端。
![5](b5.png)

### 2. 使用conda
重启之后我并没有像视频中那样找不到 conda，貌似是直接默认安装到了环境变量中。
![6](b6.png)

当命令提示符前面出现“（base）”字样时，我们就已经进入了 conda。使用`conda list`命令可以查看 conda 的全局环境内装了什么包
![7](b7.png)

输入`python --version`查看一下版本。输入命令来创建一个名为 NCL-py 的 conda 空间并且安装所需的 `conda-forge` 、`xarray`、 ` netcdf4` 、`scipy`、 `pyngl` 、`pynio` 、`ncl` 包，这里有个失误，视频里是`_`，我打错成了`-`。不过一个命名没啥影响。安装的中间还是需要同意一下。
```
conda create -n NCL-py -c conda-forge xarray netcdf4 scipy pyngl pynio ncl
```
![8](b8.png)

安装完毕后，输入 conda activate NCL-py 进入我们刚才创建的环境。
接下来就要写代码了。
![9](b9.png)

## 代码编写
这里是卡住我的原因，视频中所提的NC文件我这里并没有。而且我去下载了 Panoply 这个程序后，发现解析出来的nc内容也不一样。

忽略UP主中间的错误尝试，我最后的代码是这样的
```
"""py-NCL画图"""
import numpy as np
import Ngl, Nio
import netCDF4
f = Nio.open_file('/root/python3/WRFCHEM_2019110720_CO_f001.nc')
uwnd = f.variables['CO'][:]

wks = Ngl.open_wks('png', 'NCL-py')#这里我并没有XMind，直接就选择导出png图片

res = Ngl.Resources()

res.cnFillOn = True
res.cnLinesOn = False
res.cnLineLabelsOn = False
res.lbOrientation = 'horizontal'

plot = Ngl.contour(wks, uwnd, res) #原本是画的全球图片，现在只是一个区域
# dataset = netCDF4.Dataset('/root/python3/WRFCHEM_2019110720_CO_f001.nc')
# print(dataset.variables.keys())

Ngl.end()
```

### 缺失的条件
1. 首先并没有找到nc文件中的经纬度，没有去设置。
2. 对比 Panoply 程序画图后发现上面的程序虽然可以画，但是画出来的图片太粗糙了。而且位置也很奇怪。我不清楚那个画出来的图片是对的。

因为工期原因，暂时放弃这种方式解析。

