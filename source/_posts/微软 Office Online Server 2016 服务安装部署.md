---
title: 微软 Office Online Server 2016 服务安装部署
date: 2020-04-25 10:52:38
tags: office
---

# 微软 Office Online Server 2016 服务安装部署
> 结合了几篇文章。每个文章都用了一部分。
> 一、 [微软 Office Online Server 2016 服务安装部署 + wopi代码实现](https://blog.csdn.net/jiaqu2177/article/details/81945692)
> 二、[Office Online Server 在线编辑Office文档，安装部署](https://www.netnr.com/home/list/114)
> 三、 [https://www.netnr.com/doc/code/4964095842855914510/4935912187725186675](https://www.netnr.com/doc/code/4964095842855914510/4935912187725186675)
> 四、 [office online server2016详细安装步骤及问题总结](https://blog.csdn.net/q386815991/article/details/81705128)

<!--more-->
## 一、转换服务器部署
我在本机使用vmware打开两个虚拟机，将office online server 和wopi都安装在了转换服务器上，域服务器只提供了一个域。之前大佬曾经尝试过将东西都安装在一个服务器上，没有配置转换服务器。出现了word可以打开，但是没有图片。excel只能打开没有格式的空文件，ppt无法打开的情况。
有博客说：
> 由于云服务器是克隆副本，加入域会报错：无法完成域加入，原因是试图加入的域的 SID 与本计算机的 SID 相同，解决方法，打开：windows/System32/Sysprep/Sysprep.exe，勾选 通用 重启

### office online资源
office online server 2016的安装包使用的是：
```
ed2k://|file|cn_office_online_server_last_updated_march_2017_x64_dvd_10245068.iso|730759168|DA70F58CB8FFAF37C02302F2501CE635|/
```

补丁包没有用上，安装的时候报找不到版本。但还是把连接放出来：
```
https://download.microsoft.com/download/6/B/D/6BD1D664-1212-4AB2-9BE8-447731F2CA0E/wacserver2016-kb4011025-fullfile-x64-glb.exe
```

语言包还是要装的：
```
http://go.microsoft.com/fwlink/p/?LinkId=798136
```

这些资源来自二博客的下图：
![](b1.png)

### 安装office online

1. 运行 powershell， 输入命令（Windows Server 2012 R2版本）
```
Add-WindowsFeature Web-Server,Web-Mgmt-Tools,Web-Mgmt-Console,Web-WebServer,Web-Common-Http,Web-Default-Doc,Web-Static-Content,Web-Performance,Web-Stat-Compression,Web-Dyn-Compression,Web-Security,Web-Filtering,Web-Windows-Auth,Web-App-Dev,Web-Net-Ext45,Web-Asp-Net45,Web-ISAPI-Ext,Web-ISAPI-Filter,Web-Includes,InkandHandwritingServices,NET-Framework-Features,NET-Framework-Core,NET-HTTP-Activation,NET-Non-HTTP-Activ,NET-WCF-HTTP-Activation45,Windows-Identity-Foundation,Server-Media-Foundation
```

2. 配置环境，一定要按顺序安装
.NET Framework 4.5.2

https://go.microsoft.com/fwlink/p/?LinkId=510096

Visual C++ Redistributable Packages for Visual Studio 2013

https://www.microsoft.com/download/details.aspx?id=40784

Visual C++ Redistributable for Visual Studio 2015

https://go.microsoft.com/fwlink/p/?LinkId=620071

Microsoft.IdentityModel.Extention.dll

https://go.microsoft.com/fwlink/p/?LinkId=620072

3. 将下载好的office online server安装包解压好，并点击setup.exe开始安装。一路下一步安装就行。
4. 注意事项
    - 不要在装SQL Server等服务上运行
    - 不要在端口 80、443 或 809 上安装依赖 Web 服务器 (IIS) 角色的任何服务或角色
    - 不要安装任何版本的 Office
    - 不要在域控制器上安装 Office Online Server
    - 简而言之：一台干净的服务器

### 配置启动office online
打开安装好office online的转换服务器，打开powerShell
启动服务场
```
Import-Module OfficeWebApps
```
> 如果报错，需要重启

部署服务器场: 
```
New-OfficeWebAppsFarm -InternalURL “http://xx.domin.com” -ExternalUrl “http://192.168.37.138” -AllowHttp –EditingEnabled
```
–InternalURL内部访问地址，一般是http://office主机名.AD域控地址(如下图)； 全名的查看是在转换服务器的系统属性，计算机名里。

–AllowHttp 是否允许http访问；

–ExternalUrl 外部访问地址，一般是服务器的ip地址；

–EditingEnabled 允许编辑office。
> 请以域管理账号登录

出现如下提示即部署成功
![](b2.png)

这个时候用可以看下有没有启动起来
在office online 的浏览器 输入:http://win-5ldi2svjqoj.test.com/hosting/discovery 得到以下的结果就ok 了
![](b3.png)
当然通过ip地址访问也是ok,http://192.168.37.138/hosting/discovery
当然这两个地址在域控服务器上面访问 也是ok 的

### 额外配置
office online的日志地址是
![](b5.png)

安装后的office online server 对大文件会有限制，所以需要配置才能进行访问，具体配置路径如下
![](b6.png)
![](b7.png)
将上面两个文件夹中的settings文件进行修改，在其中填入并保存
```
OpenFromUrlMaxFileSizeInKBytes=(System.Int32)512000
```
![](b8.png)
配置完成后打开CMD命令，输入services.msc打开服务，并找到office online服务。重启即可


## 安装wopi项目
本人使用的是[https://github.com/netnr/WopiHost](https://github.com/netnr/WopiHost) 。
下载压缩包，解压到任意文件夹中。
服务器管理器 -> 工具 -> （iis）管理器 -> 添加了网站。
![](b4.png)
图中的接口配置为1080
配置好后尝试使用
http://192.168.37.138:1080/wopi/files/upload/word.docx/contents
可以访问到一个显示文件信息的界面
![](b9.png)

根据第三个博客，将http://192.168.37.138:1080/wopi/files/upload/word.docx 用`encodeURIComponent`重新进行编码。

事例，其中的`officeserver`是上面的office online信息的访问地址或ip（win-5ldi2svjqoj.test.com 或 192.168.37.138）
```
http://officeserver/we/WordEditorFrame.aspx?ui=zh-CN&WOPISrc=http%3A%2F%2Fwopiserver%2Fwopi%2Ffiles%2Fupload%2Fdoc%2F2019%2F03%2Fword.docx%3Faccess_token%3D1%26UserId%3D1%26UserName%3D1

http://officeserver/wv/WordViewer/request.pdf?type=accesspdf&WOPISrc=http%3A%2F%2Fwopiserver%2Fwopi%2Ffiles%2Fupload%2Fdoc%2F2019%2F03%2Fword.docx%3Faccess_token%3D1%26UserId%3D1%26UserName%3D1
```

到这里可以显示下图就算成功。期间出现的一些bug，已经记录在了之前的博客里面。
![](b10.png)

