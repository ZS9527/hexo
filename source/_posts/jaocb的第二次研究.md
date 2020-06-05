---
title: jacob的第二次研究
date: 2020-03-07 17:05:02
tags: office
---

# jacob的第二次研究
> 这是第二次研究用java来将office文档转换成pdf图片展示。记录一下大佬提供的资料，[https://blog.csdn.net/u014727260/article/details/53858329](https://blog.csdn.net/u014727260/article/details/53858329) 。

<!--more-->

## 第一次研究历程
很久很久以前（刚刚入行的时候），有一个需求是将office转换成图片展示。当时提供给我的思路是用虚拟打印机来将文档转换，但是图片质量不好。后来尝试了poi、aspose，但是生成图片效果都太模糊，连文字都看不清楚，就切换到了jacob。

jacob也有很多缺点，我安装的版本不兼容office，只能安装wps用来调用。需要将jacob.dll放在jdk中的\jre\bin中。

但是jacob转换成的pdf的确是清晰了很多，比之前的jpg，png的展示效果也好。后来大佬写了一个pdf转SVG的工具，效果就更加的棒了，感觉就像是从240P到了1080P。

## 本次研究
上次的jacob的使用时，直接找到了一篇博客，把里面的内容拿了过来使用，并未仔细研究方法。

这次是需要将excel转换成pdf。一开始翻阅了一下jacob的源码，发现里面是调用的office的公开api。就去阅读了一下微软官网的文档[https://docs.microsoft.com/zh-cn/office/vba/api/word.document.exportasfixedformat](https://docs.microsoft.com/zh-cn/office/vba/api/word.document.exportasfixedformat) 。

### word转pdf

```
private static boolean word2PDF(String inputFile, String pdfFile) {
        int wdFormatPDF = 17;// word2pdf
        ComThread.InitSTA();
        ActiveXComponent app = null;
        Dispatch doc = null;
        Timer timer = new Timer();
        timer.schedule(new TimerTask() {
            @Override
            public void run() {
                if (!isOpen) {
                    try {
                        Runtime.getRuntime().exec("taskkill /F /IM wpp.exe /T");
                        Runtime.getRuntime().exec("taskkill /F /IM wps*.exe /T");
                    } catch (Exception e) {
                        e.printStackTrace();
                    } finally {
                        // 释放资源
                        ComThread.Release();
                    }
                }
            }
        }, 10000);
        try {
            app = new ActiveXComponent("Word.Application");// 创建一个word对象
            app.setProperty("Visible", new Variant(false)); // 不可见打开word
            app.setProperty("AutomationSecurity", new Variant(3)); // 禁用宏
            Dispatch docs = app.getProperty("Documents").toDispatch();// 获取文挡属性
            // 调用Documents对象中Open方法打开文档，并返回打开的文档对象Document
            doc = Dispatch.call(docs, "Open", inputFile, false, true).toDispatch();
            isOpen = true;
            // 调用Document对象的SaveAs方法，将文档保存为pdf格式
            Dispatch.call(doc, "SaveAs", pdfFile, wdFormatPDF);// word保存为pdf格式宏，值为17
            // word保存为pdf格式宏，值为17
            return true;
        } catch (Exception e) {
            log.error("========Error:文档转换失败：" + e.getMessage(), e);
        } finally {
            Dispatch.call(doc, "Close", false);
            if (app != null) {
                app.invoke("Quit", new Variant[]{});
            }
            timer.cancel();
            ComThread.Release();
            ComThread.quitMainSTA();
        }
        return false;
    }
```

这里是word转换pdf，其中的定时器**Timer.schedule**是为了解决一种bug。曾经出现过打开文件失败后卡住进程，导致后台开启了很多wps占用内存。暂时没有找到原因，直接用结束进程的方式来解决了。

### excel转pdf
```
private static boolean excel2PDF(String inputFile, String pdfFile) {
        ComThread.InitSTA();
        ActiveXComponent app = null;
        Timer timer = new Timer();
        timer.schedule(new TimerTask() {
            @Override
            public void run() {
                if (!isOpen) {
                    try {
                        Runtime.getRuntime().exec("taskkill /F /IM wpp.exe /T");
                        Runtime.getRuntime().exec("taskkill /F /IM wps*.exe /T");
                    } catch (Exception e) {
                        e.printStackTrace();
                    } finally {
                        // 释放资源
                        ComThread.Release();
                    }
                }
            }
        }, 10000);
        try {
            // 创建一个excel对象
            app = new ActiveXComponent("Excel.Application");
            // 不可见打开
            app.setProperty("Visible", new Variant(false));
            // 禁用宏
            app.setProperty("AutomationSecurity", new Variant(3));
            // 获得excel中所有打开的文档,返回documents对象
            Dispatch docs = app.getProperty("Workbooks").toDispatch();
//            调用Documents对象中Open方法打开文档，并返回打开的文档对象Document
            Dispatch excel = Dispatch
                .invoke(docs, "Open", Dispatch.Method,
                    new Object[]{inputFile, new Variant(false), new Variant(false)}, new int[9])
                .toDispatch();
            //获得一个sheets集合
            Dispatch currentSheet = Dispatch.get(excel, "Sheets").toDispatch();
            //获得集合数量
            int count = Integer.valueOf(Dispatch.get(currentSheet, "Count").getInt());
            for (int i = 1; i <= count; i++) {
                //根据表格下表获得工作表
                Dispatch sheetOne = Dispatch
                    .invoke(currentSheet, "Item", Dispatch.Get, new Object[]{new Variant(i)},
                        new int[1]).getDispatch();
                //获得工作表页面属性设置
                Dispatch PageSetup = Dispatch.get(sheetOne, "PageSetup").toDispatch();
                //更改页面设置为取消
                Dispatch.put(PageSetup, "Zoom", false);
                //更改打印方向为横向（只有1.87类型是excel）
                Dispatch.put(PageSetup, "Orientation", 2);
            }
            // 转换格式 0=标准 (生成的PDF图片不会变模糊) 1=最小文件(生成的PDF图片糊的一塌糊涂)
            Dispatch
                .invoke(excel, "ExportAsFixedFormat", Dispatch.Method, new Object[]{new Variant(0),
                    // PDF格式=0
                    pdfFile, new Variant(0)
                }, new int[1]);
            isOpen = true;
            //关闭
            Dispatch.call(excel, "Close", false);
            return true;
        } catch (Exception e) {
            log.error("========Error:文档转换失败：" + e.getMessage(), e);
        } finally {
            if (app != null) {
                app.invoke("Quit", new Variant[]{});
            }
            timer.cancel();
            ComThread.Release();
            ComThread.quitMainSTA();
        }
        return false;
    }
```

这里是excel转pdf。打开文件的方式还是和word一致的，调用**AutomationSecurity**属性为编程方式打开文件安全模式。

再通过
```
Dispatch excel = Dispatch
                .invoke(docs, "Open", Dispatch.Method,
                    new Object[]{inputFile, new Variant(false), new Variant(false)}, new int[9])
                .toDispatch();
```
语句调用**Open**方法，填入参数**new Object[]{inputFile, new Variant(false), new Variant(false)}**，获得工作薄。

因为excel文件会存在多个sheet，所以需要遍历调整每个sheet。
```
//获得一个sheets集合
Dispatch currentSheet = Dispatch.get(excel, "Sheets").toDispatch();
```

**sheets**本身是存在**Count**数量属性，调用get方法即可。
```
Dispatch.get(currentSheet, "Count").getInt()
```

之后可以用**Item**从集合中通过下标检索到一个对象。
```
Dispatch sheetOne = Dispatch.invoke(currentSheet, "Item", Dispatch.Get, new Object[]{new Variant(i)}, new int[1]).getDispatch();
```

业务需要原因，将打印区域取消，纸张横向。但是首先要得到单个sheet的属性才能修改。
```
//获得工作表页面属性设置
                Dispatch PageSetup = Dispatch.get(sheetOne, "PageSetup").toDispatch();
                //更改页面设置为取消
                Dispatch.put(PageSetup, "Zoom", false);
                //更改打印方向为横向（只有1.87类型是excel）
                Dispatch.put(PageSetup, "Orientation", 2);
```

完成属性调整后，就可以正常使用**ExportAsFixedFormat**方法转换了。此处的参数需要看一下文档，0是表示不压缩转换。
```
Dispatch.invoke(excel, "ExportAsFixedFormat", Dispatch.Method, new Object[]{new Variant(0),pdfFile, new Variant(0)}, new int[1]);
```

完成后正常关闭即可。

## 总结
程序员还是需要不断的学习，学习方法对了，可以快速成长。学习方法不对，很有可能原地踏步。