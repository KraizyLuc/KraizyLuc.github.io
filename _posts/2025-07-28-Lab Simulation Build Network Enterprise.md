---

title: Lab mô phỏng Xây Dựng Mạng Doanh Nghiệp vừa và nhỏ Từ A-Z Với EVE-NG
date: 2025/07/28 17:47 +0700
tags: [Network]
categories: [Labs]
author: KraizyLuc
math: true
img_path: /assets/img/NetworkWithEve
image: logopost.jpg

---

# Mở đầu:
Trong bài viết này, mình sẽ bắt đầu tìm hiểu và từng bước triển khai mô hình mạng hoàn chỉnh dành cho doanh nghiệp vừa và nhỏ với Firewall, VLAN, Domain Controller, Web Server... bằng cách sử dụng EVE-NG (Công cụ mô phỏng ảo hóa các thiết bị).

# Mục tiêu hệ thống
> Do tài nguyên máy không đủ nên mình sẽ sử dụng các thiết bị hơi cũ nhưng vẫn đảm bảo đầy đủ tính năng để có thể hoạt động một cách ổn định.
1. Phân vùng mạng rõ ràng: DMZ, Internal, User, Internet
2. Triển khai dịch vụ:
    - Web Server (Linux + Apache)
    - Active Directory Domain Controller (Windows Server 2012R2)
    - 2 máy người dùng join domain
3. Kiểm soát truy cập:
    - NAT cho phép user ra Internet
    - NAT Web Server ra ngoài (port 80/443)
    - Firewall quản lý luồng dữ liệu giữa các zone

## Ý tưởng cho mô hình xây dựng

![model](model.png)

# Phần 1: Cài đặt EVE-NG và một số thành phần cần thiết

## Step 1: Tải và cài đặt EVE-NG
Tải và cài đặt EVE-NG.

![picture1](Picture1.1.png)

Mình sẽ sử dụng vmware là nơi để triển khai EVE-NG.

![picture2](Picture1.2.png)

Thông số cơ bản cho việc mô phỏng lần này.

![picture3](Picture1.3.png)

Sau khi khởi động EVE-NG, tiến hành setup server và cấp ip cho server

![picture4](Picture1.4.png)

![picture5](Picture1.5.png)

Các bước thiết lập đã xong tiến hành truy cập thông qua web browser bằng ip đã cấp

![picture6](Picture1.6.png)

Để có thể tiếp tục lab chúng ta sẽ cần thêm vào một số thiết bị ảo tương ứng với từng Node 

![picture7](Picture1.7.png)
