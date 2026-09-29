# LAB 4 - KHẢO SÁT VÀ ĐÁNH GIÁ BỀ MẶT MẠNG BẰNG NMAP

## 1. Thông tin sinh viên

- Họ và tên: Hoàng Công Trường Lộc
- MSSV: 1150080063
- Môn học: An toàn và Bảo mật Hệ thống Thông tin
- Tên Lab: Lab 4 - Khảo sát và đánh giá bề mặt mạng bằng Nmap
- Link YouTube: https://youtu.be/8g6a9F_ZJjg

---

## 2. Phiên bản môi trường thực hành

| Thành phần | Phiên bản / Cấu hình |
|---|---|
| Máy thật | Pop!_OS Linux |
| Phần mềm ảo hóa | Oracle VirtualBox |
| Máy quét | Kali Linux 64-bit |
| Nmap | Nmap 7.99 |
| Máy đích | Metasploitable 2.0.0 |
| Kiểu mạng | Host-Only Adapter |
| Host-Only Network | vboxnet0 |
| Dải mạng | 192.168.56.0/24 |
| IP Kali Linux | 192.168.56.102/24 |
| IP Metasploitable 2 | 192.168.56.101/24 |

---

## 3. Cách dựng môi trường

Sử dụng Oracle VirtualBox trên máy thật Pop!_OS để xây dựng môi trường thực hành.

Trong VirtualBox tạo mạng Host-Only có tên:

`vboxnet0`

Dải mạng sử dụng:

`192.168.56.0/24`

### Kali Linux

Kali Linux được sử dụng làm máy quét.

Cấu hình:

- Adapter 1: Host-only Adapter
- Name: `vboxnet0`
- Adapter 2: Disable

Sau khi cấu hình, Kali Linux nhận được địa chỉ:

`192.168.56.102/24`

### Metasploitable 2

Metasploitable 2 được sử dụng làm máy đích.

Máy ảo được tạo từ file:

`Metasploitable.vmdk`

Cấu hình mạng:

- Adapter 1: Host-only Adapter
- Name: `vboxnet0`

Địa chỉ IP:

`192.168.56.101/24`

Trước khi thực hành, tạo snapshot:

`Before-Lab4`

cho cả Kali Linux và Metasploitable 2.

Sau khi hoàn tất cấu hình, kiểm tra kết nối bằng:

`ping -c 4 192.168.56.101`

Kết quả nhận đủ 4 gói tin và packet loss bằng 0%.

---

## 4. Các tình huống đã thực hiện

### 4.1. Kiểm tra địa chỉ IP Kali Linux

Lệnh:

`ip -br addr`

Kết quả:

- Interface: `eth0`
- Trạng thái: UP
- IP: `192.168.56.102/24`

Kết quả: **PASS**

---

### 4.2. Kiểm tra địa chỉ IP Metasploitable 2

Lệnh:

`ifconfig`

Kết quả:

- Interface: `eth0`
- IP: `192.168.56.101`
- Netmask: `255.255.255.0`

Kết quả: **PASS**

---

### 4.3. Kiểm tra kết nối giữa Kali và Metasploitable 2

Lệnh:

`ping -c 4 192.168.56.101`

Kết quả:

- 4 packets transmitted
- 4 packets received
- 0% packet loss

Kết quả: **PASS**

---

### 4.4. Kiểm tra phiên bản Nmap

Lệnh:

`nmap --version`

Kết quả:

`Nmap version 7.99`

Kết quả: **PASS**

---

### 4.5. Host Discovery

Lệnh:

`nmap -sn 192.168.56.0/24`

Kết quả:

Nmap phát hiện 4 host đang hoạt động trong mạng Host-Only.

Trong đó:

- `192.168.56.1`: Host-Only Adapter của máy thật
- `192.168.56.100`: VirtualBox/DHCP service
- `192.168.56.101`: Metasploitable 2
- `192.168.56.102`: Kali Linux

Kết quả: **PASS**

---

### 4.6. TCP Connect Scan

Lệnh:

`nmap -sT 192.168.56.101`

Kết quả:

- 23 cổng TCP open
- 977 cổng TCP closed

Một số cổng mở:

- 21/tcp - FTP
- 22/tcp - SSH
- 23/tcp - Telnet
- 25/tcp - SMTP
- 53/tcp - DNS
- 80/tcp - HTTP
- 139/tcp - NetBIOS
- 445/tcp - SMB
- 3306/tcp - MySQL
- 5432/tcp - PostgreSQL
- 5900/tcp - VNC

Kết quả: **PASS**

---

### 4.7. SYN Scan

Lệnh:

`sudo nmap -sS 192.168.56.101`

Kết quả:

- 23 cổng TCP open
- 977 cổng TCP closed

Kết quả gần tương tự TCP Connect Scan.

SYN Scan yêu cầu quyền sudo/root vì cần tạo raw packet.

Kết quả: **PASS**

---

### 4.8. FIN Scan

Lệnh:

`sudo nmap -sF 192.168.56.101`

Kết quả:

Nhiều cổng được hiển thị ở trạng thái:

`open|filtered`

Trạng thái này không có nghĩa chắc chắn cổng đang mở mà Nmap chưa thể phân biệt giữa cổng mở và cổng bị bộ lọc làm im lặng.

Kết quả: **PASS**

---

### 4.9. Xmas Scan

Lệnh:

`sudo nmap -sX 192.168.56.101`

Kết quả:

Nhiều cổng có trạng thái:

`open|filtered`

Kết quả tương tự FIN Scan.

Kết quả: **PASS**

---

### 4.10. NULL Scan

Lệnh:

`sudo nmap -sN 192.168.56.101`

Kết quả:

Nhiều cổng được Nmap hiển thị:

`open|filtered`

Kết quả: **PASS**

---

### 4.11. ACK Scan

Lệnh:

`sudo nmap -sA 192.168.56.101`

Kết quả:

1000 cổng TCP được Nmap xác định ở trạng thái:

`unfiltered`

ACK Scan được sử dụng để quan sát chính sách lọc và không dùng để khẳng định trực tiếp cổng đang open.

Kết quả: **PASS**

---

### 4.12. UDP Scan

Lệnh:

`sudo nmap -sU --top-ports 20 192.168.56.101`

Một số kết quả:

| Cổng UDP | Trạng thái | Dịch vụ |
|---|---|---|
| 53/udp | open | domain |
| 67/udp | open\|filtered | dhcps |
| 137/udp | open | netbios-ns |

UDP Scan thường chậm và có nhiều trạng thái `open|filtered` do UDP không sử dụng cơ chế bắt tay như TCP.

Kết quả: **PASS**

---

### 4.13. Version Detection

Lệnh:

`nmap -sV 192.168.56.101`

Thực hiện thêm:

`nmap -sV -p 21,22,80,445,3306 192.168.56.101`

Kết quả:

| Port | Protocol | Service | Version |
|---|---|---|---|
| 21 | tcp | ftp | vsftpd 2.3.4 |
| 22 | tcp | ssh | OpenSSH 4.7p1 Debian 8ubuntu1 |
| 80 | tcp | http | Apache httpd 2.2.8 (Ubuntu) DAV/2 |
| 445 | tcp | SMB | Samba smbd 3.X - 4.X |
| 3306 | tcp | mysql | MySQL 5.0.51a-3ubuntu5 |

Các dịch vụ trên đều là những phiên bản khá cũ, cần được cập nhật và kiểm tra các lỗ hổng bảo mật đã được công bố.

Kết quả: **PASS**

---

### 4.14. OS Detection

Lệnh:

`sudo nmap -O 192.168.56.101`

Mục đích là sử dụng đặc điểm TCP/IP stack để suy đoán hệ điều hành máy đích.

Kết quả cho thấy máy đích thuộc nhóm Linux/Unix.

OS Fingerprinting chỉ mang tính suy đoán, không nên coi kết quả là tuyệt đối chính xác.

Kết quả: **PASS**

---

### 4.15. Aggressive Scan

Lệnh:

`sudo nmap -A 192.168.56.101`

Aggressive Scan cung cấp nhiều thông tin gồm:

- Version Detection
- OS Detection
- Default NSE Scripts
- Traceroute
- Thông tin dịch vụ

Phương pháp này cung cấp lượng thông tin lớn hơn nhưng tạo nhiều lưu lượng mạng hơn và dễ bị hệ thống giám sát phát hiện.

Kết quả: **PASS**

---

### 4.16. Thu thập thông tin SMB bằng NSE

Lệnh:

`sudo nmap -p 445 --script smb-os-discovery 192.168.56.101`

Kết quả:

- 445/tcp: open
- Service: microsoft-ds
- OS: Unix (Samba 3.0.20-Debian)
- Computer name: metasploitable
- Domain name: localdomain
- FQDN: metasploitable.localdomain

Script thu thập thành công thông tin SMB của máy Metasploitable 2.

Kết quả: **PASS**

---

### 4.17. Kiểm tra MS17-010

Lệnh:

`sudo nmap -p 445 --script smb-vuln-ms17-010 192.168.56.101`

Kết quả:

- Cổng 445/tcp ở trạng thái open
- Dịch vụ microsoft-ds hoạt động
- Script không trả về dòng `VULNERABLE`
- Script cũng không đưa ra kết luận rõ ràng rằng hệ thống không bị ảnh hưởng

Do đó chưa đủ bằng chứng để kết luận Metasploitable 2 có hoặc không có lỗ hổng MS17-010.

Biện pháp phòng thủ:

- Cập nhật Samba/SMB
- Vá hệ điều hành
- Giới hạn cổng 445 bằng firewall
- Chỉ cho phép các máy cần thiết truy cập SMB

Kết quả: **PASS**

---

### 4.18. Xuất kết quả Nmap dạng TXT

Lệnh:

`nmap -sV -oN lab4_nmap.txt 192.168.56.101`

Kiểm tra:

`ls -l lab4_nmap.txt`

Kết quả:

File `lab4_nmap.txt` được tạo thành công.

Kết quả: **PASS**

---

### 4.19. Xuất kết quả Nmap dạng XML

Lệnh:

`nmap -sV -oX lab4_nmap.xml 192.168.56.101`

Kiểm tra:

`ls -l lab4_nmap.xml`

Kết quả:

File `lab4_nmap.xml` được tạo thành công.

Kết quả: **PASS**

---

### 4.20. Xuất kết quả Grepable

Lệnh:

`nmap -p 445 -oG smb.txt 192.168.56.101`

Sau đó:

`grep "445/open" smb.txt`

Kết quả:

`445/open/tcp//microsoft-ds///`

Điều này xác nhận cổng 445/tcp đang mở.

Kết quả: **PASS**

---

### 4.21. Chuyển XML sang HTML

Lệnh:

`xsltproc lab4_nmap.xml -o lab4_nmap.html`

Kiểm tra:

`ls -l lab4_nmap.html`

Kết quả:

File `lab4_nmap.html` được tạo thành công từ dữ liệu XML của Nmap.

Kết quả: **PASS**

---

## 5. Tổng hợp kết quả PASS/FAIL

| Nội dung | Kết quả |
|---|---|
| Tạo Host-Only Network | PASS |
| Cấu hình Kali Linux | PASS |
| Cấu hình Metasploitable 2 | PASS |
| Tạo snapshot Before-Lab4 | PASS |
| Kiểm tra IP Kali | PASS |
| Kiểm tra IP Metasploitable 2 | PASS |
| Ping kiểm tra kết nối | PASS |
| Kiểm tra Nmap | PASS |
| Host Discovery | PASS |
| TCP Connect Scan | PASS |
| SYN Scan | PASS |
| FIN Scan | PASS |
| Xmas Scan | PASS |
| NULL Scan | PASS |
| ACK Scan | PASS |
| UDP Scan | PASS |
| Version Detection | PASS |
| OS Detection | PASS |
| Aggressive Scan | PASS |
| NSE smb-os-discovery | PASS |
| NSE smb-vuln-ms17-010 | PASS |
| Xuất TXT | PASS |
| Xuất XML | PASS |
| Xuất Grepable | PASS |
| Chuyển XML sang HTML | PASS |

---

## 6. Lỗi gặp phải và cách khắc phục

### Lỗi 1: Kali Linux không nhận IPv4 Host-Only

Hiện tượng:

Khi chạy:

`ip -br addr`

interface mạng ở trạng thái UP nhưng chưa có địa chỉ IPv4 thuộc dải `192.168.56.0/24`.

Nguyên nhân:

Kali còn bật Adapter 2 ở chế độ Internal Network với tên `LabNet`.

Cách khắc phục:

Tắt Adapter 2 và chỉ giữ Adapter 1 ở chế độ:

- Host-only Adapter
- Name: `vboxnet0`

Sau khi khởi động lại, Kali nhận IP:

`192.168.56.102/24`

Kết quả: **PASS**

---

### Lỗi 2: aria2 báo HTTP 403 khi tải Metasploitable 2

Hiện tượng:

Khi tải Metasploitable 2 từ SourceForge bằng URL ban đầu, aria2 báo HTTP 403.

Cách khắc phục:

Sử dụng đường dẫn tải trực tiếp của SourceForge và chế độ tải nhiều kết nối.

Sau đó file:

`metasploitable-linux-2.0.0.zip`

được tải thành công.

Giải nén thu được:

`Metasploitable.vmdk`

Kết quả: **PASS**

---

### Lỗi 3: Nhập nhầm -sn và -sT

Đã nhập:

`nmap -sn 192.168.56.101`

trong khi cần TCP Connect Scan.

Phân biệt:

- `-sn`: Host Discovery
- `-sT`: TCP Connect Scan

Lệnh đúng:

`nmap -sT 192.168.56.101`

Kết quả: **PASS**

---

### Lỗi 4: Nhập số 0 thay cho chữ O

Đã nhập:

`sudo nmap -0 192.168.56.101`

Nmap báo:

`unrecognized option '-0'`

Nguyên nhân:

Nhập số `0` thay vì chữ `O` hoa.

Lệnh đúng:

`sudo nmap -O 192.168.56.101`

Kết quả: **PASS**

---

### Lỗi 5: Nhập nhầm cổng SMB

Ban đầu nhập:

`sudo nmap -p 455 --script smb-vuln-ms17-010 192.168.56.101`

Kết quả:

`455/tcp closed`

Nguyên nhân:

Nhập nhầm cổng 455 thay vì cổng SMB 445.

Lệnh đúng:

`sudo nmap -p 445 --script smb-vuln-ms17-010 192.168.56.101`

Kết quả:

`445/tcp open microsoft-ds`

Kết quả: **PASS**

---

## 7. Các file kết quả đã tạo

Các file được tạo trong quá trình thực hành:

- `lab4_nmap.txt`
- `lab4_nmap.xml`
- `smb.txt`
- `lab4_nmap.html`

Các file này được sử dụng làm bằng chứng cho kết quả quét Nmap.

---

## 8. Kết luận

Môi trường Lab 4 đã được dựng thành công trên Oracle VirtualBox.

Kali Linux được sử dụng làm máy quét với địa chỉ:

`192.168.56.102/24`

Metasploitable 2 được sử dụng làm máy đích với địa chỉ:

`192.168.56.101/24`

Hai máy được kết nối thông qua mạng Host-Only `vboxnet0`, giúp cô lập môi trường thực hành khỏi mạng bên ngoài.

Quá trình thực hành đã thực hiện thành công nhiều kỹ thuật Nmap bao gồm:

- Host Discovery
- TCP Connect Scan
- SYN Scan
- FIN Scan
- Xmas Scan
- NULL Scan
- ACK Scan
- UDP Scan
- Version Detection
- OS Detection
- Aggressive Scan
- NSE Script
- Xuất kết quả ra TXT
- Xuất kết quả ra XML
- Xuất Grepable
- Chuyển XML sang HTML

Qua Version Detection, Nmap phát hiện nhiều dịch vụ phiên bản cũ như:

- vsftpd 2.3.4
- OpenSSH 4.7p1
- Apache HTTP Server 2.2.8
- Samba
- MySQL 5.0.51a

Đây là những dịch vụ cần được cập nhật và kiểm tra các lỗ hổng đã được công bố.

NSE `smb-os-discovery` xác định hệ thống sử dụng Unix với Samba 3.0.20-Debian và tên máy là `metasploitable`.

Kiểm tra `smb-vuln-ms17-010` không trả về trạng thái `VULNERABLE`, vì vậy không đủ bằng chứng để kết luận hệ thống có hoặc không có MS17-010.
