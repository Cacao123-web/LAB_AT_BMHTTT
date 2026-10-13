
# LAB 6 - XÁC THỰC ACTIVE DIRECTORY VÀ CHÍNH SÁCH TRUY CẬP TRÊN pfSense

## 1. Thông tin sinh viên

- **Họ và tên:** Hoàng Công Trường Lộc
- **Mã số sinh viên:** 1150080063
- **Môn học:** An toàn và Bảo mật Hệ thống Thông tin
- **Bài thực hành:** Lab 6
- **Hệ điều hành máy thật:** Pop!\_OS Linux
- **Phần mềm ảo hóa:** Oracle VirtualBox
- **Firewall:** pfSense CE 2.7.2
- **LINKYTB:** https://youtu.be/qaFnM6HgwTM

---

## 2. Tên bài Lab

**Lab 6 - Xác thực Active Directory và chính sách truy cập trên pfSense**

---

## 3. Mục tiêu

Trong bài Lab 6, em thực hiện xây dựng môi trường mạng sử dụng pfSense làm Firewall, kết hợp Active Directory để xác thực người dùng và triển khai các chính sách kiểm soát truy cập.

Các mục tiêu chính:

- Xây dựng mô hình mạng riêng cho Lab 6 trên Oracle VirtualBox.
- Tạo NAT Network và Host-Only Network.
- Cấu hình pfSense với bốn interface WAN, LAN, MGMT và DMZ.
- Thiết lập địa chỉ IPv4 cho từng interface.
- Kiểm tra kết nối giữa máy thật Pop!\_OS và pfSense.
- Truy cập và quản trị pfSense bằng WebGUI.
- Triển khai Windows Server làm Domain Controller.
- Cài đặt Active Directory Domain Services và DNS Server.
- Tạo domain `lab6.test` và các tài khoản người dùng.
- Tích hợp Active Directory với pfSense thông qua LDAP.
- Cấu hình Captive Portal để xác thực tài khoản domain.
- Tạo Firewall Alias cho máy người dùng và website mô phỏng.
- Thiết lập Schedule giới hạn truy cập theo thời gian.
- Xây dựng các chính sách Firewall theo từng kịch bản.
- Kiểm tra Firewall Log và State Table.
- Phân tích kết quả kiểm thử và đánh giá chính sách bảo mật.

---

## 4. Mô hình mạng

### 4.1. Bảng địa chỉ IP

| Thiết bị | Interface | Địa chỉ IP | Subnet Mask | Gateway |
|----------|-----------|------------|-------------|---------|
| pfSense | WAN | DHCP | 255.255.255.0 (/24) | 192.168.250.1 |
| pfSense | LAN | 10.6.0.1 | 255.255.255.0 (/24) | - |
| pfSense | MGMT | 192.168.56.254 | 255.255.255.0 (/24) | - |
| pfSense | DMZ | 172.16.6.1 | 255.255.255.0 (/24) | - |
| Máy thật Pop!\_OS | Host-Only | 192.168.56.1 | 255.255.255.0 (/24) | Không đặt |
| DC01 | LAN | 10.6.0.10 | 255.255.255.0 (/24) | 10.6.0.1 |
| Client-A | LAN | 10.6.0.21 | 255.255.255.0 (/24) | 10.6.0.1 |
| Client-B | LAN | 10.6.0.22 | 255.255.255.0 (/24) | 10.6.0.1 |
| DMZ-WEB | DMZ | 172.16.6.10 | 255.255.255.0 (/24) | 172.16.6.1 |

### 4.2. Các vùng mạng

Mô hình Lab 6 gồm bốn vùng mạng chính:

- **WAN:** Kết nối pfSense với NAT Network `LAB6-WAN`, sử dụng dải `192.168.250.0/24`.
- **LAN:** Mạng nội bộ dành cho Domain Controller và các máy Client, sử dụng dải `10.6.0.0/24`.
- **MGMT:** Mạng quản trị riêng, sử dụng dải `192.168.56.0/24`, cho phép máy thật truy cập WebGUI của pfSense.
- **DMZ:** Vùng mạng dành cho máy chủ website mô phỏng, sử dụng dải `172.16.6.0/24`.

Việc phân chia các vùng mạng giúp tách biệt hoạt động quản trị, người dùng nội bộ và máy chủ dịch vụ.

---

## 5. Nội dung thực hiện

### 5.1. Kiểm tra cấu hình mạng trên máy thật Pop!\_OS

Trước khi cấu hình VirtualBox, tiến hành kiểm tra các interface mạng trên máy thật.

Mở Terminal và thực hiện:

```bash
ip -4 addr show
```

Kết quả kiểm tra trên máy thật ghi nhận một số địa chỉ:

```text
192.168.0.101/24
192.168.85.1/24
172.16.2.1/24
```

Trong đó:

- `192.168.0.101/24`: Địa chỉ mạng đang sử dụng trên máy thật.
- `192.168.85.1/24`: Mạng VMware vmnet1.
- `172.16.2.1/24`: Mạng VMware vmnet8.

Ngoài ra, tiến hành kiểm tra các mạng Host-Only đã được tạo trên VirtualBox.

```bash
VBoxManage list hostonlyifs
```

Ban đầu, VirtualBox hiển thị:

```text
Name:        vboxnet0
IPAddress:   10.0.0.100
NetworkMask: 255.0.0.0

Name:        vboxnet1
IPAddress:   192.168.57.1
NetworkMask: 255.255.255.0
```

Do bài Lab 6 yêu cầu mạng quản trị sử dụng dải `192.168.56.0/24`, tiến hành điều chỉnh mạng Host-Only phù hợp.

---

### 5.2. Cấu hình Host-Only Network

Truy cập:

```text
Oracle VirtualBox → Network → Host-only Networks
```

Cấu hình mạng Host-Only phục vụ quản trị pfSense.

Thông số:

| Thông số | Giá trị |
|----------|---------|
| Network Name | vboxnet1 |
| IPv4 Address | 192.168.56.1 |
| IPv4 Network Mask | 255.255.255.0 |
| DHCP Server | Disabled |

Trong quá trình cấu hình bằng Terminal, sử dụng lệnh:

```bash
VBoxManage hostonlyif ipconfig vboxnet1 --ip 192.168.56.1 --netmask 255.255.255.0
```

Ban đầu, VirtualBox trả về lỗi:

```text
VBoxManage: error: Code E_ACCESSDENIED (0x80070005)
Access denied
```

Lỗi xuất hiện khi VirtualBox từ chối thao tác cập nhật địa chỉ IP cho Host-Only Adapter.

Tiến hành kiểm tra cấu hình giới hạn mạng của VirtualBox:

```bash
cat /etc/vbox/networks.conf
```

Sau khi xử lý cấu hình giới hạn mạng Host-Only, tiếp tục cấu hình lại địa chỉ IP.

Kiểm tra:

```bash
VBoxManage list hostonlyifs
```

Kết quả cuối cùng, mạng Host-Only được sử dụng cho Lab 6 với địa chỉ:

```text
IP Address:   192.168.56.1
Subnet Mask:  255.255.255.0
DHCP Server:  Disabled
```

**Kết quả:** Hoàn thành cấu hình Host-Only Network phục vụ kết nối giữa máy thật Pop!\_OS và interface MGMT của pfSense.

---

### 5.3. Tạo NAT Network cho WAN

Trong VirtualBox, truy cập:

```text
Network → NAT Networks → Create
```

Tạo NAT Network mới với thông số:

| Thông số | Giá trị |
|----------|---------|
| Network Name | LAB6-WAN |
| IPv4 Prefix | 192.168.250.0/24 |
| DHCP Server | Enabled |

Mạng NAT được sử dụng để cấp địa chỉ IPv4 cho interface WAN của pfSense và tạo đường kết nối ra mạng bên ngoài.

**Kết quả:** NAT Network `LAB6-WAN` được tạo và sử dụng cho Adapter 1 của pfSense.

---

### 5.4. Cấu hình bốn card mạng cho pfSense

Mở Oracle VirtualBox Manager.

Chọn máy ảo pfSense và truy cập:

```text
Settings → Network
```

Tiến hành cấu hình bốn Adapter:

| Adapter | Attached to | Network Name |
|---------|-------------|--------------|
| Adapter 1 | NAT Network | LAB6-WAN |
| Adapter 2 | Host-only Adapter | vboxnet1 |
| Adapter 3 | Internal Network | lab6-lan |
| Adapter 4 | Internal Network | lab6-dmz |

Các Adapter được bật chức năng:

```text
Enable Network Adapter
Virtual Cable Connected
```

Trong quá trình cấu hình, Adapter 4 ban đầu không thể thay đổi do trạng thái máy ảo chưa cho phép chỉnh sửa.

Sau khi xử lý trạng thái máy ảo, tiến hành cấu hình Adapter 4 thành Internal Network với tên `lab6-dmz`.

---

### 5.5. Kiểm tra card mạng bằng VBoxManage

Sử dụng Terminal trên máy Pop!\_OS để kiểm tra cấu hình mạng của máy ảo pfSense.

Thực hiện:

```bash
VBoxManage showvminfo "pfSense" --machinereadable | grep -E '^(nic[1-4]|macaddress[1-4]|hostonlyadapter[1-4]|intnet[1-4]|nat-network[1-4])='
```

Kết quả:

```text
nat-network1="LAB6-WAN"
macaddress1="080027271930"
nic1="natnetwork"

hostonlyadapter2="vboxnet1"
macaddress2="08002781BD6C"
nic2="hostonly"

intnet3="lab6-lan"
macaddress3="080027B63FDB"
nic3="intnet"

intnet4="lab6-dmz"
macaddress4="080027E195E5"
nic4="intnet"
```

Từ kết quả trên, xác định được mỗi Adapter được gán đúng chế độ mạng theo mô hình Lab 6.

---

### 5.6. Khởi động pfSense

Khởi động máy ảo pfSense trên VirtualBox.

Hệ thống sử dụng:

```text
pfSense CE 2.7.2-RELEASE (amd64)
```

Sau khi khởi động, truy cập giao diện Console để kiểm tra các interface mạng.

Tại Console, chọn:

```text
1) Assign Interfaces
```

Bỏ qua cấu hình VLAN bằng cách nhập:

```text
n
```

pfSense nhận diện bốn card mạng:

```text
em0
em1
em2
em3
```

---

### 5.7. Gán các interface cho pfSense

Thực hiện gán interface theo thứ tự:

```text
WAN  → em0
LAN  → em2
OPT1 → em1
OPT2 → em3
```

Trong đó:

- `em0`: Kết nối NAT Network.
- `em2`: Kết nối mạng LAN-DOMAIN.
- `em1`: Kết nối Host-Only Network.
- `em3`: Kết nối mạng DMZ.

Sau khi hoàn tất, xác nhận cấu hình bằng lựa chọn:

```text
y
```

Việc đối chiếu địa chỉ MAC được sử dụng để xác định chính xác card mạng tương ứng với từng interface.

---

### 5.8. Cấu hình địa chỉ IPv4 cho LAN

Từ Console pfSense, chọn:

```text
2) Set interface(s) IP address
```

Chọn interface LAN.

Cấu hình:

```text
IPv4 Configuration: Static
IPv4 Address: 10.6.0.1
Subnet Bit Count: 24
Upstream Gateway: None
IPv6 DHCP: No
DHCP Server: Disabled
WebConfigurator Protocol: HTTPS
```

Sau khi lưu cấu hình, kiểm tra kết quả:

```text
LAN (lan) → em2 → 10.6.0.1/24
```

Địa chỉ `10.6.0.1` được sử dụng làm Default Gateway cho các máy trong mạng LAN-DOMAIN.

**Kết quả:** Cấu hình IP LAN thành công.

---

### 5.9. Cấu hình địa chỉ IPv4 cho MGMT

Tiếp tục chọn:

```text
2) Set interface(s) IP address
```

Chọn interface OPT1 tương ứng với card `em1`.

Cấu hình:

```text
IPv4 Configuration: Static
IPv4 Address: 192.168.56.254
Subnet Bit Count: 24
Upstream Gateway: None
IPv6 DHCP: No
DHCP Server: Disabled
WebConfigurator Protocol: HTTPS
```

Sau khi áp dụng, hệ thống thông báo:

```text
The IPv4 OPT1 address has been set to 192.168.56.254/24
```

Đồng thời cung cấp địa chỉ quản trị:

```text
https://192.168.56.254/
```

Đây là interface được sử dụng để quản trị pfSense từ máy thật Pop!\_OS.

**Kết quả:** Cấu hình IP MGMT thành công.

---

### 5.10. Cấu hình địa chỉ IPv4 cho DMZ

Tiếp tục sử dụng chức năng:

```text
2) Set interface(s) IP address
```

Chọn OPT2 tương ứng với card `em3`.

Cấu hình:

```text
IPv4 Configuration: Static
IPv4 Address: 172.16.6.1
Subnet Bit Count: 24
Upstream Gateway: None
IPv6 DHCP: No
DHCP Server: Disabled
WebConfigurator Protocol: HTTPS
```

Sau khi lưu, pfSense hiển thị:

```text
OPT2 (opt2) → em3 → 172.16.6.1/24
```

Địa chỉ này đóng vai trò Default Gateway cho vùng mạng DMZ.

**Kết quả:** Cấu hình IP DMZ thành công.

---

### 5.11. Truy cập giao diện WebGUI

Trên máy thật Pop!\_OS, mở trình duyệt Google Chrome.

Truy cập:

```text
https://192.168.56.254
```

Kết quả:

- Trình duyệt hiển thị trang đăng nhập pfSense.
- Có thể truy cập WebGUI thông qua mạng Host-Only.
- Đăng nhập thành công bằng tài khoản quản trị.
- Dashboard pfSense CE 2.7.2 hiển thị bình thường.

**Kết quả:** Kết nối giữa máy thật và giao diện quản trị pfSense hoạt động thành công.

---

### 5.12. Đổi tên các interface trên WebGUI

Sau khi đăng nhập Dashboard, truy cập:

```text
Interfaces → Interface Assignments
```

Ban đầu, các interface phụ còn sử dụng tên từ cấu hình cũ:

```text
DMZ  → em1
OPT2 → em3
```

Tiến hành đổi tên:

```text
DMZ (em1)  → MGMT
OPT2 (em3) → DMZ
```

Sau khi Save và Apply Changes, danh sách cuối cùng:

| Interface | Network Port |
|-----------|--------------|
| WAN | em0 |
| LAN | em2 |
| MGMT | em1 |
| DMZ | em3 |

**Kết quả:** Hoàn thành việc phân chia bốn interface theo đúng mô hình Lab 6.

---

### 5.13. Kiểm tra trạng thái WAN và LAN

Truy cập:

```text
Status → Interfaces
```

Tại mục WAN, ghi nhận:

```text
Interface: WAN (em0)
Status: Up
DHCP: Up
Gateway IPv4: 192.168.250.1
```

Tại mục LAN:

```text
Interface: LAN (em2)
Status: Up
IPv4 Address: 10.6.0.1
Subnet Mask: 255.255.255.0
```

Kết quả kiểm tra cho thấy interface WAN đã hoạt động và nhận thông tin Gateway thông qua DHCP từ NAT Network LAB6-WAN.

Interface LAN đã được cấu hình đúng địa chỉ IP theo yêu cầu.

**Kết quả:** Hoàn thành cấu hình mạng cơ bản trên pfSense.

---

## 6. Triển khai Windows Server và Active Directory

**Trạng thái: Chưa hoàn thành, thuộc giai đoạn tiếp theo của bài Lab.**

### 6.1. Cấu hình Domain Controller

Dự kiến sử dụng Windows Server 2019/2022 làm Domain Controller.

Thông số:

```text
Computer Name: DC01
IPv4 Address: 10.6.0.10
Subnet Mask: 255.255.255.0
Gateway: 10.6.0.1
DNS: 10.6.0.10
Domain: lab6.test
```

### 6.2. Cài đặt Active Directory Domain Services

Các bước triển khai dự kiến:

1. Cài đặt AD DS.
2. Cài đặt DNS Server.
3. Promote Server thành Domain Controller.
4. Tạo forest mới `lab6.test`.
5. Tạo OU `LAB6-Users`.
6. Tạo tài khoản người dùng.

Các tài khoản cần tạo:

```text
sv_a
sv_b
svc_pfsense_ldap
```

Trong đó, `svc_pfsense_ldap` được sử dụng làm Service Account kết nối LDAP giữa pfSense và Active Directory.

---

## 7. Tích hợp Active Directory với pfSense

**Trạng thái: Chưa kiểm thử.**

Truy cập:

```text
System → User Manager → Authentication Servers
```

Dự kiến tạo LDAP Authentication Server với thông số:

| Thông số | Giá trị |
|----------|---------|
| Descriptive Name | LAB6-AD |
| Type | LDAP |
| Hostname/IP | 10.6.0.10 |
| Port | 389 |
| Protocol Version | 3 |
| Search Scope | Entire Subtree |
| Base DN | DC=lab6,DC=test |
| Authentication Containers | OU=LAB6-Users,DC=lab6,DC=test |
| Bind DN | CN=svc_pfsense_ldap,CN=Users,DC=lab6,DC=test |
| User Naming Attribute | samAccountName |

Sau khi hoàn tất, sử dụng:

```text
Diagnostics → Authentication
```

để kiểm thử hai tài khoản:

```text
sv_a
sv_b
```

---

## 8. Cấu hình Captive Portal

**Trạng thái: Chưa kiểm thử.**

Truy cập:

```text
Services → Captive Portal
```

Cấu hình dự kiến:

```text
Zone Name: LAB6_USERS
Interface: LAN
Authentication Method: Authentication Backend
Authentication Server: LAB6-AD
```

Mục tiêu là yêu cầu người dùng đăng nhập bằng tài khoản Active Directory trước khi truy cập tài nguyên mạng.

---

## 9. Tạo Firewall Alias

**Trạng thái: Chưa thực hiện.**

Truy cập:

```text
Firewall → Aliases
```

Các Alias cần tạo:

| Alias | Type | Giá trị |
|-------|------|---------|
| CLIENT_A | Host | 10.6.0.21 |
| CLIENT_B | Host | 10.6.0.22 |
| POLICY_CLIENTS | Hosts | 10.6.0.21, 10.6.0.22 |
| GENERAL_SITE | Host | 172.16.6.10 |
| JOB_SITE | Host | 172.16.6.20 |
| SOCIAL_SITE | Host | 172.16.6.30 |
| RESTRICTED_SITE | Host | 172.16.6.40 |
| WEB_PORTS | Ports | 80, 443 |

---

## 10. Tạo Schedule

**Trạng thái: Chưa thực hiện.**

Truy cập:

```text
Firewall → Schedules
```

| Schedule | Thời gian | Mục đích |
|----------|-----------|----------|
| LUNCH_WINDOW | 11:30–13:00, Thứ 2–Thứ 6 | Cho phép Social Site |
| WORK_HOURS | 08:00–17:00, Thứ 2–Thứ 6 | Giới hạn dịch vụ theo giờ hành chính |
| TEST_NOW | Khoảng 10–15 phút | Kiểm thử nhanh Schedule |

---

## 11. Các kịch bản Firewall

**Trạng thái: Chưa kiểm thử thực tế.**

### 11.1. Kịch bản 1 - Chặn Client-A truy cập JOB_SITE

Mục tiêu:

- Chặn Client-A truy cập JOB_SITE.
- Vẫn cho phép Client-A truy cập GENERAL_SITE.

Rule:

```text
Block CLIENT_A → JOB_SITE TCP/80
Pass  CLIENT_A → GENERAL_SITE TCP/80
```

Kết quả mong đợi:

```text
Client-A → 172.16.6.10: Allow
Client-A → 172.16.6.20: Block
```

### 11.2. Kịch bản 2 - Chặn RESTRICTED_SITE

Mục tiêu:

Chặn cả Client-A và Client-B truy cập website thuộc nhóm Restricted.

Rule:

```text
Block POLICY_CLIENTS → RESTRICTED_SITE TCP/80
```

Kết quả mong đợi:

```text
Client-A → 172.16.6.40: Block
Client-B → 172.16.6.40: Block
```

### 11.3. Kịch bản 3 - SOCIAL_SITE theo giờ nghỉ

Mục tiêu:

Client-B chỉ được phép truy cập SOCIAL_SITE trong giờ nghỉ trưa.

Rule:

```text
Pass  CLIENT_B → SOCIAL_SITE TCP/80 + LUNCH_WINDOW
Block CLIENT_B → SOCIAL_SITE TCP/80
```

Kết quả mong đợi:

```text
Trong giờ nghỉ: Allow
Ngoài giờ nghỉ: Block
```

### 11.4. Kịch bản 4 - HTTP/HTTPS trong giờ hành chính

Mục tiêu:

Giới hạn truy cập dịch vụ web theo Schedule WORK_HOURS.

Rule:

```text
Pass  POLICY_CLIENTS → Any WEB_PORTS + WORK_HOURS
Block POLICY_CLIENTS → Any
```

Kết quả mong đợi:

- Cho phép HTTP/HTTPS trong giờ hành chính.
- Chặn những dịch vụ không được phép.
- Ngoài giờ, rule Pass theo Schedule không còn được áp dụng.

### 11.5. Kịch bản 5 - Cô lập DMZ khỏi LAN

Mục tiêu:

Ngăn máy chủ trong DMZ tự khởi tạo kết nối đến LAN.

Rule:

```text
Block DMZ net → LAN net
Pass  POLICY_CLIENTS → GENERAL_SITE TCP/80
```

Kết quả mong đợi:

```text
LAN → DMZ-WEB: Allow theo rule cho phép
DMZ-WEB → DC01: Block
```

### 11.6. Kịch bản 6 - So sánh Block và Reject

Mục tiêu:

Phân tích hành vi khi kết nối bị chặn bằng Block và Reject.

| Block | Reject |
|-------|--------|
| Bỏ gói tin | Gửi tín hiệu từ chối |
| Thường gây timeout | Thường phản hồi lỗi nhanh |
| Không chủ động trả thông báo từ chối | Chủ động từ chối kết nối |

### 11.7. Kịch bản 7 - Firewall Log và State Table

Mục tiêu:

Quan sát và phân tích các kết nối được xử lý bởi pfSense.

Các bước:

1. Bật Logging cho các rule quan trọng.
2. Tạo lưu lượng kiểm thử.
3. Kiểm tra Firewall Log.
4. Xác định Source, Destination, Protocol và Action.
5. Kiểm tra State Table.
6. Đối chiếu kết quả với chính sách đã cấu hình.

---

## 12. Bảng kết quả dự kiến

| Nguồn | Đích | Kết quả mong đợi | Kết quả thực tế |
|-------|------|------------------|-----------------|
| Client-A | GENERAL_SITE | Allow | Chưa kiểm thử |
| Client-A | JOB_SITE | Block | Chưa kiểm thử |
| Client-A | RESTRICTED_SITE | Block | Chưa kiểm thử |
| Client-B | SOCIAL_SITE trong giờ nghỉ | Allow | Chưa kiểm thử |
| Client-B | SOCIAL_SITE ngoài giờ nghỉ | Block | Chưa kiểm thử |
| Client-B | RESTRICTED_SITE | Block | Chưa kiểm thử |
| DMZ-WEB | DC01 | Block | Chưa kiểm thử |

---

## 13. Kết quả hiện tại

Sau quá trình thực hành, em đã hoàn thành các nội dung:

- Tạo mạng NAT Network riêng cho Lab 6.
- Cấu hình Host-Only Network trên VirtualBox.
- Cấu hình đầy đủ bốn Adapter cho pfSense.
- Kiểm tra địa chỉ MAC của từng Adapter.
- Gán các interface WAN, LAN, MGMT và DMZ.
- Thiết lập địa chỉ IPv4 tĩnh cho LAN, MGMT và DMZ.
- Truy cập WebGUI pfSense từ máy thật Pop!\_OS.
- Đăng nhập thành công vào Dashboard.
- Đổi tên các interface đúng theo mô hình.
- Kiểm tra WAN và DHCP ở trạng thái Up.
- Kiểm tra LAN hoạt động với địa chỉ 10.6.0.1/24.

Các nội dung Active Directory, LDAP, Captive Portal và Firewall Policy đang được tiếp tục triển khai.

---

## 14. Kết luận

Trong bài thực hành Lab 6, em đã tiến hành xây dựng môi trường mạng sử dụng pfSense CE 2.7.2 trên Oracle VirtualBox với bốn interface WAN, LAN, MGMT và DMZ.

Thông qua việc cấu hình NAT Network, Host-Only Network và Internal Network, hệ thống đã phân chia được các vùng mạng riêng biệt phục vụ quản trị, kết nối mạng nội bộ và triển khai dịch vụ trong DMZ.

Các interface LAN, MGMT và DMZ đã được cấu hình địa chỉ IP tĩnh theo mô hình yêu cầu. Đồng thời, em đã truy cập và đăng nhập thành công vào WebGUI pfSense từ máy thật Pop!\_OS, xác nhận khả năng quản trị Firewall qua mạng Host-Only.

Giai đoạn tiếp theo của Lab 6 sẽ triển khai Active Directory, LDAP Authentication, Captive Portal và các chính sách Firewall nhằm kiểm soát truy cập theo máy người dùng, địa chỉ đích, cổng dịch vụ và thời gian.

---

## 15. Tài liệu tham khảo

1. Tài liệu thực hành Lab 6 - Xác thực Active Directory và chính sách truy cập trên pfSense.
2. [Netgate pfSense Documentation](https://docs.netgate.com/pfsense/en/latest/)
3. [Microsoft Active Directory Documentation](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/)
4. [Oracle VirtualBox Documentation](https://www.virtualbox.org/manual/)
5. [Nginx Documentation](https://nginx.org/en/docs/)

---

**LAB 6 - THỰC HÀNH AN TOÀN HỆ THỐNG THÔNG TIN**

