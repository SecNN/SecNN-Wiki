# CRLF Injection

## 0x01 漏洞描述

CRLF是`回车 + 换行`（`\r\n`）的简称。在HTTP协议中，HTTP Header与HTTP Body是用两个CRLF分隔的，浏览器就是根据这两个CRLF来取出HTTP 内容并显示出来。所以，一旦我们能够控制HTTP 消息头中的字符，注入一些恶意的换行，这样我们就能注入一些会话Cookie或者HTML代码，所以CRLF Injection又叫HTTP Response Splitting，简称HRS。

## 0x02 常见应用场景

根据CRLF漏洞原理我们知道，CRLF主要是向HTTP响应头中注入了恶意代码，那么业务场景中往HTTP响应头中写入数据的地方就有可能存在CRLF漏洞，比如最常见的重定向处

```
HTTP/1.1 302 Moved Temporarily
Date: Tue, 24 Dec 2019 11:05:15 GMT
Content-Type: text/html
Connection: close
Location: https://www.baidu.com
```

## 0x03 漏洞危害

1. 通过在响应头中设置一个SESSION，即可造成一个“会话固定漏洞”；
2. 通过向响应头中注入恶意配置，即可造成一个无视浏览器Filter的反射型XSS；
3. 跳转劫持；
4. 通过注入html代码可以进行钓鱼；

## 0x04 修复建议

* 在设置HTTP响应头的代码中，过滤回车、换行（`%0d`，`%0a`，`%0D`，`%0A`)等字符，避免输入的数据污染到其他HTTP头。
* 对参数做合法性校验以及长度限制。