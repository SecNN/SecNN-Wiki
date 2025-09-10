# HTTP Parameter Pollution

## 0x01 漏洞描述

HTTP参数污染漏洞（HTTP Parameter Pollution）简称HPP，由于HTTP协议允许同名参数的存在，同时，后台处理机制对同名参数的处理方式不当，造成“参数污染”。攻击者可以利用此漏洞对网站业务造成攻击，甚至结合其他漏洞，获取服务器数据或获取服务器最高权限。

## 0x02 常见应用场景

```
GET /foo?par=first&par=last HTTP/1.1
User-Agent: Mozilla/5.0
Host: Host
Accept: */*
```

以上面数据包为例，常见的Web服务器对同样名称的参数出现多次的处理方式如下：

|     Web环境      |        参数获取函数         |   获取到的参数   |
| :--------------: | :-------------------------: | :--------------: |
|    PHP/Apache    |        $_GET("par")         |       last       |
|    JSP/Tomcat    | Request.getParameter("par") |      first       |
| Perl(CGI)/Apache |        Param("par")         |      first       |
|  Python/Apache   |       getvalue("par")       | ["first","last"] |
|   ASP.NET/IIS    | Request.QueryString("par")  |    first,last    |

## 0x03 漏洞危害

HTTP 参数污染的风险实际上取决于后端所执行的操作，以及被污染的参数提交到了哪里。

* 对客户端的攻击，比如投票、跳转、关注等；
* 绕过安全防护软件；

## 0x04 修复建议

1. 对用户输入数据的参数的格式进行验证。
2. 在WAF或其他网关设备（比如IPS）在检查URL时，对同一个参数被多次赋值的情况进行特殊处理。
3. 在代码层面，编写WEB程序时，要通过合理的`$_GET`方法获取URL中的参数值，而尝试获取web服务器返回给程序的其他值时要慎重处理。