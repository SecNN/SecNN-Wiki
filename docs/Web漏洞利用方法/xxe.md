# XXE

## 0x01 漏洞描述

XML 指可扩展标记语言（eXtensible Markup Language），是一种用于标记电子文件使其具有结构性的标记语言，被设计用来传输和存储数据。XML文档结构包括XML声明、DTD文档类型定义（可选）、文档元素。目前，XML文件作为配置文件（Spring、Struts2等）、文档结构说明文件（PDF、RSS等）、图片格式文件（SVG header）应用比较广泛。 XML 的语法规范由 DTD （Document Type Definition）来进行控制。

XML外部实体注入（XML External Entity Injection）漏洞是指当恶意用户在提交一个精心构造的包含外部实体引用的XML文档给未正确配置的XML解析器处理时，该攻击就会发生。

## 0x02 常见应用场景

XXE可能出现在一切可接收XML文档的业务逻辑处。

> SOAP型Web Service。SOAP型的Web Service允许我们使用XML格式与服务器进行通信。
>
> REST型Web Service。REST型Web Service允许我们使用JSON格式（也可以使用XML格式）与服务器进行通信。与HTTP类似，该类型服务支持GET、POST、PUT、DELETE方法。
>
> WSDL给出了SOAP型Web Service的基本定义，WSDL基于XML语言，描述了与服务交互的基本元素，比如函数、数据类型、功能等，少数情况下，WSDL也可以用来描述REST型Web Service。
>
> WADL就像是WSDL的REST版，一般用于REST型Web Service，描述与Web Service进行交互的基本元素。

比如：采用RESTful架构风格的终端通常都是以JSON作为传输格式，但是许多服务端开发框架也允许基于数据交换的XML格式作为输入，可以通过尝试将 Content-Type修改为application/xml，text/xml进行验证。

另外，采用xml的java服务中间件，比如spring，struts2等也可能出现XXE漏洞

## 0x03 漏洞危害

1. 利用 XXE 进行 DOS 攻击；
2. 读取本地任意敏感文件；
3. 利用相关协议进行SSRF探测内网主机IP、端口等信息；
4. 在特定条件下利用Java中的` jar://` 协议，php 中的 `phar://`可能导致恶意文件上传；
5. PHP中如果安装了expect扩展，可利用XXE执行任意命令

## 0x04 修复建议

### 4.1 使用语言中推荐的禁用外部实体的方法

PHP：

```php
libxml_disable_entity_loader(true);
```

Java:

```java
DocumentBuilderFactory dbf =DocumentBuilderFactory.newInstance();
dbf.setExpandEntityReferences(false);
```

Python:

```python
from lxml import etree
xmlData = etree.parse(xmlSource,etree.XMLParser(resolve_entities=False))
```

### 4.2 手动黑名单过滤

过滤XML中的相关关键词，比如：`<!DOCTYPE`、`<!ENTITY SYSTEM`、`PUBLIC`等。
