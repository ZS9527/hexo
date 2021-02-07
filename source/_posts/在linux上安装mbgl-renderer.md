---
title: 在linux上安装mbgl-renderer
date: 2020-10-10 16:10:51
tags: mbgl-renderer
---

# 在linux上安装mbgl-renderer
> docker 安装还是挺方便的。
> 这个软件包主要是用来 java 调用画图。基本上前端可以绘制出的界面，都可以绘制成图片保存。当然，我只是完成我需要的地图叠加。
> 
<!--more-->

## mbgl-renderer 安装
这个是我们组大佬找到的一种方法。为了应对客户提出的想将看到的前端界面截图保存并做成word文档保存的需求。
文档在 github [mbgl-renderer](https://github.com/consbio/mbgl-renderer)
1. 从Docker Hub中获取最新图像：
```
docker pull consbio/mbgl-renderer:latest
```
2. mbgl-server在端口8080的Docker容器中运行（并没有用这个）：
```
docker run --rm -p 8080:80 consbio/mbgl-renderer
```
3. 将本地tile目录挂载到/app/tiles容器中，以在服务器或CLI中使用它们（用这个）：
```
docker run --rm -p 9090:80 -v /root/mbgl:/app/tiles consbio/mbgl-renderer
```

## 调用绘图程序
调用代码
```java
/**
     * 绘图
     *
     * @param style 样式
     * @return bytes
     */
private byte[] render(JSONObject style) {
        if (style == null || style.isEmpty()) {
            throw new HandleException("绘图样式表为空");
        }

        Map<String, Object> renderParams = new HashMap<>(6);
        renderParams.put("ratio", 2);
        renderParams.put("width", 1024);
        renderParams.put("height", 1024);
        renderParams.put("style", style.toJSONString());
        renderParams.put("zoom", style.getDoubleValue("zoom"));
        JSONArray centers = style.getJSONArray("center");
        renderParams.put("center", centers.getDoubleValue(0) + "," + centers.getDoubleValue(1));

        return HttpUtil.createPost(RENDER_URL)
            .form(renderParams)
            .execute()
            .bodyBytes();
    }
```

其中的 `JSONObject style`是一段前端绘制地图代码。
然后再将返回的 byte[] 绘制成图片保存即可
```java
/**
     * 根据数据画图
     * @param bytes 数据
     * @param filepath 图片全路径，带文件名
     * @return 是否成功
     */
    private boolean drawImage(byte[] bytes, String filepath) {
        try {
            Path path = Paths.get(filepath);
            FileUtil.mkParentDirs(path.toString());
            // 图片写入磁盘
            Files.write(path, bytes);
        } catch (IOException e) {
            log.error(e.getMessage(), e);
            return false;
        }
        return true;
    }

```

## 更新
后来服务器关了一次机，再运行 docker 时报错
```
Is the docker daemon running?
```
启动 docker 即可
```
service docker start
```

