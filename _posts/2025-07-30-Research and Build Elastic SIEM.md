---
title: Dự án tìm hiểu và xây dựng Elastic SIEM (2024)
date: 2025/07/30 18:07 +0700
tags: [SIEM, Elastic, ELK, Research, Building]
categories: [Personal Project]
author: KraizyLuc
math: true
image: https://threatconnect.com/wp-content/uploads/2022/11/elastic-logo.jpg
---



> Xin Chào mọi người, mình là Nhật Tiến. trong bài blog này mình sẽ thực hiện một dự án triển khai Elastic SIEM-Một giải pháp dựa trên ELK stack. Đây là một dự án nhỏ mà mình vừa nghiên cứu vừa triển khai trong khi thực tập SOC năm 2024. Mong mọi người xem vui vẻ.

# Dự án tìm hiểu và xây dựng Elastic SIEM 

## Phần 1: Tìm hiểu về ELK Stack và Elastic SIEM

### 1. Định nghĩa

**ELK stack** là một tập hợp các công cụ mã nguồn mở, được thiết kế để tìm kiếm, phân tích và trực quan hóa khối lượng lớn dữ liệu trong thời gian thực, ELK viết tắt của Elasticsearch, Logstash, Kibana và đó là 3 thành phần cốt lõi của ELK ngoài ra gần đây còn có thêm Beats.

- **Elasticsearch**: Công cụ lưu trữ và truy vấn dữ liệu phi cấu trúc theo thời gian thực.
- **Logstash**: Công cụ thu thập, xử lý và chuyển đổi log từ nhiều nguồn.
- **Kibana**: Giao diện trực quan hóa dữ liệu.
- **Beats**: Bộ công cụ nhẹ để thu thập dữ liệu từ hệ thống, như Filebeat, Metricbeat, Packetbeat...

**Cơ chế hoạt động**
- **Thu thập dữ liệu**: Beats hoặc Logstash lấy log từ hệ điều hành, ứng dụng, network.
- **Xử lý**: Logstash xử lý log, gắn thẻ, định dạng rồi chuyển đến Elasticsearch.
- **Truy vấn và hiển thị**: Kibana giúp người dùng tìm kiếm, hiển thị log và xây dựng dashboard.

![image](https://hackmd.io/_uploads/SkMSxwDwle.png)

**Elastic SIEM** (Security Information and Event Management) là giải pháp được xây dựng từ ELK Stack nhưng tích hợp thêm nhiều tính năng chuyên biệt phục vụ cho an ninh mạng:
- **Phân tích sự kiện bảo mật**: Đăng nhập bất thường, tấn công mạng, hành vi nghi vấn.
- **Phát hiện xâm nhập (Detection Rules)**: Tự động hóa giám sát và cảnh báo.
- **Tích hợp dữ liệu threat intelligence**: AlienVault OTX, Abuse.ch...
- **Dashboard trực quan hóa bảo mật**: Tổng hợp sự kiện, mức độ nghiêm trọng, nguồn gốc...

## Phần 2: Xây dựng Elastic SIEM

**Mô hình ví dụ build trong lab này**

![image](https://hackmd.io/_uploads/BkcaWDvvee.png)


### Step 1: Cài đặt và cấu hình các thành phần cơ bản cho ELK Stack

**1. Chuẩn bị một máy vm ubuntu 18.04 live server để tích hợp ELK Stack**
    IP: 192.168.117.129
![image](https://hackmd.io/_uploads/ryS4MDDweg.png)
- Update và tải những thành phần cần thiết
```
    sudo apt update && sudo apt upgrade -y
    sudo apt dist-upgrade -y
    sudo apt install zip unzip -y
    sudo apt install jq -y
```

**2. Cấu hình bảo mật cho apt và cài đặt ELK Stack**

Tải apt-transport-https để kich hoạt tsl/ssl-encrypted có tác dụng thiết lập các kết nối an toàn với các máy chủ khi tải dữ liệu repositories

![image](https://hackmd.io/_uploads/Sk9iMwDDeg.png) 

Thêm repository của Elastic Stack vào trong linux. Trong đó khóa GPG được thêm vào xác minh tính toàn vẹn của các gói cài đặt. 

```li=
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add  
echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-7.x.list
```
![image](https://hackmd.io/_uploads/SkFdmDwPeg.png)

=> Từ đây chúng ta sẽ có thể tải các thành phần từ Elastic một cách an toàn.

Cập nhật lại repo trên linux và cài đặt elasticsearch và xác nhận xem đã chạy mượt chưa

![image](https://hackmd.io/_uploads/Sk_6mDPPgx.png)

![image](https://hackmd.io/_uploads/SJi67Pwwxg.png)

=> Trong trường hợp này biết được rằng elastic đã được cài thành công nhưng chưa có active trong systemctl

**3. Cấu hình file yml để elasticsearch biết cần mở những port nào hay giao tiếp ở đâu.**

Truy cập file elasticsearch.yml để điều chỉnh một số cần thiết:
 - Thay đổi cluster name
 `cluster.name : elk-lab`
 ![image](https://hackmd.io/_uploads/SkTXNwPvgg.png)
 - Thêm địa chỉ ip và port để elasticsearch giao tiếp
 ```
 network.host: 192.168.117.129
 http.port: 9200
```
 ![image](https://hackmd.io/_uploads/Hyp4NDPPge.png)
 - Chạy trên chế độ **single-node**( vì đây là lab mang tính nghiên cứu nên mình chỉ cần chạy ở chế độ độc lập và không cần cài đặt một cluster hoàn chỉnh).
 ![image](https://hackmd.io/_uploads/rJxDNvDPll.png)
 
 Khởi động elasticsearch
 ![image](https://hackmd.io/_uploads/ByU_EDPPee.png)
Kiểm tra xem elastic đã chạy đúng với ip và port chưa.![image](https://hackmd.io/_uploads/HkauVDwwxx.png)
==> Chúng ta thấy elastic phản hồi và với tagline: “You Know, for Search” thì có nghĩa service elasticsearch đang chạy ổn định trên port 9200 và port này cũng là cầu nối cho giữa Beat shipper/agents khi muốn gửi dữ liệu tới database của elasticsearch cho việc lập index.

**4. Cài đặt Kibana và config file yml**

![image](https://hackmd.io/_uploads/BkFXHPwwxx.png)

Cài đặt ip và port  và thêm tên server
```
server port: 5601
server.host: 192.168.117.129
server.name: elk-lab
```
![image](https://hackmd.io/_uploads/Bk1UHDPDge.png)
![image](https://hackmd.io/_uploads/SyMLrPDDgx.png)

Start kibana và kiểm tra có hoạt động chưa

![image](https://hackmd.io/_uploads/SkUsHvwDge.png)
![image](https://hackmd.io/_uploads/rk5sSvvPlx.png)

> [Trong phạm vi project này mình sẽ không sử dụng tới logstash]

**5. Cài đặt Filebeat**

Tải Filebeat
![image](https://hackmd.io/_uploads/Sk4lUwwvgg.png)
Config filebeat để kết nối tới dịch vụ elasticsearch
`hosts: ["192.168.117.129:9200"]`
![image](https://hackmd.io/_uploads/Sy2gIwDPlg.png)

==> Các bước cài đặt cơ bản đã hoàn thành. Bây giờ cần test để đảm bảo rằng filebeat đang thu thập log và gửi để lập index

```w!
sudo filebeat setup --index-management -E output.logstash.enabled=false 'output.elasticsearch.hosts=["192.168.1.150:9200"]' 
```

![image](https://hackmd.io/_uploads/B16GDPvPxx.png)
=> Lệnh này thông báo filebeat rằng không cần log từ logstash và đảm bảo chúng ta cấu hình đúng máy chủ trong yml file.

Khởi động dịch vụ filebeat

![image](https://hackmd.io/_uploads/BkySvDvDxg.png)

Kiểm tra xem các chỉ mục có hoạt động bình thường không

![image](https://hackmd.io/_uploads/HkLBDPwPee.png)

Kiểm tra xem các chỉ mục có hoạt động bình thường không

![image](https://hackmd.io/_uploads/B1NIwvwDgg.png)

- Ta có thể thấy tình trạng của filebeat đang màu vàng và chưa có document nào được chuyển đi. 
    > Hệ thống ELK stack hiện tại chưa có hệ thống bảo mật và chưa là một hệ thống hoàn chỉnh. Tiếp theo ta sẽ thêm xác thực bằng certificates( http -> https). Tuy nhiên certificate ở đây sẽ là tự cấp không phải của một tổ chức nào. 

**6. Tạo cơ quan cấp chứng chỉ( CA Certificates): tạo một file ==instaces.yml== bao gồm các dịch vụ và ip máy chủ để liên kết certificates.**

![image](https://hackmd.io/_uploads/ryz2vvwDex.png)

Tạo cơ quan cấp chứng chỉ bằng ++elasticsearch-certutil++

![image](https://hackmd.io/_uploads/S1j2wPPPle.png)
![image](https://hackmd.io/_uploads/HJAhPwvDxe.png)

==> Bây giờ ta đã có tệp ca/, bao gồm một tệp chứng chỉ và một khóa phù hợp cho chứng chỉ
==> Tạo chứng chỉ ==x.509== cho các instances vừa tạo trong file ==instances.yml== và trả kết quả ra file certs.zip

```w!
sudo /usr/share/elasticsearch/bin/elasticsearch-certutil cert --ca-cert ca/ca.crt --ca-key ca/ca.key --pem --in instances.yml --out certs.zip
```

![image](https://hackmd.io/_uploads/Sy8K_Dvwgx.png)
Giải nén chứng chỉ và di chuyển các thư mục vào certs![image](https://hackmd.io/_uploads/HJycdwPwgg.png)
```
• sudo mv /usr/share/elasticsearch/elasticsearch/* certs/
• sudo mv /usr/share/elasticsearch/kibana/* certs/
• sudo mv /usr/share/elasticsearch/logstash/* certs/
• sudo mv /usr/share/elasticsearch/fleet/* certs/
```

Tiếp theo ta cần chuẩn bị 2 thư mục để chứa từng chứng chỉ được public và chứng chỉ chuyên cho từng dịch vụ 
```
• sudo mkdir -p /etc/kibana/certs/ca
• sudo mkdir -p /etc/elasticsearch/certs/ca
• sudo mkdir -p /etc/logstash/certs/ca
• sudo mkdir -p /etc/fleet/certs/ca
```

![image](https://hackmd.io/_uploads/HykUYwwvge.png)
Copy những chứng chỉ public
```
• sudo cp ca/ca.* /etc/kibana/certs/ca
• sudo cp ca/ca.* /etc/elasticsearch/certs/ca
• sudo cp ca/ca.* /etc/logstash/certs/ca
• sudo cp ca/ca.* /etc/fleet/certs/ca
```

![image](https://hackmd.io/_uploads/SkmPFDDPex.png)

Tương tự copy những chứng chỉ chuyên dụng 
```
• sudo cp certs/elasticsearch.* /etc/elasticsearch/certs/
• sudo cp certs/kibana.* /etc/kibana/certs/
• sudo cp certs/logstash.* /etc/logstash/certs/
• sudo cp certs/fleet.* /etc/fleet/certs/
```

![image](https://hackmd.io/_uploads/Sy1N9vwwel.png)

### Step 2: Cấu hình bảo mật cho hệ thống

**1. Cấu hình bảo mật cho hệ thống**
- Khi chúng ta cài đặt elasticsearch, logstash và kibana thì sẽ có một user được tạo ra cho mỗi dịch vụ đó.
- Trong bước này chúng ta sẽ phân bổ quyền hợp lý để góp phần củng cố bảo mật cho hệ thống ELK Stack.
- Chỉnh sửa quyền truy cập cho tất cả file elasticsearch => tương tự với kibana và logstash

![image](https://hackmd.io/_uploads/BJo1jwDPxe.png)

**2. Cấu hình dịch vụ**
- Các bước trước ta đã cấu hình chứng chỉ để xác minh danh tính cho dịch vụ
- Tiếp theo cần cấu hình cho các dịch vụ để chỉ ra các file thích hợp để sử dụng chúng nhằm giao tiếp và thực hiện công việc thông qua https
- Chỉnh sửa kibana yaml file

![image](https://hackmd.io/_uploads/Hyf-oDPvex.png)

Tương tự với elasticsearch

![image](https://hackmd.io/_uploads/B1sAsDvDgl.png)

=> Khởi động lại elasticsearch và kibana để áp dụng các thay đổi mới

Kiểm tra xem elasticsearch và kibana đã nhận chứng chỉ và nâng lên https hay chưa

![image](https://hackmd.io/_uploads/rJcZ3PvPge.png)

**3. Tạo password**

Để tạo những password cho user và tài khoản dịch vụ ta dùng elasticsearch-setup-password 

![image](https://hackmd.io/_uploads/H1VaavvDxl.png)
==> Config file kibana.yml để thêm user kibana-system được tạo ra và nó có thể tương tác với elasticsearch


Truy cập vào elastic với tài khoản elastic được tạo ra

![image](https://hackmd.io/_uploads/ry8kCDwvel.png)

### Step 3: Nhập dữ liệu với Agent và Fleet

Hiện tại ELK Stack đã chạy và có thể tương tác được. Tuy nhiên vẫn chưa có logs nào được gửi đến vì vậy các bước tiếp theo sẽ làm việc với Filebeat để có thể chạy trên https và có thể nhờ các agent gửi log đến ELK Stack cho chúng ta.

chúng ta có thể quản lý các agent đã được triển khai bằng dịch vụ Fleet

**1. Tạo một server Fleet**

![image](https://hackmd.io/_uploads/r1tGRvPDlg.png)

Set-up Fleet
```!
Select an Agent Policy
-------------------------------------
> Agent Policy: Default Fleet Server Policy

Choose a Deployment Mode for Security
-------------------------------------
> Production - Provide your own certificates. This option will require agents to specify a cert key when enrolling with Fleet.

Add your Fleet Server Host
-------------------------------------
> https://192.168.117.129:8220
Generate a service token 
-------------------------------------
Redacted
```

Quay trở về server ubuntu và tiến hành tải và cài đặt elastic-agent 

![image](https://hackmd.io/_uploads/S12vkdwDxg.png)

==> Chạy theo trình tự trong fleet set up để khởi động server

Kiểm tra xem Fleet đã trên đúng host chưa và đảm bảo cả elasticsearch nữa.

![image](https://hackmd.io/_uploads/rJDqJODwgx.png)

**2. Tạo chính sách cho Agent**

- Vì sao phải cần tạo Fleet?
  - Cung cấp một nơi để quản lý tập trung tất cả các agent và những log mà những agent đó đang thu thập
  - Tương tự như c2 server mà dành cho thu thập log
  - Để chỉ định các log cụ thể mà chúng ta muốn thu thập ta cần tạo chính sách

Ví dụ muốn thu thập log của router hay của máy windows thì sẽ cần thu thập rất nhiều loại logs khác nhau. Để giải quyết vấn đề này ta sẽ tạo 2 bộ chính sách cho windows và router. Sau khi cài agent trên endpoint mà ta đang giám sát. Ta sẽ nói cho các Agent biết cần đi theo chính sách nào.

![image](https://hackmd.io/_uploads/Hk1fl_DPxl.png)

Bây giờ ta sẽ thêm những cái tích hợp(integrations) vào chính sách. Integrations sẽ nói cho những agent biết là loại log nào sẽ cần thu thập và từ đâu. Khi áp dụng những tích hợp này vào những chính sách và đưa agent vào một máy nào đó, nó sẽ kết nối tới Fleet server để xác định tự cấu hình như thế nào cho việc thu thập log và những thay đổi.

- Bắt đầu bằng việc thêm EDR vào chính sách bằng integrations tên Endpoint Security: EDR(Endpoint Detection & response): là hệ thống phát hiện và phản hồi các mối nguy hại tại endpoint
![image](https://hackmd.io/_uploads/HyrJbuwPgl.png)
- Thêm windows integrations
  - Cho phép giám sát hệ điều hành windows, dịch vụ, ứng dụng,…
  - Thu thập log từ windows machine

![image](https://hackmd.io/_uploads/HyDFWdPPgx.png)

**3. Pre-config Endpoint**

Chuyển sang máy mà ta sẽ theo dõi
Kiểm tra xem là máy này có thể kết nối tới elk stack hay không
![image](https://hackmd.io/_uploads/Sy7pWuDDxg.png)
Trạng thái kết nối rất tốt

- Tiếp theo cần cài đặt Sysmon ở trên Endpoint
- Sysmon là một công cụ giúp giám sát và phân loại những gì đang xảy ra trong máy windows
- Bắt đầu tải xuống tệp thực thi sysmon

![image](https://hackmd.io/_uploads/H1WGGdwPge.png)

Tiếp theo cần set-up agent và config sysmon. Có một template khá hay trong sysmon-config là SwiftOnSecurity

![image](https://hackmd.io/_uploads/rkYQMuPvgg.png)

Đảm bảo rằng  PowerShell Script block logging được bật. Đó là tính năng cho phép ghi lại mọi tập lệnh đang chạy trên máy thông qua PowerShell

![image](https://hackmd.io/_uploads/rJSNGdPPll.png)

Điều này sẽ kiểm tra xem registry key có tồn tại để ghi log khối tập lệnh hay không. Nếu không, nó sẽ tạo khóa và đặt giá trị của nó thành giá trị thực boolean của “1”.

4. Áp dụng Agents

Bây giờ máy windows đã sẵn sàng để ghi log và cài đặt elastic-agent nhưng có một vấn đề là máy windows này lại không tin vào cơ quan cấp chứng chỉ của dịch vụ.
==> Thêm chứng chỉ đó vào như là một chứng chỉ đáng tin cậy

- Windows
![image](https://hackmd.io/_uploads/SyCYzuvwxx.png)

- Linux
![image](https://hackmd.io/_uploads/Bka9zOPPgl.png)

5. Tải xuống elastic-agent và kêu chúng kết nối tới fleet server

Windows elastic agent
![image](https://hackmd.io/_uploads/BJ5jfOvPlx.png)

Linux elastic agent
![image](https://hackmd.io/_uploads/HJZ2fdvvlg.png)

Sau khi tải xuống tệp thực thi, quay lại Giao diện web Kibana sử dụng lệnh đăng ký agent tới Fleet server.
![image](https://hackmd.io/_uploads/ry2nMuDDxx.png)

==> Làm theo hướng dẫn và bây giờ trong Fleet sẽ thấy agent đang chạy trong máy windows và dưới chính sách Windows Endpoints.

![image](https://hackmd.io/_uploads/SywAzODPxx.png)

Vô phần Discover ta sẽ thấy các log đang dần được agent gửi tới và được elasticsearch lưu trữ trong database.

![image](https://hackmd.io/_uploads/HJe1XuwPxx.png)

### Step 4: Kích hoạt và cấu hình Threat Intelligence và Detections

> Tổng hợp các bước trước: Ta đã có thể tổng hợp được các logs và triển khai một số bảo vệ endpoint trên host machine. Thay vì việc tìm kiếm thủ công các logs hiệu quả trong threat hunting/malware reverse/ IR nhưng ta có thể bật một số threat intelligence và rules detection trong stack. Điều này giúp có những thông tin mới nhất từ cộng đồng giúp cho SIEM và cũng như tạo cảnh báo.

**1. Kích hoạt Filebeat module và cập nhật chứng chỉ**

- Config filebeat để có thể vận chuyển một số dữ liệu đến stack
- Đầu tiên kích hoạt module system và threatintel
  - System: giúp thu thập và phân tích các log hệ thống từ nhiều nguồn(hệ điều hành, dịch vụ, cpu,bộ nhớ, đĩa cứng)
  - Threatintel: Giúp thu thập và phân tích các mối đe dọa an ninh từ nhiều nguồn( danh sách malicious ip/domain, thông tin mối đe dọa từ cộng đồng.
![image](https://hackmd.io/_uploads/SJeCNmdwPgg.png)

**2. Tạo API key và config filebeat**

- Sử dụng Dev tool console để tương tác với ELK Stack API để:
```
* Tạo role filebeat_writer
* Cung cấp các quyền cần thiết để role đó có thể truy cập cluster
* Cho role quyền truy cập tới chỉ mục filebeat-* nới chứa các log từ filebeat
* Tạo khóa API để sử dụng thông tin xác thực nhằm tận dụng những quyền đó
```

![image](https://hackmd.io/_uploads/ByssmODwge.png)

Đây là nơi mà có thể tương tác với các Endpoint khác nhau để yêu cầu hoặc gửi các dữ liệu khác nhau. Ví dụ có thể sử dụng nó để truy vấn thông tin, chẳng hạn như “Người dùng nào có quyền truy cập vào elasticstack?” hoặc có thể sử dụng nó để gửi/cập nhật thông tin. Ta sẽ sử dụng nó để gửi thông tin đến máy chủ thông qua POST request nhằm tạo vai trò và tạo khóa API.

![image](https://hackmd.io/_uploads/H1a1SOPDgx.png)

- Config ++filebeat yaml++
- Config để cho filebeat biết cách tiếp cận kibana và elasticsearch thông qua https với chứng chỉ phù hợp.  Và thêm thông tin id, key vừa mới tạo.

![image](https://hackmd.io/_uploads/H1B_r_wwgx.png)

![Picture3](https://hackmd.io/_uploads/HkKJ8uDwee.png)

**3. Thêm nguồn cung cấp dữ liệu Threat intelligece AlienVault OTX**

Mở file threatintel yaml và những nguồn mở và miễn phí như: OTX, abuseurl, abusemalware, malwarebazaar 

![image](https://hackmd.io/_uploads/S1RfLODveg.png)

==> abuseurl, abusemalware và malware Bazaar đều thuộc công ty abuse.ch và không yêu cầu khóa API hoặc tài khoản cá nhân. Tuy nhiên, sẽ cần tạo một tài khoản OTX Alienvault nếu muốn lấy dữ liệu threat intelligence của họ.

Tạo otx key và nhập vào api key trong mục otx trong file threatintel.yml

![image](https://hackmd.io/_uploads/SkPNI_Pwxe.png)

Quay lại giao diện kibana vào phần security overview sẽ thấy một số newfeeds về threat intelligence được thêm vào và bao gồm cả log otx

![image](https://hackmd.io/_uploads/ByrS8dwwxx.png)

**4. Thiết lập Dashboards**

- Dashboards rất hữu ích trong việc phân tích nhanh chóng. 
- Ta sẽ thêm vào một số pre-built threat intelligence dashboard. Để làm được thì cần chạy dưới quyền kibana_admin. 

==> Thay đổi thông tin đăng nhập từ api sang username/password để có thể tạo dashboard

![image](https://hackmd.io/_uploads/SkzOUdPDxg.png)

Restart dịch vụ và chạy để có thể áp dụng dashboards
![image](https://hackmd.io/_uploads/SkMYIuPDgl.png)

Sau khi xong thì ta đưa cách xác thực về api
![image](https://hackmd.io/_uploads/r1OFIOvwgg.png)

==> Cuối cùng restart filebeat và vào dashboards trong giao diện kibana để tìm filebeat threatintel hoặc bất kỳ dashboards nào để xem tổng quan về phát hiện

![image](https://hackmd.io/_uploads/SkHcI_wwge.png)

**5. Kích hoạt các quy tắc được xây dựng sẵn**

- Để có thể lọc ra những cảnh báo độc hại ta có thể sử dụng ELK’s pre-built detection rules
- những quy tắc này có thể làm giảm hiệu suất của rất nhiều. Về cơ bản, các quy tắc này là bộ lọc cho các logs đến. Chúng là các truy vấn riêng lẻ mà mỗi nhật ký được sàng lọc để kiểm tra xem có bất kỳ kết quả trùng khớp nào không và thông báo. Nếu bật 300 quy tắc thì có thể có 300 truy vấn đang được chạy trên mỗi nhật ký khi nhật ký đó được nhập và lập chỉ mục. Một số giải pháp giúp giảm bớt gánh nặng này ở cấp Agent bằng cách lọc trước/gắn thẻ nhật ký. Việc này có thể linh hoạt ở một mức độ nào đó nhưng, nó có thể rất đáng kể và có thể yêu cầu mở rộng quy mô trong môi trường hệ thống.
- Thêm Prebuilt detection rules vào agent policy Windows Endpoints

![image](https://hackmd.io/_uploads/rJ7n8uwwll.png)

Sau khi thêm xong vào phần Security -> rules để có thể áp dụng các rules theo ý của người dùng.
![image](https://hackmd.io/_uploads/HkTnUuDPlx.png)

Ở đây áp dụng các rules trong tag Windows và host
![image](https://hackmd.io/_uploads/Bk1RLODPgl.png)

**6. Tạo quy tắc phát hiện dựa trên dữ liệu threat intel**

- Trong phần cuối ta sẽ kết nối 2 nguồn dữ liệu: dữ liệu từ các logs và dữ liệu từ threat intelligence để cảnh báo IOC bất cứ khi nào xuất hiện trong mạng chúng ta. Đây là một việc hữu ích trong việc phát hiện và ứng phó sớm sự cố và nó cũng giúp tự động hóa một phần. 
- Để làm được việc đó cần biết dữ liệu nào cần thêm vào ELK stack. Ví dụ một số IOC đơn giản như:
  - Ips
  - Urls
  - File hashes
- Tìm một số dữ liệu quan tâm dựa trên các IOC đó. Vào phần discover trong kibana
![image](https://hackmd.io/_uploads/ryuSDuPwgx.png)

Tiếp theo vào phần rules để bắt đầu tạo rule indicator match. Trong phần này ta sẽ tùy chỉnh trong mục log-* nơi chứa destination.ip

![image](https://hackmd.io/_uploads/rk9Lv_Dvel.png)

==> Nó sẽ tạo ra cảnh báo mối đe dọa ip nếu trùng với ip đích trong phần log.
- Tiếp theo ta sẽ chỉ định tên, mô tả và mức độ nghiêm trọng của cảnh báo. Bởi vì cái này cảnh báo trực tiếp các IOC từ các threat intel nên để mức độ priority cao một tí. 
![image](https://hackmd.io/_uploads/HkLDDuvvxg.png)

Cuối cùng là tùy chỉnh hành động khi rule được thực thi
![image](https://hackmd.io/_uploads/HkVYPuPwgg.png)

## Phần 3: Thực hành với Elastic SIEM

> Trong phần này về cơ bản thì các thành phần Elastic SIEM không thay đổi nhưng mô hình lab này sẽ thay đổi đôi chút

![image](https://hackmd.io/_uploads/Sk50p9Ovgg.png)

Các thành phần
Wan: Internet
LAN: gồm 3 máy
1) 2 máy để config và là nguồn lấy logs đẩy lên elastic SIEM
   - 1 máy có hệ thống elastic SIEM được config hoàn chỉnh
2) 1 tường lửa pfSense: 
   - Dùng để kiểm soát lưu lượng mạng
   - Cho mạng lan kết nối ra ngoài internet
   - Đưa log lên SIEM

### Tích hợp pfSense vào elastic SIEM
Ở trên Elastic SIEM cài integration pfSense vào Default fleet server policy tạo từ trước

![image](https://hackmd.io/_uploads/B1TDAcOPgg.png)

Cho syslog host chạy trên ==0.0.0.0== để có thể nghe hết các lưu lượng và port là ==9001== 
![image](https://hackmd.io/_uploads/HJNjC5dvll.png)

Bên pfSense ta sẽ vào phần `status => system logs => setting => tích vào remote logging` 
![image](https://hackmd.io/_uploads/SJAhCq_Dxe.png)
![image](https://hackmd.io/_uploads/H1EaAquwgl.png)

> Tuy nhiên nếu cài đặt thông thường thì pfsense sẽ chỉ đảy log metric mà thôi nếu muốn tất cả log bao gồm cả alert thì phải cài thêm integration "Custom Log" và tự parse theo format của pfsense

### Tích hợp sysmon và powershell log vào Elastic SIEM

> Mình đã tích hợp những nguồn log này ở phần trước.

### Use case DDos
**1. DDOS là gì?**

- Distributed denial-of-service (ddos) là cuộc tấn công từ chối dịch vụ phân tán nhắm đến mục tiêu như là website hoặc các server thông qua việc gây gián đoạn dịch vụ mạng nhằm làm cạn kiệt tài nguyên của ứng dụng. Cách tấn công là gây tràn site bằng những lưu lượng lỗi, khiến cho máy chủ cạn kiệt tài nguyên. 
![image](https://hackmd.io/_uploads/Hys_-o_Dxe.png)

**2. Tiến hành tấn công ddos vào máy chủ windows 10**

**`Attack:`**

Cài đặt dvwa trên windows 10 để mô phỏng các cuộc tấn công
![image](https://hackmd.io/_uploads/Sk55ZsdPgg.png)

Bên máy kali cài đặt tool slowloris để có thể dos vào trang dvwa
![image](https://hackmd.io/_uploads/BJUjZs_vxl.png)

Tiến hành ddos thì ta thấy tool bắt đầu gửi những packet liên tục đến mục tiêu. Khiến cho trang không thể truy cập được

![image](https://hackmd.io/_uploads/B1D2biuwgx.png)

**`Detection:`**
Qua bên elastic để theo dõi các log được gửi về

![image](https://hackmd.io/_uploads/Skq6WiuDgx.png)

==> Ta có thể thấy các một số lượng lớn log thuộc máy windows gửi về dưới dạng là `endpoint.events.network` và đây cũng là thể loại dataset stream cần theo dõi khi nghi ngờ hệ thống bị ddos. Ở đây ta thấy bị ddos trên trang dvwa theo cổng ==80== và ip nạn nhân là ==192.168.1.100==

![image](https://hackmd.io/_uploads/S17AWsOwxe.png)

Máy ddos có địa chỉ là ==192.168.1.99==
Tiếp tục để elastic SIEM có thể thông báo về khả năng ddos ta cần tạo rule theo thể loại threshole

- Loại rule này sẽ thiết lập một ngưỡng nhất định nếu log trùng nhau bị lập nhiều lần sẽ kích hoạt rule
- Trong trường hợp này ta thấy website của hệ thống bị tấn công và tấn công trên cổng 80 nên ta sẽ gán query là ==destination.port: 80==
- Đặt nếu một ip liên tục gửi trên ==100== lần sẽ kích hoạt rule

![image](https://hackmd.io/_uploads/ryKyzodDex.png)
![image](https://hackmd.io/_uploads/r1LefiOwle.png)
![image](https://hackmd.io/_uploads/ry9gMj_wlg.png)

==> Chạy ddos lại lần nữa và thấy hệ thống SIEM cảnh báo

![image](https://hackmd.io/_uploads/ryzbMouvee.png)

**`Response:`**
Có nhiều cách để khắc phục, giảm thiểu và ngăn chặn ddos
1. Sử dụng tính năng trên tường lửa pfSense kết hợp với suricata để ngăn chặn, giảm thiểu
- Cài đặt suricata package
- Config interfaces cho suricata
![image](https://hackmd.io/_uploads/Hk9Szsdwge.png)

 
- Ở đây sẽ chọn vùng mạng Lan
 ![image](https://hackmd.io/_uploads/ry_Ifodvgg.png)


- Để có thể giảm thiểu và ngăn chặn dos/ddos ta cần chọn block offenders
 ![image](https://hackmd.io/_uploads/HJaLMo_wxl.png)


- Khuyến khích tạo một ip pass list gồm những ip quan trọng như của quản trị để không bị chặn khi cuộc tấn công diễn ra
 ![image](https://hackmd.io/_uploads/rkIPzouPxg.png)

- Tiếp tục bên mục global settings chọn cài đặt ETopen để có thể ngăn chặn mối đe dọa
 ![image](https://hackmd.io/_uploads/BkjPGsuDgl.png)

- Tạo rule để có thể phát hiện dos/ddos
```
#DOS ATTACK DETECTION
alert tcp !$HOME_NET any -> $HOME_NET any (flags: S; msg:"Possible SYN DoS"; flow: stateless; threshold: type both, track by_dst, count 1000, seconds 3; sid:10002;rev:1;)
#alert tcp !$HOME_NET any -> $HOME_NET any (flags: A; msg:"Possible ACK DoS"; flow: stateless; threshold: type both, track by_dst, count 1000, seconds 3; sid:10001;rev:1;)
alert tcp !$HOME_NET any -> $HOME_NET any (flags: R; msg:"Possible RST DoS"; flow: stateless; threshold: type both, track by_dst, count 1000, seconds 3; sid:10003;rev:1;)
alert tcp !$HOME_NET any -> $HOME_NET any (flags: F; msg:"Possible FIN DoS"; flow: stateless; threshold: type both, track by_dst, count 1000, seconds 3; sid:10004;rev:1;)
alert udp !$HOME_NET any -> $HOME_NET any (msg:"Possible UDP DoS"; flow: stateless; threshold: type both, track by_dst, count 1000, seconds 3; sid:10005;rev:1;)
alert icmp !$HOME_NET any -> $HOME_NET any (msg:"Possible ICMP DoS"; threshold: type both, track by_dst, count 250, seconds 3; sid:10006;rev:1;)

#DDOS ATTACK DETECTION
alert tcp !$HOME_NET any -> $HOME_NET any (flags: S; msg:"Possible SYN DDoS"; flow: stateless; threshold: type both, track by_dst, count 100000, seconds 10; sid:100002;rev:1;)
alert tcp !$HOME_NET any -> $HOME_NET any (flags: A; msg:"Possible ACK DDoS"; flow: stateless; threshold: type both, track by_dst, count 100000, seconds 10; sid:100001;rev:1;)
alert tcp !$HOME_NET any -> $HOME_NET any (flags: R; msg:"Possible RST DDoS"; flow: stateless; threshold: type both, track by_dst, count 100000, seconds 10; sid:100003;rev:1;)
alert tcp !$HOME_NET any -> $HOME_NET any (flags: F; msg:"Possible FIN DDoS"; flow: stateless; threshold: type both, track by_dst, count 100000, seconds 10; sid:100004;rev:1;)
alert udp !$HOME_NET any -> $HOME_NET any (msg:"Possible UDP DDoS"; flow: stateless; threshold: type both, track by_dst, count 100000, seconds 10; sid:100005;rev:1;)
alert icmp !$HOME_NET any -> $HOME_NET any (msg:"Possible ICMP DDoS"; threshold: type both, track by_dst, count 100000, seconds 10; sid:100006;rev:1;)

#PING OF DEATH DETECTION
alert icmp any any -> $HOME_NET any (msg:"Possible Ping of Death"; dsize: > 10000; sid:555555;rev:1;)
```

 ![image](https://hackmd.io/_uploads/Skz_fouvxx.png)

- Tấn công ddos lại và ta thấy alert bắt đầu hiện cuộc tấn công từ máy ==192.168.1.99== liên tục từ nhiều port khác nhau đánh vào máy nạn nhận với ==port 80== 
 ![image](https://hackmd.io/_uploads/HyFjGjODel.png)

==> Sau đó pfSense đã tự block ip tấn công
 ![image](https://hackmd.io/_uploads/SkXhzidvel.png)

2. sử dụng WAF cho website để chặn ddos
3. Liên tục theo dõi lưu lượng truy cập vào website và hệ thống
4. Giới hạn số lượng truy cập
5. Nếu dịch vụ bị tấn công không quan trọng có thể tắt tạm thời để nhường tài nguyên cho các dịch vụ khác
6. Chuẩn bị băng thông dự phòng
7. Sử dụng kỹ thuật anycast network diffusion( sử dụng nhiều điểm cuối phân tán)

### Trigger rule được build sẵn từ Elastic

#### Suspicious Execution via Scheduled Task

5.1 Giới thiệu
Xác định việc thực thi một chương trình đáng ngờ thông qua các tác vụ được lên lịch bằng cách xem xét dòng quy trình và cách sử dụng dòng lệnh.

5.2 Tags:
- Domain: Endpoint 
- OS: Windows 
- Use Case: Threat Detection 
- Tactic: Persistence 
- Tactic: Execution 
- Data Source: Elastic Defend

5.3 Nguồn log: 
- winlogbeat-* 
- logs-endpoint.events.process-* 
- logs-windows.* 

5.4 Rule type: eql
5.5 Severity: medium
5.6 Risk score: 47
5.7 MITRE ATT&CK™
```!
Tactic:
    - Name: Persistence (Duy trì)
    - ID: TA0003 
    - Reference URL: https://attack.mitre.org/tactics/TA0003/ 
Technique:
    - Name: Scheduled Task/Job 
    - ID: T1053 
    - Reference URL: https://attack.mitre.org/techniques/T1053/ 
Sub-technique:
    - Name: Scheduled Task 
    - ID: T1053.005 
    - Reference URL: https://attack.mitre.org/techniques/T1053/005/ 
Tactic:
    - Name: Execution ( Thực thi)
    - ID: TA0002 
    - Reference URL: https://attack.mitre.org/tactics/TA0002/ 
Mô tả: Chiến thuật này liên quan đến các kỹ thuật mà kẻ tấn công sử dụng để thực thi mã độc hại trên hệ thống mục tiêu. Mục tiêu là chạy mã của kẻ tấn công trên hệ thống local để thực hiện các hoạt động tiếp theo.

Technique:
    - Name: Scheduled Task/Job 
    - ID: T1053 
    - Reference URL: https://attack.mitre.org/techniques/T1053/ 
Mô tả: Kỹ thuật này liên quan đến việc kẻ tấn công tạo hoặc sửa đổi Scheduled Task trên hệ thống để thực thi mã độc hại. Điều này có thể được sử dụng cả cho mục đích duy trì và thực thi.
Sub-technique:
    - Name: Scheduled Task 
    - ID: T1053.005 
    - Reference URL: https://attack.mitre.org/techniques/T1053/005/ 
Mô tả: Đây là một kỹ thuật cụ thể trong việc sử dụng Scheduled Task trên hệ thống Windows. Kẻ tấn công có thể tạo hoặc sửa đổi Scheduled Task để thực thi các lệnh hoặc script độc hại.
```

5.8 Các trường hợp Rule Trigger và false positive
Các trường hợp Rule Trigger:
1. Tạo tác vụ định kỳ mới với lệnh đáng ngờ:
    - Sử dụng PowerShell hoặc cmd để tải xuống và thực thi mã từ internet.
    - Thực thi các lệnh mã hóa hoặc được obfuscated.
2. Sửa đổi tác vụ định kỳ hiện có:
    - Thay đổi lệnh thực thi của tác vụ đã tồn tại thành một lệnh đáng ngờ.
3. Thời gian thực thi bất thường:
    - Tác vụ được cấu hình để chạy vào thời điểm không bình thường (ví dụ: giữa đêm).

Các trường hợp false positive:

- Các tác vụ định kỳ được tạo bởi quản trị viên hệ thống cho mục đích bảo trì.
- Nhiều phần mềm sử dụng tác vụ định kỳ để kiểm tra và cài đặt cập nhật.
- Các công cụ sao lưu thường sử dụng tác vụ định kỳ để thực hiện sao lưu tự động.
- Phần mềm chống virus có thể tạo tác vụ định kỳ để thực hiện quét hệ thống.
- Các ứng dụng đồng bộ hóa dữ liệu có thể sử dụng tác vụ định kỳ.
- Trong môi trường doanh nghiệp, có thể có nhiều script tự động hóa chạy như tác vụ định kỳ.

5.9 Rule query
![image](https://hackmd.io/_uploads/rJAXBo_vxg.png)
![image](https://hackmd.io/_uploads/HkJNSjdPxl.png)

Giải thích:
Đây là một truy vấn quy tắc được sử dụng trong việc giám sát hoặc phát hiện an ninh cho hệ thống, có thể là dùng cho một công cụ hoặc nền tảng cho phép định nghĩa các quy tắc để giám sát các tiến trình và sự kiện. Phân tích:
1. lọc cơ bản:
   - process where host.os.type == "windows": Lựa chọn các tiến trình đang chạy trên hệ điều hành Windows.
   - and event.type == "start": Tập trung vào các sự kiện khởi động tiến trình.
2. Xác định các tiến trình cụ thể:
   - process.parent.name : "svchost.exe": Lọc các tiến trình mà tiến trình cha có tên là "svchost.exe".
   - and process.parent.args : "Schedule": Lọc các tiến trình mà tiến trình cha có đối số là "Schedule".
3. Xác định các chương trình đáng ngờ:
   - process.pe.original_file_name in (...): Kiểm tra xem tên tệp gốc của tiến trình có khớp với bất kỳ tên tệp thực thi nghi ngờ nào được liệt kê không. Những tên này bao gồm các chương trình thực thi thường bị lạm dụng như cscript.exe, powershell.exe, cmd.exe, v.v.
4. Xác định các đường dẫn đáng ngờ:
   - process.args : (...): Lọc các tiến trình dựa trên đối số của chúng khớp với bất kỳ đường dẫn đáng ngờ nào được chỉ định. Những đường dẫn này bao gồm các thư mục người dùng (C:\Users\*), thư mục hệ thống (C:\Windows\Temp\*), và các vị trí khác có thể nhạy cảm.
5. Loại trừ:
   - not (...): Loại trừ một số thực thi đã biết là hợp lệ hoặc được phép có thể gây ra các báo động giả trong quá trình phát hiện.
     - Loại trừ cmd.exe thực thi các tệp .bat từ các vị trí cụ thể.
     - Loại trừ cscript.exe chạy calluxxprovider.vbs từ một đường dẫn cụ thể.
     - Loại trừ powershell.exe với đối số cụ thể được thực thi bởi một ID người dùng cụ thể (S-1-5-18), thường liên quan đến các tiến trình hệ thống.
     - Loại trừ msiexec.exe được thực thi bởi một ID người dùng cụ thể (S-1-5-18), một lần nữa liên quan đến các tiến trình hệ thống.

5.10 Demo
**Attack**
- Tình huống: Kẻ tấn công đã kiểm soát được SHELL máy nạn nhân và đã thực hiện chèn mã độc vào thư mục temp nhưng kẻ tấn công muốn xóa dấu vết mỗi khi user đăng nhập hệ thống
```shell!
Start-Process "C:\Windows\System32\svchost.exe" -ArgumentList 'netsvcs -p -s Schedule -File "C:\Users\tien\del_temp.ps1"' -WindowStyle Hidden
```
**Detect**

Log của dịch vụ liên quan đến task Schedule
- Tiến trình PowerShell Đáng ngờ (PID 12668)
- Command Line: `"C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" evchost.exe netsvcs -p -s Schedule -File "C:\Users\tisen\del_temp.ps1"`
- Tham số bất thường:
    ```!
    -s Schedule: Thực thi qua Task Scheduler (cơ chế persistence thường được sử dụng).
    -File C:\Users\tisen\del_temp.ps1: Tệp PowerShell script đáng ngờ
    ```
Tiến trình Cha (PID 1036) - `svchost.exe`
- Command Line: `C:\WINDOWS\system32\svchost.exe +k netsvcs -p`
    > Quan hệ cha-con bất thường: svchost.exe (dịch vụ hệ thống) sinh ra powershell.exe → Vi phạm mô hình hoạt động thông thường (cảnh báo "Unusual Parent-Child Relationship").

![image](https://hackmd.io/_uploads/rJ_-ujuDee.png)
![image](https://hackmd.io/_uploads/BJYWOidvex.png)
![image](https://hackmd.io/_uploads/HkjZdoODxg.png)

Alert về dâu hiệu bất thường của Schedule Task
![image](https://hackmd.io/_uploads/ByoXOs_vlx.png)

**Response**
Ngắn hạn: các tool có thể hổ trợ như:
1.	Sysinternals Autoruns có thể phát hiện các thay đổi hệ thống như hiển thị các công việc hiện đang được lên lịch.
2.	Các công cụ như TCPView &; Process Explore có thể giúp xác định các kết nối từ xa cho các dịch vụ hoặc quy trình đáng ngờ.
Cấu hình các Task Scheduler để thực thi dưới dạng tài khoản được xác thực thay vì hệ thống.
Vô hiệu hóa task đáng ngờ: 
    - Sử dụng lệnh: schtasks /query /fo LIST /v để liệt kê tất cả tasks.
    - Vô hiệu hóa task đáng ngờ: schtasks /change /tn "TÊN_TASK" /disable
    
Dài hạn
Cập nhật hệ thống: Đảm bảo tất cả hệ điều hành và phần mềm được cập nhật với các bản vá bảo mật mới nhất. 
Triển khai GPO: 
1. Hạn chế việc tạo Scheduled Tasks cho người dùng không được ủy quyền.
2. Giới hạn quyền thực thi từ các thư mục người dùng và thư mục Temp.
3. Implement Application Whitelisting: Chỉ cho phép các ứng dụng đã được phê duyệt chạy trên hệ thống. 
4. Tăng cường giám sát: 
    - Cấu hình cảnh báo cho việc tạo hoặc sửa đổi Scheduled Tasks.
    - Theo dõi chặt chẽ các hoạt động trong thư mục đáng ngờ được liệt kê trong rule.

---

# Lời kết: Đây cũng là kết thúc cho Project xây dựng mô phỏng về ELK stack/SIEM và thực hành tìm hiểu trên SIEM. Cảm ơn mọi người đã đọc.
