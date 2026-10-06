# LAB 5 - XÂY DỰNG FIREWALL PFSENSE TRÊN ORACLE VIRTUALBOX

## 1. Thông tin sinh viên

- **Họ và tên:** Hoàng Công Trường Lộc
- **Mã số sinh viên:** 1150080063
- **Môn học:** An toàn và Bảo mật Hệ thống Thông tin
- **Bài thực hành:** Lab 5
- **Hệ điều hành máy thật:** Pop!_OS Linux
- **Phần mềm ảo hóa:** Oracle VirtualBox
LINKYTB https://youtu.be/e8p_6z2VU-I
---

## 2. Tên bài Lab

**Lab 5 - Xây dựng và cấu hình Firewall pfSense với WAN, LAN, DMZ và Outbound NAT trên Oracle VirtualBox**

---

## 3. Mục tiêu

Trong bài Lab 5, em thực hiện xây dựng mô hình mạng sử dụng pfSense làm Firewall với các mục tiêu:

- Cài đặt pfSense trên Oracle VirtualBox.
- Cấu hình các card mạng WAN, LAN và DMZ.
- Thiết lập địa chỉ IP cho mạng LAN và DMZ.
- Kiểm tra kết nối giữa máy thật và pfSense.
- Truy cập và cấu hình pfSense bằng giao diện WebGUI.
- Tạo máy Windows Server trong vùng DMZ.
- Cấu hình địa chỉ IP cho máy DMZ-Web.
- Tạo Firewall Rule cho vùng DMZ.
- Kiểm tra Automatic Outbound NAT của pfSense.
- Chuẩn bị môi trường cho các bài kiểm thử Firewall và Port Forward.

---

## 4. Mô hình mạng

| Thiết bị | Interface | Địa chỉ IP | Subnet Mask | Gateway |
|---|---|---|---|---|
| pfSense | WAN | DHCP | Theo mạng vật lý | DHCP |
| pfSense | LAN | 10.0.0.1 | 255.0.0.0 (/8) | - |
| pfSense | DMZ | 172.16.0.1 | 255.255.0.0 (/16) | - |
| Máy thật Pop!_OS | Host-only | 10.0.0.100 | 255.0.0.0 (/8) | Không đặt |
| DMZ-Web | DMZ | 172.16.0.2 | 255.255.0.0 (/16) | 172.16.0.1 |

Mô hình gồm ba vùng mạng chính:

- **WAN:** Kết nối pfSense với mạng ngoài thông qua Bridged Adapter.
- **LAN:** Mạng nội bộ sử dụng dải `10.0.0.0/8`.
- **DMZ:** Mạng dành cho máy chủ dịch vụ sử dụng dải `172.16.0.0/16`.

---

## 5. Nội dung thực hiện

### 5.1. Kiểm tra mạng trên máy thật Pop!_OS

Trước khi tạo mô hình, kiểm tra địa chỉ IP và bảng định tuyến:

```bash
ip addr
ip route
```

Máy thật đang sử dụng Wi-Fi với:

```text
IP: 192.168.0.101
Gateway: 192.168.0.1
Interface: wlp113s0
```

Trong quá trình kiểm tra phát hiện interface `vmnet8` sử dụng mạng:

```text
172.16.2.0/24
```

Dải mạng này nằm trong mạng DMZ `172.16.0.0/16`, do đó có khả năng gây chồng lấn mạng.

Tạm thời tắt interface `vmnet8`:

```bash
sudo ip link set vmnet8 down
```

Sau đó kiểm tra lại:

```bash
ip route
```

Kết quả route `172.16.2.0/24` đã biến mất.

---

### 5.2. Tải bộ cài pfSense

Di chuyển vào thư mục lưu bộ cài:

```bash
cd ~/Downloads/LAB5-pfSense/LAB5-pfSense/INSTALL_MEDIA
```

Tải pfSense CE 2.7.2:

```bash
wget https://atxfiles.netgate.com/mirror/downloads/pfSense-CE-2.7.2-RELEASE-amd64.iso.gz
```

Kiểm tra SHA-256:

```bash
sha256sum pfSense-CE-2.7.2-RELEASE-amd64.iso.gz
```

Kết quả:

```text
883fb7bc64fe548442ed007911341dd34e178449f8156ad65f7381a02b7cd9e4
```

Mã SHA-256 khớp với giá trị yêu cầu.

Giải nén:

```bash
gzip -dk pfSense-CE-2.7.2-RELEASE-amd64.iso.gz
```

Thu được file:

```text
pfSense-CE-2.7.2-RELEASE-amd64.iso
```

---

### 5.3. Tạo máy ảo pfSense

Tạo máy ảo mới trên Oracle VirtualBox với cấu hình:

```text
VM Name: pfSense
OS: BSD
Version: FreeBSD (64-bit)
RAM: 2048 MB
CPU: 2 vCPU
Virtual Disk: khoảng 20 GB
```

Gắn file ISO:

```text
pfSense-CE-2.7.2-RELEASE-amd64.iso
```

---

### 5.4. Cấu hình 3 card mạng cho pfSense

#### Adapter 1 - WAN

```text
Enable Network Adapter: Yes
Attached to: Bridged Adapter
Name: wlp113s0
```

Adapter 1 được sử dụng làm WAN.

#### Adapter 2 - LAN

```text
Enable Network Adapter: Yes
Attached to: Host-only Adapter
Name: vboxnet0
```

Mạng LAN:

```text
10.0.0.0/8
```

#### Adapter 3 - DMZ

```text
Enable Network Adapter: Yes
Attached to: Internal Network
Name: dmz-net
```

Mạng DMZ:

```text
172.16.0.0/16
```

---

### 5.5. Cấu hình Host-only Network

VirtualBox ban đầu tạo `vboxnet0` với địa chỉ:

```text
192.168.56.1/24
```

Theo mô hình bài Lab cần chuyển sang:

```text
IPv4 Address: 10.0.0.100
Subnet Mask: 255.0.0.0
DHCP Server: Disabled
```

Cho phép VirtualBox sử dụng mạng `10.0.0.0/8`:

```bash
sudo mkdir -p /etc/vbox
echo "* 10.0.0.0/8" | sudo tee /etc/vbox/networks.conf
```

Cấu hình `vboxnet0`:

```bash
VBoxManage hostonlyif ipconfig vboxnet0 --ip=10.0.0.100 --netmask=255.0.0.0
```

Kiểm tra:

```bash
ip addr show vboxnet0
```

Kết quả:

```text
inet 10.0.0.100/8
```

Kiểm tra DHCP:

```bash
VBoxManage list dhcpservers
```

Kết quả:

```text
Enabled: No
```

Như vậy Host-only được cấu hình:

```text
vboxnet0 = 10.0.0.100/8
DHCP = Disabled
```

---

### 5.6. Cài đặt pfSense

Khởi động máy ảo pfSense từ ISO.

Trong trình cài đặt thực hiện:

```text
Install
→ Auto (UFS)
→ Entire Disk
→ GPT
→ Finish
→ Reboot
```

Sau khi cài đặt hoàn tất, tháo ISO khỏi ổ đĩa quang để tránh boot lại trình cài đặt.

pfSense sau đó khởi động thành công từ ổ cứng ảo.

---

### 5.7. Cấu hình LAN trên console pfSense

Tại console pfSense chọn:

```text
2) Set interface(s) IP address
```

Chọn LAN và cấu hình:

```text
IPv4 Address: 10.0.0.1
Subnet bit count: 8
IPv4 Gateway: None
IPv6: Không sử dụng
DHCP Server: No
WebConfigurator: HTTPS
```

Sau khi hoàn thành:

```text
LAN (lan) -> em1 -> v4: 10.0.0.1/8
```

Giao diện quản trị WebGUI:

```text
https://10.0.0.1
```

---

### 5.8. Kiểm tra kết nối từ máy thật đến pfSense

Trên máy thật Pop!_OS:

```bash
ping 10.0.0.1
```

Kết quả:

```text
12 packets transmitted
12 received
0% packet loss
```

Kết luận: máy thật Pop!_OS đã giao tiếp thành công với pfSense thông qua mạng Host-only.

---

### 5.9. Truy cập giao diện WebGUI pfSense

Trên trình duyệt truy cập:

```text
https://10.0.0.1
```

Trang đăng nhập pfSense xuất hiện.

Sau khi đăng nhập, thực hiện Setup Wizard.

Tại bước Configure LAN Interface:

```text
LAN IP Address: 10.0.0.1
Subnet Mask: 8
```

Giữ nguyên cấu hình và tiếp tục.

Sau khi hoàn tất Setup Wizard, truy cập thành công Dashboard của pfSense.

---

### 5.10. Cấu hình vùng DMZ

Trên pfSense truy cập:

```text
Interfaces → Assignments
```

Thêm card mạng thứ ba:

```text
em2 → OPT1
```

Sau đó vào:

```text
Interfaces → OPT1
```

Cấu hình:

```text
Enable Interface: Yes
Description: DMZ
IPv4 Configuration Type: Static IPv4
IPv4 Address: 172.16.0.1
Subnet Mask: /16
IPv4 Upstream Gateway: None
```

Sau khi Save và Apply:

```text
DMZ = 172.16.0.1/16
```

---

### 5.11. Tạo máy DMZ-Web

Tạo máy ảo:

```text
VM Name: DMZ-Web
```

Hệ điều hành thực tế sử dụng:

```text
Windows Server 2025 Standard Evaluation
```

Cấu hình máy ảo:

```text
RAM: 2048 MB
CPU: 2 vCPU
Virtual Disk: khoảng 30 GB
```

Card mạng:

```text
Enable Network Adapter: Yes
Attached to: Internal Network
Name: dmz-net
```

DMZ-Web và interface DMZ của pfSense cùng nằm trong Internal Network:

```text
dmz-net
```

---

### 5.12. Cài đặt Windows Server trên DMZ-Web

Khởi động máy DMZ-Web và cài Windows Server.

Sau khi hoàn tất:

- Đặt mật khẩu Administrator.
- Đăng nhập Windows Server.
- Mở Windows PowerShell bằng quyền Administrator.

Kiểm tra card mạng:

```powershell
Get-NetAdapter
```

Kết quả card mạng:

```text
Ethernet
```

---

### 5.13. Cấu hình IP cho DMZ-Web

Tắt DHCP:

```powershell
Set-NetIPInterface -InterfaceAlias "Ethernet" -Dhcp Disabled
```

Cấu hình địa chỉ IPv4:

```powershell
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 172.16.0.2 -PrefixLength 16 -DefaultGateway 172.16.0.1
```

Kiểm tra:

```powershell
ipconfig
```

Kết quả:

```text
IPv4 Address: 172.16.0.2
Subnet Mask: 255.255.0.0
Default Gateway: 172.16.0.1
```

DNS chưa cấu hình ở bước này.

---

### 5.14. Kiểm tra kết nối DMZ ban đầu

Thực hiện:

```powershell
ping 172.16.0.1
```

Kết quả ban đầu:

```text
Request timed out
100% packet loss
```

Nguyên nhân là interface DMZ mới tạo chưa có Firewall Rule cho phép lưu lượng từ DMZ đi qua pfSense.

---

### 5.15. Tạo Firewall Rule cho vùng DMZ

Trên pfSense vào:

```text
Firewall → Rules → DMZ
```

Tạo rule:

```text
Action: Pass
Interface: DMZ
Address Family: IPv4
Protocol: Any
Source: DMZ subnets
Destination: Any
Description: Pass DMZ net to Any
```

Sau khi Save và Apply, rule hiển thị:

```text
Pass | IPv4 | Any | DMZ subnets | Any
```

Rule này cho phép lưu lượng IPv4 từ vùng DMZ đi qua pfSense.

---

### 5.16. Kiểm tra Automatic Outbound NAT

Trên pfSense truy cập:

```text
Firewall → NAT → Outbound
```

Chế độ đang sử dụng:

```text
Automatic Outbound NAT rule generation
```

Trong phần Automatic Rules đã có các mạng:

```text
10.0.0.0/8
172.16.0.0/16
```

Điều này chứng minh pfSense đã tự động tạo Outbound NAT cho cả LAN và DMZ.

Không cần tạo Manual Outbound NAT ở bước cấu hình cơ bản.

---

## 6. Ảnh minh chứng

Các ảnh minh chứng đã chụp trong quá trình thực hành:

1. Kiểm tra địa chỉ IP và bảng định tuyến trên Pop!_OS.
2. Kiểm tra route sau khi tắt `vmnet8`.
3. Kiểm tra SHA-256 bộ cài pfSense.
4. Tạo máy ảo pfSense.
5. Cấu hình RAM, CPU và ổ cứng pfSense.
6. Cấu hình Adapter 1 - WAN.
7. Cấu hình Adapter 2 - LAN.
8. Cấu hình Adapter 3 - DMZ.
9. Cấu hình Host-only `10.0.0.100/8`.
10. Console pfSense hiển thị LAN `10.0.0.1/8`.
11. Ping từ Pop!_OS tới `10.0.0.1` thành công.
12. Trang đăng nhập pfSense.
13. Setup Wizard cấu hình LAN `10.0.0.1/8`.
14. Dashboard pfSense.
15. Gán interface OPT1.
16. Cấu hình DMZ `172.16.0.1/16`.
17. Cấu hình card Internal Network `dmz-net` cho DMZ-Web.
18. Cấu hình IP `172.16.0.2/16` cho DMZ-Web.
19. Kết quả ping DMZ ban đầu bị timeout.
20. Firewall Rule `Pass DMZ net to Any`.
21. Automatic Outbound NAT có cả LAN và DMZ.

---

## 7. Kết quả đạt được

Sau phần thực hành hiện tại, em đã hoàn thành:

- Cài đặt pfSense CE 2.7.2 trên Oracle VirtualBox.
- Cấu hình ba vùng WAN, LAN và DMZ.
- Cấu hình WAN bằng Bridged Adapter.
- Cấu hình LAN pfSense tại `10.0.0.1/8`.
- Cấu hình Host-only của máy thật tại `10.0.0.100/8`.
- Tắt DHCP trên Host-only.
- Kiểm tra kết nối Pop!_OS → pfSense thành công.
- Truy cập pfSense WebGUI thành công.
- Hoàn thành Setup Wizard.
- Cấu hình DMZ tại `172.16.0.1/16`.
- Tạo máy DMZ-Web.
- Cấu hình DMZ-Web tại `172.16.0.2/16`.
- Đặt Gateway của DMZ-Web là `172.16.0.1`.
- Tạo Firewall Rule `Pass DMZ net to Any`.
- Xác nhận Automatic Outbound NAT có LAN `10.0.0.0/8`.
- Xác nhận Automatic Outbound NAT có DMZ `172.16.0.0/16`.

---

## 8. Nội dung tiếp tục thực hiện

Các bước tiếp theo của Lab 5:

- Kiểm tra lại `ping 172.16.0.1` sau khi tạo DMZ Rule.
- Kiểm tra kết nối Internet từ DMZ-Web bằng `ping 8.8.8.8`.
- Cấu hình DNS `8.8.8.8` cho DMZ-Web.
- Kiểm tra DNS bằng `Resolve-DnsName`.
- Cài đặt IIS trên DMZ-Web.
- Kiểm tra IIS bằng `curl.exe http://localhost`.
- Chuẩn hóa LAN Firewall Rules.
- Disable Default allow LAN to any IPv4/IPv6.
- Giữ Anti-Lockout Rule.
- Reset States.
- Tạo rule nền tảng `Pass LAN net → Any`.
- Thực hiện tình huống chặn ICMP nhưng vẫn cho Web/DNS.
- Thực hiện tình huống chỉ cho một host cụ thể ra Internet.
- Thực hiện tình huống cô lập DMZ khỏi LAN.
- Cấu hình Port Forward WAN → DMZ.
- Bật Firewall Logging.
- Kiểm tra và phân tích Firewall Log.

---

## 9. Kết luận

Trong Lab 5, em đã xây dựng thành công mô hình Firewall pfSense trên Oracle VirtualBox.

pfSense được cấu hình với:

```text
WAN: DHCP
LAN: 10.0.0.1/8
DMZ: 172.16.0.1/16
```

Máy thật Pop!_OS sử dụng:

```text
10.0.0.100/8
```

để quản trị pfSense thông qua Host-only Network.

Máy DMZ-Web được cấu hình:

```text
IPv4: 172.16.0.2/16
Gateway: 172.16.0.1
```

Firewall Rule cho DMZ đã được tạo:

```text
Pass DMZ net → Any
```

Automatic Outbound NAT của pfSense đã nhận diện:

```text
LAN: 10.0.0.0/8
DMZ: 172.16.0.0/16
```


