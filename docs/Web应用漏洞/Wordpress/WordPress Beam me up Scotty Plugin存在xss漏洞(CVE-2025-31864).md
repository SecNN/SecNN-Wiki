## WordPress Beam me up Scotty Plugin存在xss漏洞(CVE-2025-31864)

Beam me up Scotty 插件 1.0.23 及以下版本中，由于“返回顶部按钮”自定义设置的数据类型验证和转义处理不足，存在存储型跨站点脚本漏洞。
这些自定义设置只有具有管理员权限的用户才能访问，如果具有管理员权限的攻击者利用此漏洞，则所有显示“返回顶部按钮”的页面的访问者都将面临跨站点脚本攻击。

## fofa

```
body="/wp-content/plugins/web-directory-free"
```

## 第一步
导航到自定义“返回顶部按钮”的菜单（/wp-admin/themes.php?page=beam-me-up-scotty_settings）
![image](https://github.com/user-attachments/assets/14e3a132-b211-484d-8bf7-1c3c7f26904d)

## 第二步
在启用了代理工具（例如BurpSuite）拦截的情况下，点击‘返回顶部按钮’设置菜单底部的‘保存’按钮，即可拦截保存‘返回顶部按钮’自定义设置的请求包。
![image](https://github.com/user-attachments/assets/076678dc-90cd-4847-a337-eff3875f0a65)
将请求payload中payload的值改为beam_me_up_scotty_bottom_indentation如下，然后执行Forward
```
20px;}</style><script>alert("XSS")</script><style>foo{bottom:0
```
![image](https://github.com/user-attachments/assets/f2f67dc8-ff2b-4094-b547-31f53cc94527)

## 第三步
访问WordPress网站后，即可确认存在XSS漏洞。
![image](https://github.com/user-attachments/assets/cf800af5-0d55-46b1-a825-297494d5b18b)


## 漏洞来源
- https://github.com/4m3rr0r/CVE-2025-30208-PoC
