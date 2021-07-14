---
titl![b1](C:\Users\ezio\Desktop\b1.png)e: 当拿到WSDL连接后怎么调用
date: 2021-01-09 18:23:27
tags: WSDL
---

# 当拿到WSDL连接后怎么调用
> 百度参考了一些文章，发现不太适用新版idea。自己来总结一下
> 参考文章：[WebService接口的几种调用方式--wsdl文件类型](https://www.cnblogs.com/javamjh/p/13323096.html)
<!--more--->

## 通过WSDL链接使用idea生成java文件
我的idea版本是

![b1](b1.png)

直接新建项目，选择 WebServices Client，注意：这里的 Version 选择和 WSDL 链接有关。会影响到后面的 b4 图中选项。

![b2](b2.png)

填写好项目名后选择 Finish，

![b3](b3.png)

在窗口中填好参数。

![b4](b4.png)

确定后生成文件，如果产生了报错。从 b2 中换一个 Version 再试一下。

![b5](b5.png)

之后的main方法调用内容，就要看 WSDL 中的方法了。

![b6](b6.png)

## 后面几种方法
我试了一下后面的几种方法，其中调用方法是可以在 idea 生成的文件中看到。但是我本身的wsdl 文件中的调用方法所需参数太多。用起来还没有第一个方便。

### 方式二:使用apache的动态代理方式实现

我自己在使用这个方法的时候，Call类的创建时缺少参数，导致方法调用失败。

```java
import java.net.URL;
import javax.xml.namespace.QName;
import org.apache.axis.client.Call;
import org.apache.axis.client.Service;
public class UsingDII {
    public static void main(String[] args) {
        try {
            int a = 100, b=60;　　　　　　　// 对应的targetNamespace
            String endPoint = "http://198.168.0.88:6888/ormrpc/services/EASLogin";
            Service service = new Service();
            Call call = (Call)service.createCall();
            call.setOperationName(new QName(endPoint,"EASLogin"));
            call.setTargetEndpointAddress(new URL(endPoint));　　　　　　　// a,b 调用此方法的参数
            String result = (String)call.invoke(new Object[]{new Integer(a),new Integer(b)});
            System.out.println("result is :"+result);        
        } catch (Exception e) {
            e.printStackTrace();
        }     
    }
}
```

### 方式三：使用Dynamic Proxy动态代理
```java
import java.net.URL;
import javax.xml.*;
public class UsingDP {
    public static void main(String[] args) {
        try {
            int a = 100, b=60;
            String wsdlUrl = "http://198.168.0.88:6888/ormrpc/services/EASLogin?wsdl";
            String nameSpaceUri = "http://198.168.0.88:6888/ormrpc/services/EASLogin";
            String serviceName = "EASLoginProxyService";
            String portName = "EASLogin";
            ServiceFactory serviceFactory = ServiceFactory.newInstance();        
            Service service = serviceFactory.createService(new URL(wsdlUrl),new QName(nameSpaceUri,serviceName));　　　　　　  // 返回值是自己封装的类
            AddFunctionServiceIntf adsIntf = (AddFunctionServiceIntf)service.getPort(new QName(nameSpaceUri,portName),AddFunctionServiceIntf.class);        
            System.out.println("result is :"+adsIntf.addInt(a, b));       
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

### 方式四:使用httpclient方式
```java
import java.io.ByteArrayInputStream;import java.io.IOException;import java.io.InputStream;import java.util.List;


import org.apache.commons.httpclient.HttpClient;import org.apache.commons.httpclient.methods.InputStreamRequestEntity;import org.apache.commons.httpclient.methods.PostMethod;import org.apache.commons.httpclient.methods.RequestEntity;import org.apache.log4j.Logger;import org.dom4j.Document;import org.dom4j.io.SAXReader;

// 这里引得依赖  包的话需要自己找了 下面地址可以找到//https://mvnrepository.com/
public static InputStream postXmlRequestInputStream(String requestUrl, String xmlData) throws IOException{
        
        PostMethod postMethod = new PostMethod(requestUrl);
        byte[] b = xmlData.getBytes("utf-8");
        InputStream is = new ByteArrayInputStream(b, 0, b.length);
        RequestEntity re = new InputStreamRequestEntity(is, b.length, "text/xml;charset=utf-8");
        postMethod.setRequestEntity(re);
        
        HttpClient httpClient = new HttpClient();
        httpClient.getParams().setAuthenticationPreemptive(true);
        httpClient.getHostConfiguration().setProxy(CommonPptsUtil.get("PROXY_HOST"), Integer.valueOf(CommonPptsUtil.get("PROXY_PORT")));
        
        int statusCode = httpClient.executeMethod(postMethod);
        logger.debug("responseCode:"+statusCode);
        if (statusCode != 200) {
            return null;
        }
        return postMethod.getResponseBodyAsStream();
    }
    
    public static void main(String[] args) {
        String reqJsonStr = "{\"workId\":\"20171018161622\",\"status\":\"201\",\"startTime\":\"2017-10-18 16:16:22\"}";
        
        String xmlData =  "<soapenv:Envelope xmlns:soapenv=\"http://schemas.xmlsoap.org/soap/envelope/\" xmlns:ser=\"http://service.interfacemodule.cic.com/\"><soapenv:Header/><soapenv:Body><ser:statusWriteBack><jsonString>"
                + "{\"workId\":\"314\",\"orderId\":\"5207675\",\"longitude\":\"104.068310\",\"latitude\":\"30.539503\",\"sendTime\":\"2019-08-13 08:38:45\",\"servicePerName\":\"于xx\",\"servicePerPhone\":\"184xxxx7680\"}"
                + "</jsonString></ser:statusWriteBack></soapenv:Body></soapenv:Envelope>";
        String url = "http://xx.xxx.246.88:7103/avs/services/CCService?wsdl";
        SAXReader reader  = new SAXReader();
        String result = "";
        try {
            InputStream in = postXmlRequestInputStream(url,xmlData);
            if(in!=null){
                Document doc = reader.read(in);
                result = doc.getRootElement().element("Body").element("statusWriteBackResponse").element("return").getText();
                logger.debug("result:"+result);
            }
        } catch (Exception e) {
            logger.error("error:",e);
            e.printStackTrace();
        }
    }}
```

CommonPptsUtil：//就是获取配置文件里的代理信息
```java
import org.apache.commons.configuration.ConfigurationException;
import org.apache.commons.configuration.PropertiesConfiguration;
import org.apache.commons.configuration.reloading.FileChangedReloadingStrategy;
import org.apache.log4j.Logger;

/**
 * 通用属性文件工具类
 * 
 * @author y.c
 * 
 */
public class CommonPptsUtil {

    private static final Logger logger = Logger.getLogger(CommonPptsUtil.class);

    private static final String CONFIG_FILE = "common.properties";

    private static PropertiesConfiguration ppts;

    static {
        try {
            ppts = new PropertiesConfiguration(CONFIG_FILE);
            ppts.setReloadingStrategy(new FileChangedReloadingStrategy());
        } catch (ConfigurationException e) {
            logger.error("文件【common.properties】加载失败！");
            ppts = null;
        }
    }

    /**
     * 获取属性值
     * 
     * @param key
     * @return 属性值
     */
    public static String get(String key) {
        if (ppts == null) {
            return null;
        }

        return ppts.getString(key);
    }
}

```


### 方式五:使用CXF动态调用webservice接口
```java
package cxfClient;
  
import org.apache.cxf.endpoint.Endpoint;
import javax.xml.namespace.QName; 
import org.apache.cxf.jaxws.endpoint.dynamic.JaxWsDynamicClientFactory;
import org.apache.cxf.service.model.BindingInfo;
import org.apache.cxf.service.model.BindingOperationInfo;
  
public class CxfClient {
  
    public static void main(String[] args) throws Exception {
        String url = "http://localhost:9091/Service/SayHello?wsdl";
        String method = "say";
        Object[] parameters = new Object[]{"我是参数"};
        System.out.println(invokeRemoteMethod(url, method, parameters)[0]);
    }
     
    public static Object[] invokeRemoteMethod(String url, String operation, Object[] parameters){
        JaxWsDynamicClientFactory dcf = JaxWsDynamicClientFactory.newInstance();
        if (!url.endsWith("wsdl")) {
            url += "?wsdl";
        }
        org.apache.cxf.endpoint.Client client = dcf.createClient(url);
        //处理webService接口和实现类namespace不同的情况，CXF动态客户端在处理此问题时，会报No operation was found with the name的异常
        Endpoint endpoint = client.getEndpoint();
        QName opName = new QName(endpoint.getService().getName().getNamespaceURI(),operation);
        BindingInfo bindingInfo= endpoint.getEndpointInfo().getBinding();
        if(bindingInfo.getOperation(opName) == null){
            for(BindingOperationInfo operationInfo : bindingInfo.getOperations()){
                if(operation.equals(operationInfo.getName().getLocalPart())){
                    opName = operationInfo.getName();
                    break;
                }
            }
        }
        Object[] res = null;
        try {
            res = client.invoke(opName, parameters);
        } catch (Exception e) {
            e.printStackTrace();
        }
        return res;
    }  
}
```

