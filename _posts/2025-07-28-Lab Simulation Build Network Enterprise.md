---

title: Lab mô phỏng Xây Dựng Mạng Doanh Nghiệp vừa và nhỏ Từ A-Z Với EVE-NG
date: 2025/07/28 17:47 +0700
tags: [Network]
categories: [Labs]
author: KraizyLuc
math: true
image: /assets/img/NetworkWithEve/logopost.jpg

---

## Mở đầu:
Trong bài viết này, mình sẽ bắt đầu tìm hiểu và từng bước triển khai mô hình mạng hoàn chỉnh dành cho doanh nghiệp vừa và nhỏ với Firewall, VLAN, Domain Controller, Web Server... bằng cách sử dụng EVE-NG (Công cụ mô phỏng ảo hóa các thiết bị).

## Mục tiêu hệ thống
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

![alt text](/assets/img/NetworkWithEve/model.png)

## Phần 1: Cài đặt EVE-NG và một số thành phần cần thiết

### Step 1: Tải và cài đặt EVE-NG
Tải và cài đặt EVE-NG.

![picture1](/assets/img/NetworkWithEve/Picture1.1.png)

Mình sẽ sử dụng vmware là nơi để triển khai EVE-NG.

![picture2](/assets/img/NetworkWithEve/Picture1.2.png)

Thông số cơ bản cho việc mô phỏng lần này.

![picture3](/assets/img/NetworkWithEve/Picture1.3.png)

Sau khi khởi động EVE-NG, tiến hành setup server và cấp ip cho server

![picture4](/assets/img/NetworkWithEve/Picture1.4.png)

![picture5](/assets/img/NetworkWithEve/Picture1.5.png)

Các bước thiết lập đã xong tiến hành truy cập thông qua web browser bằng ip đã cấp

![picture6](/assets/img/NetworkWithEve/Picture1.6.png)

Để có thể tiếp tục lab chúng ta sẽ cần thêm vào một số thiết bị ảo tương ứng với từng Node 

![picture7](/assets/img/NetworkWithEve/Picture1.7.png)


## Step 2: Cài đặt một số công cụ hổ trợ và thêm các thiết bị vào eve-ng
Tải và cài đặt Windows client side( Có thể giúp kết nối tới giao diện của các thiết bị và có thể kiểm tra bắt gói tin)

![image](/assets/img/NetworkWithEve/Picture2.1.png)

Tải và cài đặt Winscp: có thể dễ dàng trong việc thêm các image thiết bị vào trong eve-ng

![image](/assets/img/NetworkWithEve/Picture2.2.png)

Kết nối tới server và thêm router và switch vào

![image](/assets/img/NetworkWithEve/Picture2.3.png)

![image](/assets/img/NetworkWithEve/Picture2.4.png)

![image](/assets/img/NetworkWithEve/Picture2.5.png)

Sửa lại quyền sở hữu và phân quyền cho toàn bộ file và thư mục chứa image máy ảo trong EVE-NG. Đảm bảo các file thuộc về đúng user chạy web GUI của EVE-NG để có thể tránh lỗi không khởi động được node.

![image](/assets/img/NetworkWithEve/Picture2.6.png)

Thêm máy windows vào

![image](/assets/img/NetworkWithEve/Picture2.7.png)

Thêm máy windows server 2012 và cài đặt dịch vụ active directory domain, dhcp và dns

![image](/assets/img/NetworkWithEve/Picture2.8.png)

Thêm một linux server để làm web server( apache 2)

![image](/assets/img/NetworkWithEve/Picture2.9.png)

![image](/assets/img/NetworkWithEve/Picture2.10.png)

Thêm và cài đặt firewall pfsense  và fortigate

![image](/assets/img/NetworkWithEve/Picture2.11.png)

![image](/assets/img/NetworkWithEve/Picture2.12.png)

![image](/assets/img/NetworkWithEve/Picture2.13.png)