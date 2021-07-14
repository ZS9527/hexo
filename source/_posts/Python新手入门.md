---
title: Python新手入门
date: 2021-01-30 10:31:28
tags: Python
---

# Python新手入门

> 业务需要Python画图，大佬已经写好了画图程序。我这边需要接入到业务中。
>
> 还是需要学习如何在本地运行起来。

## 本地环境安装

大佬给了我几个链接和程序，我照着开始研究。

### **【Anaconda/Miniconda 管理不同版本 Python 环境/依赖安装**】

   [**https://docs.conda.io/en/latest/**](https://docs.conda.io/en/latest/)

   [**https://blog.csdn.net/qq_18668137/article/details/80807829**](https://blog.csdn.net/qq_18668137/article/details/80807829)

1. **搜索依赖包**

   https://anaconda.org/search?q=pandas

2. **上海交通大学镜像**

   https://mirrors.sjtug.sjtu.edu.cn/docs/anaconda

3. **conda install卡在solving environment解决办法**

   https://www.codeleading.com/article/39333157784/

4. **在 Windows 安装虚拟环境，安装依赖后使用如果报错，需要注意环境变量的设置**

   https://www.cnblogs.com/maomaozi/p/14619961.html

### **【准备工作】**

   **# Install**

   下载 Anaconda/Miniconda 进行安装（推荐 Miniconda）

   **# Get Start**

   1. 创建 Python 3.9 环境

   conda create -n py39 -c conda-forge python=3.9

   2. 安装相关依赖（可以按需安装）

   conda install -c conda-forge -n py39 autopep8 -y

   // cartopy numpy matplotlib xarray pygrib cmaps netcdf4

   3. 切换到刚刚安装的环境

   conda activate py39

### **【Python】**

### **# 基础教程**

https://www.runoob.com/python3/python3-tutorial.html

- 遍历数组 https://www.cnblogs.com/pizitai/p/6398276.html

### **【Unidata】**

**# Python 气象绘图示例**

https://unidata.github.io/python-gallery/examples

## 安装后，对实例代码进行启动
通过上面大佬的指引，我在本地装好了 Anaconda。又安装了 Pycharm。
上面的准备工作中，又创建了除了 base 环境以外的 Python3.9 环境。
本来是打算用 spyder, 但是同事推荐 Pycharm。
所以需要在 Pycharm 中修改要使用的 Python 版本。
### 修改Pycharm环境
在 Pycharm 中找到 File -> setting -> Project: （项目所在文件夹）-> Python Interpreter
![b1.png](b1.png)
我截图的是已经添加了一个的情况，点击右上角齿轮，点击 add 。
![b2.png](b2.png)
之后选择 Conda Environment -> Existing environment 。在Interpreter 中选择要使用的环境的python.exe，完成环境添加。

### 添加所需包
之后运行 py 文件，报了个 ModuleNotFoundError: No module named 'cartopy' 。
双击 Anaconda Prompt ，可以从前面的（）中看出现在运行的环境。
可以使用 `conda env list`命令来查看conda 中装了几个环境，以及他们的名字是什么，所在本地目录。
使用`conda activate py39` 来切换运行环境。
再使用`conda install -c conda-forge cartopy` 命令将依赖包安装好。
虽然 pygrib 这个包安装有点慢，但是挂了一夜后还是装好了。

## 遇到的问题
1. 遇到了一个奇怪的bug， `Unable to find boot.def`
```

ECCODES ERROR   :  Unable to find boot.def. Context path=D:/bld/eccodes_1623941334791/_h_env/Library/share/eccodes/definitions

Possible causes:
- The software is not correctly installed
- The environment variable ECCODES_DEFINITION_PATH is defined but incorrect

ecCodes assertion failed: `0' in D:\bld\eccodes_1623941334791\work\src\grib_context.c:226
```
百度后尝试在 anaconda prompt 中运行命令，还是不行。
```
set "ECCODES_SAMPLES_PATH=d:\anaconda3\Library\share\eccodes\samples"
set "ECCODES_DEFINITION_PATH=c:\ProgramData\Anaconda3\Library\share\eccodes\definitions"
```
然后我将这两个设置为电脑的环境变量，就可以了

2. `check_hostname requires server_hostname` 这个错误是因为我本地开启了代理。
