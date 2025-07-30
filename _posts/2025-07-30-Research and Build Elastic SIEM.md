---
title: Dự án tìm hiểu và xây dựng Elastic SIEM (2024)
date: 2025/07/30 18:07 +0700
tags: [SIEM, Elastic, ELK, Research, Building]
categories: [Personal Project]
author: KraizyLuc
math: true
image: https://cyberopex.ch/wp-content/uploads/2024/04/Elastic_NV_logo.svg-e1714478277482-1024x379.png
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

# Đây cũng là kết thúc cho Project xây dựng mô phỏng về ELK stack/SIEM. Cảm ơn mọi người đã đọc.
