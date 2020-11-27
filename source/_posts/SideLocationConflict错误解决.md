---
title: side location conflict错误解决
date: 2020-10-03 11:35:15
tags: jts
---

# side location conflict错误解决
> 在进行两个面集合之间的关系比较时出现的问题
> 有关博文 [http://locationtech.github.io/jts/javadoc/org/locationtech/jts/geom/Geometry.html](http://locationtech.github.io/jts/javadoc/org/locationtech/jts/geom/Geometry.html) jar 包的api 文档。和 [https://github.com/nettopologysuite/nettopologysuite/issues/129](https://github.com/nettopologysuite/nettopologysuite/issues/129) 的一句话。
<!--more-->

## 错误
```
org.locationtech.jts.geom.TopologyException: side location conflict [ (112.094482, 25.851795, NaN) ]
```
在判断从 geojson 格式转成的 org.locationtech.jts.geom.Polygon 面之间的关系时发生

## 解决方法
首先贴一下我的 geojson 转 Polygon 面的代码
```
private List<Polygon> jsonToGeometry(String jsonFilePath) throws IOException {
        List<Polygon> list = new ArrayList<>();
        try (InputStream is = GeoTools.class.getResourceAsStream(jsonFilePath)) {
            JSONObject city = JSON.parseObject(IoUtil.read(is, StandardCharsets.UTF_8));
            list = city.getJSONArray("features").stream()
                    .map(obj -> {
                        JSONObject json = (JSONObject) obj;
                        Polygon polygon = null;
                        polygon = (Polygon) new GeoJSONReader().read(json.getJSONObject("geometry").toJSONString());
                        polygon.setUserData(json.getJSONObject("properties"));
                        return polygon;
                    })
                    .collect(Collectors.toList());
        } catch (IOException e) {
            throw new HandleException(e);
        }
        return list;
    }
```
jsonFilePath 参数就是一个 geojson 的文件。
所需引用 jar 包为
```
implementation group: 'org.locationtech.jts', name: 'jts-core', version: '1.16.1'
```
我所使用的对比面与面之间关系的方法是
```
/**
     * 把县级所属城市标记出来
     * @param polygons 县
     * @param citypolygons 市
     */
    private List<Polygon> belongingCity(List<Polygon> polygons, List<Polygon> citypolygons) {
        polygons = polygons.stream().map(polygon -> {
            List<String> list = new ArrayList<>();
            citypolygons.stream().forEach(citypolygo -> {
                try {
                    Polygon union = polygon.isValid() ? polygon : (Polygon)polygon.buffer(0);
                    Polygon cityunion = citypolygo.isValid() ? citypolygo : (Polygon)citypolygo.buffer(0);
                    if (cityunion.contains(union) || cityunion.overlaps(union)) {
                        String name = ((JSONObject)polygon.getUserData()).getString("name");
                        String cityName = ((JSONObject)citypolygo.getUserData()).getString("name");
                            name = cityName + name;
                        ((JSONObject)polygon.getUserData()).put("name", name);
                        list.add(name);
                    }

                } catch (TopologyException e) {
                   log.info(e.getMessage(), e);
                }
            });
            return polygon;
        }).collect(Collectors.toList());
        return polygons;
    }
```
这个判断面与面之间的关系这里还是有一点问题，将一个县分到了两个市所属下。
其中解决这次报错的方法是
```
polygon.isValid() ? polygon : (Polygon)polygon.buffer(0)
```

`isValid`方法是判断 Geometry 根据OGC SFS规范测试这在拓扑上是否有效。
然后再用`buffer`方法拿出一个面。其中有一些要注意的点，在原文回答中有写：
>There are two possible reasons that you are getting this exception:
>Most likely you have invalid geometries to start with. You can test geometries for validity by either checking feature.Geometry.IsValid property or by using the NetTopologySuite.Operation.Valid.IsValidOp class. The later gives you the location where the validation test failed.
If this is the case you can attempt to fix the geometry by assigning feature.Geometry = feature.Geometry.Buffer(0);. While this works for most of the cases, it is not guaranteed to do so. An example where this might not work is a "bow-tie: or "figure-8" polygon, with one very small lobe and one large one. Depending on the orientations of the lobes, the Buffer(0) operation may keep the small lobe and discard the "valid" large lobe.
It can also happen during operations on geometries due to some subtle kinds of cross-geometry
topology, such as two line segments being almost but not quite coincident. This can be avoided by using the NetTopologySuite.Precision.GeometryPrecisionReducer.
I would need to see the geometries to tell more.

要注意自相交面处理后的截取的面是否为小的那个面。
成功解决问题