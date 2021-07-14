---
title: Python绘制高度场和风场
date: 2021-03-06 18:35:37
tags: Python
---

# Python绘制高度场和风场

> 业务需要，需要我们用数据来画形势场图。

## 参考文章

之前没看过Python绘制图片，查了很多专业文章。

1. 基础语法。

   [https://www.runoob.com/python/file-methods.html](https://www.runoob.com/python/file-methods.html)

2. Cartopy绘图，使用其中的设置投影，读取自定义shp文件。
   [http://www.meteoai.cn/post/%E6%8D%8D%E5%8D%AB%E7%A5%96%E5%9B%BD%E9%A2%86%E5%9C%9F%E4%BB%8E%E6%AF%8F%E4%B8%80%E5%BC%A0%E5%9C%B0%E5%9B%BE%E5%BC%80%E5%A7%8B/](http://www.meteoai.cn/post/%E6%8D%8D%E5%8D%AB%E7%A5%96%E5%9B%BD%E9%A2%86%E5%9C%9F%E4%BB%8E%E6%AF%8F%E4%B8%80%E5%BC%A0%E5%9C%B0%E5%9B%BE%E5%BC%80%E5%A7%8B/)

3. 高空天气图绘制，使用其中的风场图绘制方法。
   [https://mp.weixin.qq.com/s?src=11&timestamp=1624939451&ver=3159&signature=IRheGU9yvwuDA-4XS-NSe5TGXMIY-EMplUfDDakGER1AObyIs70jFyHOY87B0jrVnwLhCQqayiB4o1-tTTfnY8TiNgFVgCQa*NAzEiAUwXDl7JXDiFHQJgHMsCK7WxwQ&new=1](https://mp.weixin.qq.com/s?src=11&timestamp=1624939451&ver=3159&signature=IRheGU9yvwuDA-4XS-NSe5TGXMIY-EMplUfDDakGER1AObyIs70jFyHOY87B0jrVnwLhCQqayiB4o1-tTTfnY8TiNgFVgCQa*NAzEiAUwXDl7JXDiFHQJgHMsCK7WxwQ&new=1)
   
4. 气象绘图合集，从中了解一下专业术语和基础知识
   [https://mp.weixin.qq.com/s?src=11&timestamp=1625024794&ver=3161&signature=fHXwoUxOQIKXr5glWFcneH4I-MfVtJvdVK4lL7OsULP-gdbyeDzzLspQ27G*xqYIcD9qJ6Drxz61gl8vz0VSadHwB9FOFnZ9Njkqq3okYa2kjY6wUpPFjMH8IWjl8yOt&new=1](https://mp.weixin.qq.com/s?src=11&timestamp=1625024794&ver=3161&signature=fHXwoUxOQIKXr5glWFcneH4I-MfVtJvdVK4lL7OsULP-gdbyeDzzLspQ27G*xqYIcD9qJ6Drxz61gl8vz0VSadHwB9FOFnZ9Njkqq3okYa2kjY6wUpPFjMH8IWjl8yOt&new=1)
   
5. PyMICAPS，开源项目。使用其中的读MICAPS第4类文件和第11类风场流线图文件的方法
   [https://github.com/flashlxy/PyMICAPS/edit/master/Micaps4Data.py](https://github.com/flashlxy/PyMICAPS/edit/master/Micaps4Data.py)
   
6. 添加地图要素，使用这个来给白色底图增加一些颜色
   [https://mp.weixin.qq.com/s?__biz=MzIxODQxODQ4NQ==&mid=2247484117&idx=1&sn=929eea3b6a422333e7b72a8aa8db5294&chksm=97eb9e8fa09c1799dfed2ad4e4e7ae28f678cc1b48f9447598a29c83058d2c9643f44a152c21&scene=21#wechat_redirect](https://mp.weixin.qq.com/s?__biz=MzIxODQxODQ4NQ==&mid=2247484117&idx=1&sn=929eea3b6a422333e7b72a8aa8db5294&chksm=97eb9e8fa09c1799dfed2ad4e4e7ae28f678cc1b48f9447598a29c83058d2c9643f44a152c21&scene=21#wechat_redirect)
   
7. 添加点描述，不过我自己使用时候，还是有些问题。因为格点过于密集的缘故，加上文字后会和风向标重合。
   [https://zhuanlan.zhihu.com/p/107077358](https://zhuanlan.zhihu.com/p/107077358)

8. python一键将白色背景变为透明背景。因为前端需要叠加自己的地图，所以需要将背景转换为透明
   [https://www.cnblogs.com/aminor/p/13914684.html](https://www.cnblogs.com/aminor/p/13914684.html)

同时也参考了同事的绘制折线图demo，来完成中心气压的标志，等高线标注，java调用py文件这个三个功能。之后又加入了去掉白边，白色背景透明功能。

## 代码
代码中使用的数据有micaps4高度场数据，micaps11风场数据。
打开正确的注解后，就可以画出有底图的图片。

```python
# -*- coding: utf-8 -*-
"""
Created on Tue Jun 29 16:38:58 2021

@author: ezio
"""

import codecs
import numpy as np
import re
import math
import sys

from PIL import Image

import cartopy.crs as ccrs  # 投影方式
import matplotlib.pyplot as plt
import cartopy.feature as cfeat  # 使用shp文件
from cartopy.io.shapereader import Reader  # 读取自定的shp文件
from cartopy.mpl.ticker import LongitudeFormatter, LatitudeFormatter  # 将x，y轴换成经纬度

plt.rcParams['font.sans-serif'] = ['SimHei']


class Wind: pass


def plot_maxmin_points(lon, lat, data, extrema, nsize, symbol, color='k',
                       plotValue=True, transform=None):
    from scipy.ndimage.filters import maximum_filter, minimum_filter

    if (extrema == 'max'):
        data_ext = maximum_filter(data, nsize, mode='nearest')
    elif (extrema == 'min'):
        data_ext = minimum_filter(data, nsize, mode='nearest')
    else:
        raise ValueError('Value for hilo must be either max or min')

    mxy, mxx = np.where(data_ext == data)

    for i in range(len(mxy)):
        if (extent[0] < lon[mxy[i], mxx[i]] < extent[1] and extent[2] < lat[mxy[i], mxx[i]] < extent[3]):
            plt.text(lon[mxy[i], mxx[i]], lat[mxy[i], mxx[i]], symbol, color=color, size=24,
                     clip_on=True, horizontalalignment='center', verticalalignment='center',
                     transform=transform)
            plt.text(lon[mxy[i], mxx[i]], lat[mxy[i], mxx[i]],
                     '\n' + str(np.int64(data[mxy[i], mxx[i]])),
                     color=color, size=12, clip_on=True, fontweight='bold',
                     horizontalalignment='center', verticalalignment='top', transform=transform)
        # else:
        #     print(lon[mxy[i], mxx[i]], lat[mxy[i], mxx[i]])


def spot_uv(lon, lat, us, vs):

    # 给点增加文字
    for x in range(len(us)):
        for y in range(len(us[x])):
            ax.text(lon[x][y] - 2, lat[x][y] - 1, us[x][y], fontsize=6, verticalalignment='center',
                     horizontalalignment='center', rotation=0)
            ax.text(lon[x][y], lat[x][y] - 1, vs[x][y], fontsize=6, verticalalignment='center',
                     horizontalalignment='center', rotation=0)



def ReadFromFile(path, self):
    """
        读micaps第11类数据文件到内存
        :return: 
        """
    print("ttt")
    # 打开文件
    file_object = codecs.open(path, mode='r', encoding="GBK")
    all_the_text = file_object.read().strip()
    # 将文件格式化
    contents = re.split(r'[\s]+', all_the_text)
    file_object.close()
    if len(contents) < 17:
        return
    # 填充属性
    self.dataflag = contents[0].strip()
    self.style = contents[1].strip()
    self.title = contents[2].strip()
    self.yy = int(contents[3].strip())
    self.mm = int(contents[4].strip())
    self.dd = int(contents[5].strip())
    self.hh = int(contents[6].strip())
    self.forehh = int(contents[7].strip())
    self.level = contents[8].strip()
    self.deltalon = float(contents[9].strip())
    self.deltalat = float(contents[10].strip())
    self.beginlon = float(contents[11].strip())
    self.endlon = float(contents[12].strip())
    self.beginlat = float(contents[13].strip())
    self.endlat = float(contents[14].strip())
    self.sumlon = int(contents[15].strip())
    self.sumlat = int(contents[16].strip())
    self.distance = float(contents[17].strip())
    self.min = float(contents[18].strip())
    self.max = float(contents[19].strip())
    self.def1 = contents[20].strip()
    self.def2 = contents[21].strip()
    # 返回一个有终点和起点的固定步长的排列
    self.x = np.arange(self.beginlon, self.endlon + self.deltalon, self.deltalon)
    self.y = np.arange(self.beginlat, self.endlat + self.deltalat, self.deltalat)
    # 根据传入的两个一维数组参数生成两个数组元素的列表
    self.X, self.Y = np.meshgrid(self.x, self.y)
    # 初始化数据集合
    self.U = np.zeros((self.sumlat, self.sumlon))
    self.V = np.zeros((self.sumlat, self.sumlon))
    self.Z = np.zeros((self.sumlat, self.sumlon))
    # 填充数据
    for i in range(self.sumlon):
        for j in range(self.sumlat):
            self.U[j, i] = float(contents[17 + j * self.sumlon + i])

    vbegin = 17 + self.sumlat * self.sumlon
    for i in range(self.sumlon):
        for j in range(self.sumlat):
            self.V[j, i] = float(contents[vbegin + j * self.sumlon + i])

    for i in range(self.sumlon):
        for j in range(self.sumlat):
            self.Z[j, i] = math.sqrt(self.U[j, i] ** 2 + self.V[j, i] ** 2)
    return self


def Read4File(path, self):
    print("4")
    # 打开文件
    file_object = codecs.open(path, mode='r', encoding="GBK")
    all_the_text = file_object.read().strip()
    # 将文件格式化
    contents = re.split(r'[\s]+', all_the_text)
    file_object.close()
    if len(contents) < 23:
        return
    self.dataflag = contents[0].strip()
    self.style = contents[1].strip()
    self.title = contents[2].strip()
    self.yy = int(contents[3].strip())
    self.mm = int(contents[4].strip())
    self.dd = int(contents[5].strip())
    self.hh = int(contents[6].strip())
    self.forehh = int(contents[7].strip())
    self.level = contents[8].strip()

    self.deltalon = float(contents[9].strip())
    self.deltalat = float(contents[10].strip())
    self.beginlon = float(contents[11].strip())
    self.endlon = float(contents[12].strip())
    self.beginlat = float(contents[13].strip())
    self.endlat = float(contents[14].strip())

    self.sumlon = int(contents[15].strip())
    self.sumlat = int(contents[16].strip())
    self.distance = float(contents[17].strip())
    self.min = float(contents[18].strip())
    self.max = float(contents[19].strip())
    self.def1 = contents[20].strip()
    self.def2 = contents[21].strip()

    x = np.arange(self.beginlon, self.endlon + 0.9 * self.deltalon, self.deltalon)
    y = np.arange(self.beginlat, self.endlat + 0.9 * self.deltalat, self.deltalat)
    self.X, self.Y = np.meshgrid(x, y)
    if self.dataflag == 'diamond' and self.style == '4':
        begin = 22
        self.Z = np.zeros((self.sumlat, self.sumlon))
        if contents is not None:
            contents = np.array(contents[begin:]).astype(np.float64)
            self.Z = contents.reshape((self.sumlat, self.sumlon))
    return self


# 用命令调用py文件运行，并传值
# python micaps-hight-wind.py ../data/20010108.000 ../data/20010108(2).000 2020年1月1日08时700百帕高度场+风场 ../images/2020年1月1日08时700百帕高度场+风场.png
# args = sys.argv
# print(args)
# windPath = args[1]
# highPath = args[2]
# title = args[3]
# savePath = args[4]
title = '2020年1月1日08时700百帕高度场 + 风场'
wind = ReadFromFile("../data/20010108.000", Wind())
high = Read4File("../data/20010108(2).000", Wind())
# wind = ReadFromFile(windPath, Wind())
# high = Read4File(highPath, Wind())
# 设定投影
proj = ccrs.PlateCarree()
# 创建画布
fig = plt.figure(figsize=(9, 6), dpi=150)
ax = plt.axes(projection=proj)
# 定义数据经纬度范围
extent = [70, 140, 10, 60]
ax.set_extent(extent)
# 国界线
# country_shp = Reader('../shp/china/bou1_4l.shp')
# fea_country = cfeat.ShapelyFeature(country_shp.geometries(), proj,
#                                    edgecolor='#000000', facecolor='#F0F0D9')
# ax.add_feature(fea_country, linewidth=1, alpha=1)
# # 地理信息的添加
# ax.add_feature(cfeat.OCEAN.with_scale('50m'))
# ax.add_feature(cfeat.COASTLINE.with_scale('50m'), lw=0.6)
# ax.add_feature(cfeat.LAND.with_scale('50m'))
# ax.add_feature(cfeat.LAKES.with_scale('50m'))
# ax.add_feature(cfeat.RIVERS.with_scale('50m'), lw=0.5)
# # 添加经纬度标注
# ax.set_xticks([70, 85, 100, 115, 130])
# ax.set_yticks([10, 20, 30, 40, 50, 60])
# ax.xaxis.set_major_formatter(LongitudeFormatter())
# ax.yaxis.set_major_formatter(LatitudeFormatter())

# 绘制等高线
ac = ax.contour(high.X, high.Y, high.Z, transform=proj, colors='#090AFD', linewidths=1.2)

# 等高线标注
ax.clabel(ac, fontsize=14, colors='#090AFD', inline_spacing=1, fmt='%d')
# 绘制 H/L 高低压区标记 5 是设置GD标志的疏密度
plot_maxmin_points(high.X, high.Y, high.Z, 'max', 5,
                   symbol='G', color='b', transform=proj)
plot_maxmin_points(high.X, high.Y, high.Z, 'min', 5,
                   symbol='D', color='r', transform=proj)
# # 给风场增加文字 不行
# spot_uv(wind.X[::3, ::3], wind.Y[::3, ::3],
#          wind.U[::3, ::3], wind.V[::3, ::3])
# 绘制风向标 ::1是格点密度
ax.barbs(wind.X[::1, ::1], wind.Y[::1, ::1],
         wind.U[::1, ::1], wind.V[::1, ::1],
         barb_increments={'half': 2, 'full': 4, 'flag': 20}, zorder=5, length=5, linewidth=0.2)
#设置标题
# ax.set_title(title, fontsize=12)

ax.spines['top'].set_visible(False)
ax.spines['right'].set_visible(False)
ax.spines['bottom'].set_visible(False)
ax.spines['left'].set_visible(False)

# 保存图片
savePath = "../images/2020年1月1日08时700百帕高度场++风场.png"
#去掉白边，后面两个参数控制
plt.savefig(savePath, bbox_inches='tight', pad_inches=0.0)
#show必须在savefig后
fig.show()
# 清空画布
plt.clf()
print(wind.title)

#将白色底图转为透明
pic = Image.open(savePath)
# 转为RGBA模式
pic = pic.convert('RGBA')
width, height = pic.size
# 获取图片像素操作入口
array = pic.load()
for i in range(width):
    for j in range(height):
        # 获得某个像素点，格式为(R,G,B,A)元组
        pos = array[i, j]
        # 如果R G B三者都大于240(很接近白色了，数值可调整)
        isEdit = (sum([1 for x in pos[0:3] if x > 240]) == 3)
        if isEdit:
            # 更改为透明
            array[i, j] = (255, 255, 255, 0)

# 保存图片
pic.save(savePath)


```


