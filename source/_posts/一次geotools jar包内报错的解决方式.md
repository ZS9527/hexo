---
title: 一次geotools jar包内报错的解决方式
date: 2019-08-02 19:26:40
tags: bug
---
# 一次geotools jar包内报错的解决方式
> 在此记录一下geotools这个jar包在使用中碰到的问题。目前仍旧没有解决，只是解决了一部分。

<!--more-->

## 最根本需求：
目前碰到的需求是需要将矩形地图按照省级边界线将多余的地方裁剪。一切的问题都是出在解决这个需求的路上发生的。

### 第一个bug
**关键字：** Points of LinearRing do not form a closed linestring。

问题报错出现在了jar包内的org.geotools.geojson.geom.PolygonHandler这个类的第54行到65行
```
try {
                LinearRing outer = factory.createLinearRing(rings.get(0));
                LinearRing[] inner = null;
                if (rings.size() > 1) {
                    inner = new LinearRing[rings.size() - 1];
                    for (int i = 1; i < rings.size(); i++) {
                        inner[i - 1] = factory.createLinearRing(rings.get(i));
                    }
                }

                value = factory.createPolygon(outer, inner);
                rings = null;
            } catch (IllegalArgumentException e) {
                return true;
            }
```
**处理方式：**暂时只能捕获异常后返回正常的值。在java目录下新建一个和本类相同的类和jar包路径，重新rebuild项目。即可让jar包在调用时使用自己修改过的这个类
![](b1.png)

### 第二个bug
使用格点文件直接进行裁剪和叠加色斑图时，会产生一些未裁剪成功的图，以及和未裁剪的色斑图的拼图面不一致的情况。目前的处理方式是将格点离散成站点格式，再重新插值为格点。希望这种方式可以处理裁剪后色斑拼图丢失的问题。

**2019年09月04日更新**
没能想到别的办法来处理这个裁剪的问题，而且上述的离散再插值的方式，经过长时间的采集数据发现，仍会有丢失色斑的情况出现。只能放弃从后端来处理地图边界问题，转而从前端用空白背景来掩盖未被裁剪掉的省外部分。