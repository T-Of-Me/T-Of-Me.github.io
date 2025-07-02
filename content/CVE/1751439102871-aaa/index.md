---
title: "CVE-2020-30190"
date: 2025-07-02
draft: false
description: "a description"
tags: ["CVE"]
---

# Metasploit Framework 🐱‍👤

# Gainning access

CVE-2020-30190
**Gathering**

```
nmap -sn 192.186.53.0/24
nmap -sS -A 192.168.52.153
```

Dựa vào việc xử lí các URI không đúng cách nên đã thực thi được mã tuỳ ý cụ thể là đoạn này
![image](https://hackmd.io/_uploads/HkRrSIMBle.png)

Auke đây là hướng dẫn về cách sử dụng metasploit demo lại CVE này
![image](https://hackmd.io/_uploads/BJywS8MHgx.png)
![image](https://hackmd.io/_uploads/S1jvBLfBgg.png)
![image](https://hackmd.io/_uploads/HyNOBIzSxe.png)

Sau khi gen được file doc thì bắt đầu launch server lên để lừa nạn nhân tải về
![image](https://hackmd.io/_uploads/HJ0qSUGrll.png)
![image](https://hackmd.io/_uploads/H1rsrUMSll.png)
![image](https://hackmd.io/_uploads/r1asSIMSge.png)
Sau đó nạn nhân tải về và click vào file
![image](https://hackmd.io/_uploads/SJq3rLzBee.png)
![image](https://hackmd.io/_uploads/HJBaHUMBel.png)

# Privilege Registry Key Hijacking

Dùng thêm kĩ thuật bypass uac để leo thang lấy hashdump
![image](https://hackmd.io/_uploads/B19ABUGrll.png)
![image](https://hackmd.io/_uploads/BJvkUUzBeg.png)

Hack tạo 1 register giả sau đó lợi dụng cơ chế tìm kiếm key ưa tiên HKCU . Thực thi payload của mình ở 1 cấp độ cao hơn
![image](https://hackmd.io/_uploads/HJg-L8zSll.png)
![image](https://hackmd.io/_uploads/Sy2-8UfBee.png)
![image](https://hackmd.io/_uploads/ry7GLUMrgx.png)

# Packing Executables

![image](https://hackmd.io/_uploads/BJf7UIfree.png)
![image](https://hackmd.io/_uploads/H13mUIMrgl.png)
![image](https://hackmd.io/_uploads/ByH4ILfHel.png)
