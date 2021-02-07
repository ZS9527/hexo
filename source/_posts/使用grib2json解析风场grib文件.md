---
title: 使用grib2json解析风场grib文件
date: 2020-10-17 18:28:54
tags: grib2json
---

# 使用grib2json解析风场grib文件
> 本 jar 包由大佬在研究绘制风场动态图时提供。
> 参考博客 [https://www.iteye.com/blog/umbrellall1-2509006](https://www.iteye.com/blog/umbrellall1-2509006)

<!--more-->

## 尝试过程
大佬提供了这个 jar 包的 github 地址。我去研究了一下，把源码 clone 到本地。
到本地时出现了 `edu.ucar.grib` 依赖包无法下载问题，尝试更换 maven 源到阿里云，依旧不行。
在 [https://mvnrepository.com/](https://mvnrepository.com/) 中搜索了一下。主仓库Central中没能搜到4.3.19版本，其他版本并不能使用。但是在别的仓库 Boundless 中搜索到，但是数据仓库已经404。
向大佬求助，大佬查到了这个 jar 包的官网 [https://docs.unidata.ucar.edu/netcdf-java/current/userguide/using_netcdf_java_artifacts.html](https://docs.unidata.ucar.edu/netcdf-java/current/userguide/using_netcdf_java_artifacts.html) 。但是我更换这个官网的源后，依旧不能下载。没办法只能手动替换 maven 文件夹中的 jar 包，这才成功的开了起来。

### 第二个问题
开启后，由于我本人开发经验太少的原因。没有接触过 `CliFactory.parseArguments(Options.class, args);` 相关。
按照步骤将源码 `maven package` 再试图使用命令行 `grib2json --names --data --fp 2 --fs 103 --fv 10.0 gfs.t18z.pgrbf00.2p5deg.grib2` 没能得到想要的结果。
通过关键字 `CliFactory.parseArguments` 找到了上面的那篇博客，才成功的将命令行输对。
``` java
public static void main(String[] args) {
        try {
//            args = new String[]{
//                "--data","--fp wind","--filter.surface 103","--filter.value 10.0","E:\\idea-workspace\\grib2json\\target\\grib2json-0.8.0-SNAPSHOT\\bin\\ Z_NAFP_C_BABJ_20201203000801_P_CLDAS_RT_CHN_0P05_HOR-WIN-2020120300.grib2"};
            args = new String[]{
                "--data","--names","--filter.parameter","wind","--filter.surface","103","--filter.value", "10.0","--output", "D://a.txt", "D://text.GRB2"};

            Options options = CliFactory.parseArguments(Options.class, args);
            if (options.getShowHelp() || options.getFile() == null) {
                printUsage();
                System.exit(options.getShowHelp() ? 0 : 1);
                return;
            }

            LoggerContext lc = (LoggerContext) LoggerFactory.getILoggerFactory();
            if (!options.getEnableLogging()) {
                lc.stop();
            }

            List<Options> optionGroups = options.getRecipe() != null ?
                readRecipeFile(args, options.getRecipe()) :
                Collections.singletonList(options);

            new Grib2Json(options.getFile(), optionGroups).write();
        }
        catch (JewelRuntimeException t) {
            printUsage();
            System.out.println();
            System.err.println(t.getMessage());
            System.exit(1);
        }
        catch (IllegalArgumentException e) {
            System.err.println(e.getMessage());
            System.exit(1);
        }
        catch (Throwable t) {
            t.printStackTrace(System.err);
            System.exit(2);
        }
    }
```

之后就将解析出来的文件返回给了前端使用。