# LAB 8 – LOCAL DNS ATTACK

## DNS Spoofing và DNS Cache Poisoning trong môi trường pfSense và VirtualBox

### 1. Thông tin sinh viên

- **Họ và tên:** Hoàng Công Trường Lộc
- **Mã số sinh viên:** 1150080063
- **Môn học:** An toàn và Bảo mật Hệ thống Thông tin
- **Bài thực hành:** Lab 8
- **Hệ điều hành máy thật:** Pop!_OS Linux
- **Phần mềm ảo hóa:** Oracle VirtualBox
- **Firewall:** pfSense 2.7.2
- **Hệ điều hành máy ảo:** Ubuntu Server, Kali Linux
- **Video YTB:** https://youtu.be/UqfWq2Eb-X8

---

### 2. Tên bài Lab

**Lab 8 – Xây dựng và cấu hình mạng DNS-LAB cô lập, tìm hiểu DNS Spoofing và DNS Cache Poisoning trên pfSense và Oracle VirtualBox.**

---

### 3. Mục tiêu

Trong Lab 8, em thực hiện xây dựng hệ thống mạng DNS-LAB riêng biệt nhằm tìm hiểu nguyên lý hoạt động của DNS và các nguy cơ bảo mật liên quan đến quá trình phân giải tên miền.

Các mục tiêu chính:

- Xây dựng môi trường thực hành trên Oracle VirtualBox.
- Sử dụng pfSense làm Firewall để phân chia và quản lý mạng.
- Tạo mạng DNS-LAB riêng biệt với các mạng LAN và DMZ.
- Cấu hình Firewall Rules để kiểm soát lưu lượng.
- Tìm hiểu cách triển khai DNS Server bằng BIND9.
- Tìm hiểu các kỹ thuật DNS Spoofing và DNS Cache Poisoning.
- Quan sát gói tin DNS bằng Wireshark.
- Nghiên cứu biện pháp phòng chống và khôi phục hệ thống sau thử nghiệm.

---

### 4. Mô hình hệ thống

| Thiết bị | Vai trò | Địa chỉ IP |
|---|---|---|
| pfSense | Firewall / Gateway DNSLAB | 10.9.0.1/24 |
| DNS-SERVER | Máy chủ DNS BIND9 | 10.9.0.10 |
| VICTIM | Máy người dùng | 10.9.0.100 |
| Kali Linux | Máy mô phỏng tấn công | 10.9.0.200 |
| APP-TEST | Máy chủ DNS hợp lệ | 172.16.6.20 |
| DMZ-WEB | Máy chủ Web | 172.16.6.10 |

Mạng DNS-LAB sử dụng dải địa chỉ `10.9.0.0/24` và được kết nối với pfSense bằng Internal Network `dns-lab`.

---

### 5. Các bước thực hiện

#### Bước 1: Chuẩn bị môi trường

- Mở Oracle VirtualBox.
- Kiểm tra các máy ảo pfSense, Ubuntu Server, Kali Linux và DMZ-Web.
- Tạo Snapshot `LAB8_START_BACKUP` để sao lưu cấu hình pfSense.
- Kiểm tra trạng thái các máy ảo trước khi thay đổi cấu hình mạng.

#### Bước 2: Cấu hình Adapter 5 cho pfSense

Sử dụng VBoxManage để thêm card mạng mới:

```bash
VBoxManage modifyvm "pfSense" --nic5 intnet --intnet5 "dns-lab"
VBoxManage modifyvm "pfSense" --nictype5 82540EM
VBoxManage modifyvm "pfSense" --cableconnected5 on
```

Kiểm tra cấu hình:

```bash
VBoxManage showvminfo "pfSense" --details
```

#### Bước 3: Cấu hình interface DNSLAB

Đăng nhập giao diện quản trị pfSense và thực hiện:

**Interfaces → Assignments → Add em4**

Thiết lập:

- Enable Interface: Bật
- Description: `DNSLAB`
- IPv4 Configuration Type: Static IPv4
- IPv4 Address: `10.9.0.1/24`
- Upstream Gateway: None

Sau đó chọn Save và Apply Changes.

#### Bước 4: Cấu hình Firewall Rules

Truy cập:

**Firewall → Rules → DNSLAB**

Thiết lập các quy tắc:

| STT | Action | Source | Destination | Port |
|---|---|---|---|---|
| 1 | Pass | 10.9.0.10 | 172.16.6.20 | DNS 53 |
| 2 | Pass | 10.9.0.0/24 | 172.16.6.10 | HTTP 80 |
| 3 | Block + Log | 10.9.0.0/24 | 10.6.0.0/24 | Any |
| 4 | Block + Log | 10.9.0.0/24 | 192.168.56.0/24 | Any |
| 5 | Block + Log | 10.9.0.0/24 | Any | Any |

Các quy tắc giúp kiểm soát kết nối từ mạng DNSLAB đi qua pfSense và bảo vệ các mạng nội bộ khác.

#### Bước 5: Triển khai DNS-SERVER

Nội dung thực hành:

- Cấu hình Ubuntu Server trong mạng DNSLAB.
- Thiết lập địa chỉ IP `10.9.0.10/24`.
- Cài đặt dịch vụ BIND9.
- Cấu hình Forward Zone và Reverse Zone.
- Kiểm tra DNS bằng dig, nslookup.

#### Bước 6: Cấu hình Victim và Kali Linux

- Victim: `10.9.0.100`
- Kali Linux: `10.9.0.200`
- Mạng sử dụng: `dns-lab`
- Công cụ quan sát: Wireshark và tcpdump.

#### Bước 7: Tìm hiểu các tình huống tấn công DNS

**Task 1 – HOSTS File Redirection**

Quan sát tác động khi file hosts trên máy Victim bị thay đổi, dẫn đến chuyển hướng truy cập tên miền.

**Task 2 – Direct DNS Spoofing**

Tìm hiểu cách một phản hồi DNS giả mạo có thể ảnh hưởng đến kết quả phân giải tên miền của máy Victim.

**Task 3 – DNS Cache Poisoning**

Tìm hiểu hiện tượng dữ liệu DNS không chính xác được lưu trong cache của DNS resolver và có thể ảnh hưởng đến nhiều máy sử dụng chung DNS Server.

#### Bước 8: Khôi phục và phòng chống

- Khôi phục file hosts ban đầu.
- Xóa DNS cache.
- Kiểm tra lại kết quả phân giải DNS hợp lệ.
- Tắt các công cụ mô phỏng.
- Tìm hiểu DNSSEC, DNS COOKIE và phân đoạn mạng.
- Kiểm tra lại Firewall Rules và cấu hình bảo mật.

---


### 6. Kết luận

Qua Lab 8, em hiểu rõ hơn về cách xây dựng một môi trường mạng DNS riêng biệt bằng Oracle VirtualBox và pfSense.

Bài thực hành giúp em củng cố kiến thức về cấu hình Firewall, phân chia mạng và kiểm soát lưu lượng giữa các vùng mạng. Đồng thời, em tìm hiểu được những nguy cơ liên quan đến việc giả mạo phản hồi DNS, thay đổi file hosts và nhiễm độc bộ nhớ đệm DNS.

Việc triển khai môi trường thực hành cô lập giúp hạn chế ảnh hưởng đến các hệ thống khác và tạo điều kiện thuận lợi cho việc phân tích, đánh giá các cơ chế phòng chống tấn công DNS.

---
