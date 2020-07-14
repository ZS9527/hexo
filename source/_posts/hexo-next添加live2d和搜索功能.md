---
title: hexo+next添加live2d和搜索功能
date: 2020-05-01 18:42:48
tags:
---

# hexo+next添加live2d和搜索功能
> 之前的网站看腻了，就来换个新的设计。再加上修改自己使用时候出现的一些不方便的地方。都是直接用原文中的步骤进行就可以了。
> 原文地址[Hexo Next主题添加搜索功能](https://www.jianshu.com/p/202c9e789c8f)
> [hexo-helper-live2d](https://github.com/EYHN/hexo-helper-live2d/blob/master/README.zh-CN.md)

<!--more-->

## Hexo Next主题添加搜索功能
1. 在hexo的根目录下执行命令：
```
npm install hexo-generator-searchdb --save
```

2. 在根目录下的/theme/next/\_config.yml文件中添加配置：
```
search:
  path: search.xml
  field: post
  format: html
  limit: 10000
```
3. 在根目录下的/theme/next/\_config.yml文件中搜索`local_search`，将`enable`改为`true`：
```
local_search:
  enable: true
```
完成。

## 添加hexo-helper-live2d
1. 安装模块
```
npm install --save hexo-helper-live2d
```
2. 向Hexo的\_config.yml 文件或主题的 \_config.yml 文件中添加配置。（我是添加在了hexo的文件中）
```
# hexo-helper-live2d 看板娘
live2d:
  model:
    use: live2d-widget-model-koharu
    scale: 1
    hHeadPos: 0.5
    vHeadPos: 0.618
  display:
    superSample: 2
    width: 150
    height: 300
    position: left
    hOffset: 0
    vOffset: -20
  mobile:
    show: true
    scale: 0.5
  react:
    opacityDefault: 0.7
    opacityOnHover: 0.2
```
3. 更换一个模型
```
npm install live2d-widget-model-koharu
```
4. 添加点击弹出对话功能