---
title: 在windows上的WSL上安装onlyoffice
date: 2021-01-23 17:14:07
tags: ONLYOFFICE
---

# 在windows上的WSL上安装onlyoffice

> 需要一个在线文档编辑功能，之前的OOS安装起来太麻烦了。最关键的还是排版和在本地的文档排版不一样。不能满足要求
> 就更换成了现在的这个OnlyOFFICE，这个软件有开源版本和商业版本，开源版本限制同时编辑人数，但是自然有办法绕过去。
> 官方源码：[https://github.com/ONLYOFFICE/Docker-DocumentServer](https://github.com/ONLYOFFICE/Docker-DocumentServer)
> 我使用的Demo：[https://github.com/xsi640/onlyoffice_demo](https://github.com/xsi640/onlyoffice_demo)
> 官方的Demo有点难懂，而且我部署在本地后还会报错，只好切换成别人的。
> 包含onlyoffice 添加中文字体
<!--more-->

## 安装步骤
首先确保你的电脑上装好了 Docker。至于 Windows 上的 WSL 怎么装 Docker 就不记录了。
### 初级版本：先跑起来
直接使用最简单的命令
```
docker run -i -t -d -p 9090:80 onlyoffice/documentserver
```
之后会自动安装好。然后访问自己电脑 ip + 9090，注意不要用 localhost 或者 127.0.0.1 。会出现稀奇古怪的问题，详情要看[https://www.jianshu.com/p/8bba5d06eb78](https://www.jianshu.com/p/8bba5d06eb78) ，其中还有解决上传文件后，点击文件无法保存等问题。
然后访问 ip + 9090 就可以查看是不是装好了服务端。
![b1.png](b1.png)

然后可以使用图中的命令，创建一个例子来测试一下功能。docker 运行命令后，就可以访问
ip + 9090/example/ 来使用例子。或者点击上图命令下方的 <u>here</u> 也可以进入。

```
docker exec 2bb19c45c17f sudo supervisorctl start ds:example
```
![b2.png](b2.png)

### 进阶版本：把文件夹映射出来
使用命令
```
docker run -i -t -d -p 9090:80 -v /e/docker/onlyoffice/logs:/var/log/onlyoffice -v /e/docker/onlyoffice/data:/var/www/onlyoffice/Data  onlyoffice/documentserver

```
这样就可以把日志文件夹，数据文件文件夹，字体文件夹映射出来了。

## 安装后安装中文字体
尝试添加一个字体，为了命令运行速度，删除了其他所有字体。

进入docker 容器的fonts文件夹，其中有truetype文件夹和X11文件夹。
```
docker exec 76bcc /bin/bash
cd /usr/share/fonts/
```

删除其他字体：只保留truetype文件夹
```
rm -R X11
cd truetype
rm -R *
exit
```

加入自己的字体（注意字体要用下面的方法把显示名称切换为中文的，不然会直接显示拼音），这里的 fonts 文件夹是在我 C 盘根目录下新建的目录。
```
docker cp /fonts/ 0750:/usr/share/fonts/
```

再次进入容器，清除字体缓存
```
docker exec -it 76bcc /bin/bash
sudo mkfontscale
sudo mkfontdir
sudo fc-cache -fv
exit
```

运行字体脚本，一行Done，两行stopped，两行started。之后完成
```
docker exec 0750 /usr/bin/documentserver-generate-allfonts.sh
```

清除浏览器缓存后，就发现字体选择哪里有自己添加的字体了。如果没成功，就用别的浏览器看看有没有，可能是缓存没清除的原因。我这里是因为清除了其他的字体缘故，用其他字体的时候会显示是小方块。

如果字体显示是拼音，那就需要用到一个字体编辑器：FontCreator 。将字体导入到里面后，选择
![b3.png](b3.png)

![b4.png](b4.png)
之后导出文件，保存为 ttf 格式。然后再把这个文件用上面的方法导入进去，就显示你输入的中文名称了。

从CSDN中找到了一个中文字体压缩包，比自己做字体方便多了。



## 后续问题
这里中文字体有个问题，显示出的结果和wps中的同名字体不太一样。因为时间问题，暂停研究。