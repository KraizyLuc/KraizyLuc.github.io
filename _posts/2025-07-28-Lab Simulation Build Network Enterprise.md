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


### Step 2: Cài đặt một số công cụ hổ trợ và thêm các thiết bị vào eve-ng
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

## Phần 2: Config kết nối với nhau

### Step 1: Firewall

Kiểm soát truy cập các vùng bằng Firewall

![image](/assets/img/NetworkWithEve/Picture14.png)

Trong lab này sẽ sử dụng Firewall fortigate như là một gateway để quản lý các lưu lượng vào và ra của mạng doanh nghiệp
 
 ==> Và Firewall này cũng đóng vai trò như một router luôn.

![image](/assets/img/NetworkWithEve/Picture15.png)

Config interface port1 để có thể kết nối tới internet: Chỉnh về mode static để tự set ip nằm trong mạng thật của máy >> 192.168.0.110

![image](/assets/img/NetworkWithEve/Picture16.png)

Vào giao diện của Fortigate

![image](/assets/img/NetworkWithEve/Picture17.png)

 Điều chỉnh port1 là Wan cổng mà đi ra ngoài internet 

![image](/assets/img/NetworkWithEve/Picture18.png)

Port2 sẽ là của vùng dmz

![image](/assets/img/NetworkWithEve/Picture19.png)

Tiếp tục đặt port3 là lan và cấp ip là 10.10.20.1/255.255.255.0 và bật dhcp server để tự cấp ip cho máy

![image](/assets/img/NetworkWithEve/Picture20.png)

Đặt một static route như là đường đi cho mạng internal ra ngoài internet với default gateway là của mạng isp và sử dụng trên port1 wan

![image](/assets/img/NetworkWithEve/Picture21.png)

Tạo một policy và cài đặt NAT cho vùng User để user có thể truy cập ra internet bên ngoài

![image](/assets/img/NetworkWithEve/Picture22.png)

Tạo các vlan 10,20 để các máy thông với nhau 

![image](/assets/img/NetworkWithEve/Picture23.png)

Tạo một static route để đi ra mạng internet

![image](/assets/img/NetworkWithEve/Picture24.png)

- Tạo các policy bao gồm:
    - Máy tính user kết nối được internet
    - Máy tính user và domain controller thông lan với nhau để user có thể join domain
    - Nat webserver ra internet để các máy user có thể kết nối tới
    - Vùng internal network có thể kết nối internet

![image](/assets/img/NetworkWithEve/Picture25.png)

### Step 2: Switch

Tạo đường trunk và chuyển mode cho cổng e0/0 của switch user

![image](/assets/img/NetworkWithEve/Picture26.png)

Config cổng e0/1 cho phép truy cập vlan10

![image](/assets/img/NetworkWithEve/Picture27.png)

Tương tự như vậy  config bên vlan 20. Tạo đường trunk cho vlan 10,20 và chuyển mode trunk

![image](/assets/img/NetworkWithEve/Picture28.png)

Chuyển cổng e0/1 sang mode access và cho truy cập vlan 20

![image](/assets/img/NetworkWithEve/Picture29.png)

### Step 3: Máy User

Kiểm tra trong máy user xem đã nhận ip và truy cập internet được chưa

![image](/assets/img/NetworkWithEve/Picture30.png)

==> Đã thấy truy cập được internet

Kiểm tra xem có nat thành công web server bằng cách lấy máy user truy cập vào web 

![image](/assets/img/NetworkWithEve/Picture31.png)

==> Kết nối thành công

2 vùng vlan 10,20 đã thông nhau

![image](/assets/img/NetworkWithEve/Picture32.png)


### Step 4: Máy windows server

Trên windows server vẫn có thể truy cập ra internet bên ngoài

![image](/assets/img/NetworkWithEve/Picture33.png)

Trên máy windows server tạo 2 tài khoản người dùng cho 2 máy user1 và user2

![image](/assets/img/NetworkWithEve/Picture34.png)

Đăng nhập trên máy user1: Password bắt buộc đổi khi lần đầu đăng nhập

![image](/assets/img/NetworkWithEve/Picture35.png)

![image](/assets/img/NetworkWithEve/Picture36.png)

Đăng nhập vào tài khoản domain thành công và ở windows server có thêm vào một máy tính

![image](/assets/img/NetworkWithEve/Picture37.png)

Tương tự bên máy user2 cũng join vào domain

![image](/assets/img/NetworkWithEve/Picture38.png)


## Lời kết

Đây là tất cả bài lab mini của mình khi mô phỏng lại một mô hình mạng của doanh nghiệp vừa và nhỏ. Mình làm bài lab này trong quá trình tìm hiểu các bước đầu tiên để tiến đến gần hơn việc nắm rõ SOC. Nếu có gì sai sót hoặc không đúng logic xin mọi người thông cảm ạ ♥. Hẹn mọi người vào những project sắp tới!!.