---
title: 气象相关MeteoInfo失败使用过程记录
date: 2020-05-09 18:21:01
tags: 气象
---

# 气象相关MeteoInfo失败使用过程记录
> 虽然成功画图，但是依旧是不能够有足够的参数调整。图片一看就是有问题的图，更换文件类型去尝试，未能成功画图。

<!--more-->

## 初步安装
meteoInfo的安装没有多麻烦。直接百度[MeteoInfo](http://meteothink.org/downloads/index.html) 找到压缩包，下载后解压，启动MeteoInfoLab.exe文件即可。

## 明确目的
主要目的是为了nc类型文件，micap4类型文件等值线图的绘制。

## 尝试过程
一开始是想要寻找现成的脚本运行，然后发现都是有点专业和高级。百度后最快找到的教程是[MeteoInfoLab脚本汇总贴](MeteoInfoLab)，在里面寻找到关键字相符的文章后，开始每个语句去试，不知是不是语句太简单的缘故，博主并没有注释。在官网中找到的文档404了。

### 第一个文件
尝试了一下[MeteoInfoLab脚本示例：创建netCDF文件（合并文件）](http://bbs.06climate.com/forum.php?mod=viewthread&tid=38489&extra=page%3D1) 
并不能成功的按照步骤来画图，我自己的nc文件并没找到方法读出lat和lon。再次仔细翻看帖子，并没有找到类似的文件解析。最终只好逐渐的一个语句一个语句的去尝试，结果如下：
```
f = addfile('C:/Users/Administrator/Desktop/nc.nc')
var = f['CO'][:]
contourf(var)
print var
```
这个语句可以出图，但是没有边界线。也和之前的图片不同。

### 第二个文件
从一个帖子中得知，解压出的sample文件夹有事例文件。取出后用同样的语句进行尝试，这次却没能够重新画图，直接报错。并没有找到原因。
```
"D:\Downloads\Compressed\MeteoInfo_2.2.1\MeteoInfo\pylib\mipylib\plotlib\miplot.py", line 2180, in contour
    r = gca.contour(*args, **kwargs)
  File "D:\Downloads\Compressed\MeteoInfo_2.2.1\MeteoInfo\pylib\mipylib\plotlib\_axes.py", line 2048, in contour
    gdata = np.asgriddata(args[0])
  File "D:\Downloads\Compressed\MeteoInfo_2.2.1\MeteoInfo\pylib\mipylib\numeric\core\numeric.py", line 2349, in asgriddata
    return data.asgriddata()
  File "D:\Downloads\Compressed\MeteoInfo_2.2.1\MeteoInfo\pylib\mipylib\numeric\core\dimarray.py", line 667, in asgriddata
    gdata = GridData(self._array, xdata, ydata, self.fill_value, self.proj)
	at org.meteoinfo.data.GridData.<init>(GridData.java:149)

	at sun.reflect.NativeConstructorAccessorImpl.newInstance0(Native Method)

	at sun.reflect.NativeConstructorAccessorImpl.newInstance(Unknown Source)

	at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(Unknown Source)

	at java.lang.reflect.Constructor.newInstance(Unknown Source)

	at org.python.core.PyReflectedConstructor.constructProxy(PyReflectedConstructor.java:213)

java.lang.ArrayIndexOutOfBoundsException: java.lang.ArrayIndexOutOfBoundsException: 12
```

### 第三个文件
这次是尝试使用MICAP4类型文件，根据[MeteoInfoLab脚本示例：micaps4类数据](http://bbs.06climate.com/forum.php?mod=viewthread&tid=45105&extra=page%3D1)画出的图片十分的奇怪，直接就可以看出来是个失败的了。
```
f = addfile_micaps('C:\\Users\\Administrator\\Desktop\\20041520.000')
t1 = f.gettime(0)
data = f['var'][0,0,:,:]
axesm()
geoshow('cn_province')
geoshow('country', edgecolor='k')
levs = [0.1,2,5,10,20,30,50,80,100,120,150]
cols = makecolors(len(levs)+1, cmap='MPL_rainbow')
cols[0] = 'w'
layer = imshowm(data, levs, colors=cols)
colorbar(layer, label='mm', extendrect=False)
```

## 结论
由于时间，以及资料太少了的原因。画图暂停继续研究，试图从客户那边直接要一个程序，我们直接采集图片。

专业的知识还是比较难以搜索，从百度中查找关键词MICAP4只能查到一些莫名其妙的东西。google查meteoInfo也没查出太多有用的东西。这两个关键字大都是指向了气象家园这个论坛。ncl的关键词查询稍稍多了一些，关联到了PyNGL这个技术。然后又牵扯到了python，没用的网页瞬间多了不少呢。

看来专业的知识还是需要花功夫去查一查书籍。直接找脚本还是有些困难的。