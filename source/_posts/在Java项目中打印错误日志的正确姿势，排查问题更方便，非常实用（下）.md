---
title: 在Java项目中打印错误日志的正确姿势，排查问题更方便，非常实用（下）
date: 2020-07-04 17:53:04
tags: Java
---

# 在Java项目中打印错误日志的正确姿势，排查问题更方便，非常实用（下）

> 在看公众号的时候发现的，拆成了两篇文章。阅读起来方便一点
> 作者：琴水玉
> cnblogs.com/lovesqcc/p/4319594.html

<!--more-->

## 如何编写更容易排查问题的错误日志

### 打错误日志的基本原则：

1. **尽可能完整。**每一条错误日志都完整描述了：什么场景下发生了什么错误， 什么原因（或者哪些可能原因）， 如何解决（或解决提示）；
2. **尽可能具体。**比如 NC 资源不足， 究竟具体指什么资源不足， 是否可以通过程序直接指明；通用错误，比如 VM NOT EXIST ， 要指明在什么场景下发生的，可能便于后续统计的工作。
3. **尽可能直接。**最理想的错误日志应该让人在第一直觉下能够知道是什么原因导致，该怎么去解决，而不是还要通过若干步骤去查找真正的原因。
4. **将已有经验集成直接到系统中。**所有已经解决过的问题及经验都要尽可能以友好的方式集成到系统中，给新进人员更好的提示，而不是埋藏在其他地方。
5. **排版要整洁有序， 格式统一化规范化。**密密麻麻、随笔式的日志看着就揪心， 相当不友好， 也不便于排查问题。
6. **采用多个关键字唯一标识请求**，突出显示关键字：时间、实体标识（比如vmname）、操作名称。

### 排查问题的基本步骤：
```
登录到应用服务器 -> 打开日志文件 -> 定位到错误日志位置 -> 根据错误日志的线索的指导去排查、确认问题和解决问题。
```

其中：
1. **从登陆到打开日志文件：**由于应用服务器有多台， 要逐一登录上去查看实在不方便。 需要编写一个工具放在 AG 上直接在 AG 上查看所有服务器日志， 甚至直接筛选出所需要的错误日志。
2. **定位错误日志位置。** 目前日志的排版密密麻麻，不易定位到错误日志。一般可以先采用"时间"来定位到错误日志的附近前面的地方， 然后使用 实体关键字 / 操作名称 组合来锁定错误日志地方。 根据 requestId 定位错误日志虽然比较符合传统，但是要先找到 requestId , 并且不具有描述性。最好能直接根据时间/内容关键字来定位错误日志位置。
3.  **分析错误日志。**错误日志的内容最好能够更加直接明了， 能够明确指明与当前要排查的问题特征是吻合的， 并且给出重要线索。

通常， 程序错误日志的问题就是日志内容是针对当前代码情境才能理解，看上去简洁， 但总是写的不全， 半英文格式；一旦离开代码情境， 就很难知道究竟说的是什么， 非要让人思考一下或者去看看代码才能明白日志说的是什么含义。 这不是自己给自己罪受？

```
if ((storageType == StorageType.dfs1 || storageType == StorageType.dfs2)
                && (zone.hasStorageType(StorageType.io3) || zone.hasStorageType(StorageType.io4))) {
// 进入dfs1 和dfs2 在io3 io4 存储。
} else {
      log.info("zone storage type not support, zone: " + zone.getZoneId() + ", storageType: "
+ storageType.name());
      throw new BizException(DeviceErrorCode.ZONE_STORAGE_TYPE_NOT_SUPPORT);
}
```
zone 要支持什么 storage type 才是正确的?   Do Not Let Me Think !

错误日志应该做到： 即使离开代码情境，也能清晰地描述发生了什么。

此外，如果能够直接在错误日志中说明清楚原因， 在做巡检日志的时候也可以省些力气。 
从某种意义上来说， 错误日志也可以是一种非常有益的文档，记录着各种不合法的运行用例。

### 程序错误日志的内容可能存在如下问题：

#### 1. 错误日志没有指明错误参数和内容：
```
catch(Exception ex){
      log.error("control ip insert failed", ex);
      return new ResultSet<AddControlIpResponse>(
ControlIpErrorCode.ERROR_CONTROL_IP_INSERT_FAILURE);
}
```
没有指明插入失败的 control ip. 如果加上 control ip 关键字， 更容易搜索和锁定错误。

类似的还有：
```
log.error("Get some errors when insert subnet and its IPs into database. Add subnet or IP failure.", e);
```
没有指明是哪个 subnet 的它下属的哪些 IP. 值得注意的是， 要指明这些要额外做一些事情， 可能会稍微影响性能。这时候需要权衡性能和可调试性。

**解决方案：**使用 String.format("Some msg to ErrorObj: %s", errobj) 方法指明错误参数及内容。
这通常要求对 DO 对象编写可读的 toString 方法。

#### 2. 错误场景不明确：
```
log.error("nc has exist, nc ip" + request.getIp());
```
在 createNc 中检测到 NC 已经存在报错。 但是日志上没有指明错误场景， 让人猜测，为什么会报 NC 已存在错误。

可以改为
```
log.error("nc has exist when want to create nc, please check nc parameters. Given nc ip: " + request.getIp());
log.error("[create nc] nc has exist, please check nc parameters. Given nc ip: " + request.getIp());
```

类似的还有：
```
log.error("not all vm destroyed, nc id " + request.getNcId());
```

可以改为
```
log.error("[delete nc] some vms [%s] in the nc are not destroyed. nc id: %s", vmNames, request.getNcId());
```

**解决方案：** 错误消息加上 when 字句， 或者错误消息前加上 【接口名】,  指明错误场景，直接从错误日志就知道明白了。
一般能够知道 executor 的可以加上 【接口名】， service 加上 when 字句。

#### 3. 内容不明确, 或不明其义：

```
if(aliMonitorReporter == null) {
        log.error("aliMonitorReporter is null!");
} else {
       aliMonitorReporter.attach(new ThreadPoolMonitor(namePrefix, asynTaskThreadPool.getThreadPoolExecutor()));
}
```

改为
```
log.error("aliMonitorReporter is null, probably not initialized properly, please check configuration in file xxx.");
```

类似的还有：
```
if (diskWbps == null && diskRbps == null && diskWiops == null    && diskRiops == null) {
      log.error("none of attribute is specified for modifying");
      throw new BizException(DeviceErrorCode.NO_ATTRIBUTE_FOR_MODIFY);
}
```

**解决方案：** 更清晰贴切地描述错误内容。

#### 4. 排查问题的引导内容不明确：
```
log.error("get gw group ip segment failed. zkPath: " + LockResource.getGwGroupIpSegmnetLockPath(request.getGwGroupId()));
```
zkPath ?  如何去排查这个问题？ 我该去找谁？ 到哪里去查找更具体的线索？

**解决方案：** 加上相应的背景知识和引导排查措施。

#### 5. 错误内容不够具体细致：
```
if (!ncResourceService.isNcResourceEnough(ncResourceDO,    vmResourceCondition)) {
      log.error("disk space is not enough at vm's nc, nc id:" + vmDO.getNcId());
      throw new BizException(ResourceErrorCode.ERROR_RESOURCE_NOT_ENOUGH);
}
```
究竟是什么资源不够？ 目前剩余多少？ 现在需要多少？ 值得注意的是， 要指明这些要额外做一些事情， 可能会稍微影响性能。这时候需要权衡性能和可调试性。

**解决方案：**  通过改进程序或程序技巧， 尽可能揭示出具体的差异所在， 减少人工比对的操作。

#### 6. 半英文句式读起来不够清晰明白，需要思考来拼凑起完整的意思：
```
log.warn("cache status conflict, device id "+deviceDO.getId()+" db status "+deviceDO.getStatus() +", nc status "+ status);
```
改为
```
log.warn(String.format("[query cache status] device cache status conflicts between regiondb and nc, status of device '%s' in regiondb is %s , but is %s in nc.", deviceDO.getId(), deviceDO.getStatus(), status));
```

**解决方案：** 改为自然可读的英文句式。

#### 总结
错误日志格式可以为：
```
log.error("[接口名或操作名] [Some Error Msg] happens. [params] [Probably Because]. [Probably need to do].");

或者

log.error(String.format("[接口名或操作名] [Some Error Msg] happens. [%s]. [Probably Because]. [Probably need to do].", params));

或者

log.error("[Some Error Msg] happens to 错误参数或内容 when [in some condition]. [Probably Because]. [Probably need to do].");

或者

log.error(String.format("[Some Error Msg] happens to %s when [in some condition]. [Probably Because]. [Probably need to do].", parameters));

```

[Probably Reason]. [Probably need to do]. 在某些情况下可以省略； 在一些重要接口和场景下最好能说明一下。

每一条错误日志都是独立的，尽可能完整、具体、直接说明何种场景下发生了什么错误，由什么原因导致，要采用什么措施或步骤。

**问题：**
1.  String.format 的性能会影响打日志吗？ 一般来说， 错误日志应该是比较少的， 使用 String.format 的频度并不会太高，不会对应用和日志造成影响。
2.  开发时间非常紧张时， 有时间去斟酌字句吗？ 建立一个标准化的内容格式，将内容往格式套，可以节省斟酌字句的时间。
3.  什么时候使用 info, warn , error ? 
info 用于打印程序应该出现的正常状态信息， 便于追踪定位； 
warn 表明系统出现轻微的不合理但不影响运行和使用； 
error 表明出现了系统错误和异常，无法正常完成目标操作。

错误日志是排查问题的重要手段之一。 当我们编程实现一项功能时， 通常会考虑可能发生的各种错误及相应原因：
要排查出相应的原因， 就需要一些关键描述来定位原因。这就会形成三元组：
错误现象 -> 错误关键描述 -> 最终的错误原因。
需要针对每一种错误尽可能提供相应的错误关键描述，从而定位到相应的错误原因。
也就是说，编程的时候，要仔细思考， 哪些描述是非常有利于定位错误原因的， 尽可能将这些描述添加到错误日志中。
 
 
文中没有指出的问题或困难，  请提出你的建议。