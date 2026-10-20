# LAB 7 – Xây dựng chính sách Web Protection trên mô hình pfSense + Web Proxy

## 1. Thông tin sinh viên

- **Họ và tên:** Hoàng Công Trường Lộc
- **Mã số sinh viên:** 1150080063
- **Môn học:** An toàn và Bảo mật Hệ thống Thông tin
- **Bài thực hành:** Lab 7
- **Hệ điều hành máy thật:** Pop!_OS Linux
- **Phần mềm ảo hóa:** Oracle VirtualBox
- **Link video YouTube:** Chưa cập nhật

---

## 2. Tên bài Lab

**LAB 7 – Xây dựng chính sách Web Protection trên mô hình pfSense + Web Proxy**

## 3. Mục tiêu

Trong bài LAB 7, em thực hiện xây dựng hệ thống kiểm soát truy cập website bằng pfSense kết hợp với Web Proxy, nhằm:

- Sử dụng pfSense làm Firewall và Gateway để kiểm soát lưu lượng mạng.
- Triển khai Web Proxy bằng Squid và E2Guardian trên Ubuntu Server.
- Xây dựng chính sách cho phép, chặn và cảnh báo truy cập website theo danh sách.
- Ngăn người dùng truy cập website trực tiếp để bỏ qua Proxy.
- Sử dụng ClamAV để kiểm tra nội dung tải xuống.
- Cấu hình giới hạn truy cập theo thời gian và kiểm thử giới hạn dung lượng file.
- Theo dõi Firewall Log và Proxy Log để đánh giá hiệu quả của các chính sách.

## 4. Môi trường thực hành

| Thành phần | Địa chỉ IP theo mô hình | Vai trò |
|---|---|---|
| pfSense – LAN | 10.6.0.1 | Firewall, Gateway |
| DC01 | 10.6.0.10 | Active Directory, DNS |
| Client-A | 10.6.0.21 | Máy kiểm thử |
| Client-B | 10.6.0.22 | Máy kiểm thử |
| WEB-PROXY | 10.6.0.30 | Squid, E2Guardian, ClamAV |
| DMZ-WEB | 172.16.6.10 | Website mô phỏng |

**Lưu ý:** Đây là mô hình mục tiêu của bài thực hành. Không phải tất cả thành phần đều đã được triển khai.

## 5. Nội dung thực hiện

### 5.1. Kiểm tra hệ thống pfSense

- Kiểm tra cấu hình các card mạng WAN, LAN, MGMT và DMZ.
- Kiểm tra địa chỉ IP bằng lệnh `ifconfig em0`.
- Kiểm tra kết nối Internet bằng lệnh `ping -c 4 8.8.8.8`.
- Kiểm tra DNS bằng lệnh `nslookup google.com`.
- Kiểm tra và khắc phục lỗi DNS Resolver.

**Kết quả:**

- pfSense đã kết nối Internet thành công.
- Ping 8.8.8.8 nhận đủ 4/4 gói, không mất gói.
- Ban đầu DNS mặc định trả về lỗi SERVFAIL/REFUSED.
- Sau khi bổ sung DNS Server 8.8.8.8, hệ thống phân giải thành công google.com qua DNS nội bộ 127.0.0.1.

### 5.2. Cấu hình máy Client-A

- Kiểm tra cấu hình mạng trên Windows-Lab.
- Chuyển card mạng thử nghiệm sang Internal Network `lab6-lan`.
- Cấu hình IP 10.6.0.21/24.
- Cấu hình Default Gateway 10.6.0.1.
- Cấu hình DNS Server 10.6.0.10.

**Kết quả:**

- Ping đến pfSense 10.6.0.1 thành công, tỷ lệ mất gói 0%.
- Ping đến DC01 10.6.0.10 chưa thành công do chưa xác nhận được máy chủ hoặc khả năng phản hồi ICMP.

### 5.3. Triển khai Web Proxy

**Trạng thái: Chưa thực hiện.**

Dự kiến:

- Cài đặt Ubuntu Server 24.04 LTS.
- Cài đặt Squid, E2Guardian và ClamAV.
- Cấu hình E2Guardian tại cổng 8080.
- Cấu hình Squid chỉ lắng nghe trên 127.0.0.1:3128.

### 5.4. Xây dựng chính sách Web Protection

**Trạng thái: Chưa thực hiện.**

Các nhóm chính sách cần kiểm thử:

- Hard Block: chặn hoàn toàn website.
- Warning: hiển thị cảnh báo và cho phép vượt qua tạm thời.
- Exception: cho phép website trong danh sách ngoại lệ.
- Schedule: giới hạn truy cập theo khung thời gian 60 phút.
- Antivirus: kiểm tra file tải xuống bằng ClamAV.
- Download Size: kiểm thử giới hạn tải file lớn hơn 100 MB.

### 5.5. Kiểm thử và thu thập Log

**Trạng thái: Chưa thực hiện.**

Dự kiến kiểm tra:

- Firewall Log của pfSense.
- Access Log của E2Guardian.
- Access Log của Squid.
- Kết quả truy cập website từ Client-A và Client-B.

## 6. Kết quả hiện tại

Hệ thống đã hoàn thành bước kiểm tra kết nối pfSense và bước kết nối Client-A với mạng LAN.

Các phần cài đặt Web Proxy, tạo website mô phỏng, xây dựng chính sách lọc Web và kiểm thử bảo mật sẽ tiếp tục được thực hiện.

## 7. Kết luận

Qua giai đoạn chuẩn bị, em đã kiểm tra các thành phần mạng trong VirtualBox, xác nhận pfSense có thể kết nối Internet, xử lý sự cố phân giải DNS và cấu hình kết nối Client-A đến Firewall.

Đây là nền tảng để tiếp tục triển khai hệ thống Web Protection bằng Squid, E2Guardian và ClamAV trong các giai đoạn tiếp theo.

## 8. Video minh chứng

**YouTube:** Chưa cập nhật.

## 9. Tài liệu tham khảo

- Tài liệu hướng dẫn LAB 7 – Xây dựng chính sách Web Protection trên mô hình pfSense + Web Proxy.
- pfSense Documentation.
- Ubuntu Server Documentation.
- E2Guardian Documentation.
- Squid Proxy Documentation.
