# LAB 4 - KHẢO SÁT VÀ ĐÁNH GIÁ BỀ MẶT MẠNG BẰNG NMAP

## 1. Thông tin sinh viên

- Họ và tên: Hoàng Công Trường Lộc
- MSSV: 1150080063
- Môn học: An toàn và Bảo mật Hệ thống Thông tin
- Tên Lab: Lab 4 - Khảo sát và đánh giá bề mặt mạng bằng Nmap

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

## 3. Cách dựng môi trường

Sử dụng Oracle VirtualBox trên máy thật Pop!_OS để xây dựng môi trường thực hành. Trong VirtualBox tạo mạng Host-Only có tên `vboxnet0`, sử dụng dải mạng `192.168.56.0/24`.

Kali Linux được sử dụng làm máy quét. Adapter 1 của Kali được cấu hình ở chế độ Host-only Adapter, Name là `vboxnet0`. Sau khi cấu hình, Kali Linux nhận được địa chỉ IP `192.168.56.102/24`.

Metasploitable 2 được sử dụng làm máy đích. Máy ảo được tạo từ file ổ đĩa có sẵn `Metasploitable.vmdk`. Adapter 1 của Metasploitable 2 được cấu hình ở chế độ Host-only Adapter, Name là `vboxnet0`. Metasploitable 2 nhận được địa chỉ IP `192.168.56.101/24`.

Trước khi thực hành, tạo snapshot `Before-Lab4` cho cả Kali Linux và Metasploitable 2 để có thể khôi phục lại trạng thái ban đầu khi cần.

Sau khi hoàn tất cấu hình mạng, kiểm tra kết nối từ Kali Linux đến Metasploitable 2 bằng lệnh:

`ping -c 4 192.168.56.101`

Kết quả nhận đủ 4 gói tin và không có packet loss, chứng minh hai máy có thể giao tiếp với nhau trong mạng Host-Only.

## 4. Các tình huống đã thực hiện

### 4.1. Kiểm tra địa chỉ IP của Kali Linux

Lệnh thực hiện:

`ip -br addr`

Kết quả:

- Interface: `eth0`
- Trạng thái: UP
- Địa chỉ IP: `192.168.56.102/24`

Kết quả: PASS

### 4.2. Kiểm tra địa chỉ IP của Metasploitable 2

Lệnh thực hiện:

`ifconfig`

Kết quả:

- Interface: `eth0`
- Địa chỉ IP: `192.168.56.101`
- Netmask: `255.255.255.0`

Kết quả: PASS

### 4.3. Kiểm tra kết nối từ Kali Linux đến Metasploitable 2

Lệnh thực hiện:

`ping -c 4 192.168.56.101`

Kết quả:

- 4 packets transmitted
- 4 packets received
- 0% packet loss

Kali Linux kết nối thành công đến Metasploitable 2.

Kết quả: PASS

### 4.4. Kiểm tra phiên bản Nmap trên Kali Linux

Lệnh thực hiện:

`nmap --version`

Kết quả:

Nmap version 7.99

Kết quả: PASS

### 4.5. Phát hiện các host đang hoạt động trong mạng Host-Only

Lệnh thực hiện:

`nmap -sn 192.168.56.0/24`

Kết quả:

Nmap phát hiện 4 host đang hoạt động trong mạng `192.168.56.0/24`.

Trong đó phát hiện Metasploitable 2 tại địa chỉ `192.168.56.101` và Kali Linux tại địa chỉ `192.168.56.102`. Một số địa chỉ MAC được Nmap nhận diện là Oracle VirtualBox virtual NIC.

Kết quả: PASS

## 5. Kết quả PASS/FAIL

| Nội dung kiểm tra | Kết quả |
|---|---|
| Tạo mạng Host-Only vboxnet0 | PASS |
| Cấu hình Kali Linux vào Host-Only | PASS |
| Cấu hình Metasploitable 2 vào Host-Only | PASS |
| Tạo snapshot Before-Lab4 | PASS |
| Kiểm tra IP Kali Linux | PASS |
| Kiểm tra IP Metasploitable 2 | PASS |
| Ping Kali đến Metasploitable 2 | PASS |
| Kiểm tra phiên bản Nmap | PASS |
| Host Discovery bằng nmap -sn | PASS |

## 6. Lỗi gặp phải và cách khắc phục

### Lỗi 1: Kali Linux không nhận được địa chỉ IPv4 Host-Only

Hiện tượng:

Khi chạy lệnh `ip -br addr`, interface mạng của Kali ở trạng thái UP nhưng chưa nhận được địa chỉ IPv4 thuộc dải `192.168.56.0/24`.

Nguyên nhân:

Kali Linux còn bật Adapter 2 ở chế độ Internal Network với tên `LabNet`, trong khi bài Lab 4 sử dụng mạng Host-Only `vboxnet0`.

Cách khắc phục:

Tắt Adapter 2 và chỉ giữ Adapter 1 ở chế độ Host-only Adapter, Name là `vboxnet0`. Sau khi khởi động lại Kali Linux, máy nhận được địa chỉ IP `192.168.56.102/24`.

Kết quả: PASS

### Lỗi 2: Không tải được Metasploitable 2 bằng aria2 với URL ban đầu

Hiện tượng:

Khi tải Metasploitable 2 bằng aria2 từ đường dẫn SourceForge ban đầu, chương trình báo lỗi HTTP 403.

Cách khắc phục:

Chuyển sang đường dẫn tải trực tiếp của SourceForge và sử dụng tải nhiều kết nối. Sau đó file `metasploitable-linux-2.0.0.zip` được tải thành công và giải nén để lấy file `Metasploitable.vmdk`.

Kết quả: PASS

### Lỗi 3: Nhập nhầm tùy chọn Nmap

Trong quá trình thực hành đã nhập lệnh:

`nmap -sn 192.168.56.101`

trong khi cần thực hiện TCP Connect Scan bằng tùy chọn `-sT`.

Cách khắc phục:

Kiểm tra lại cú pháp trước khi chạy lệnh và phân biệt:

`-sn`: dùng để Host Discovery.

`-sT`: dùng để TCP Connect Scan.

## 7. Kết luận

Môi trường thực hành Lab 4 đã được xây dựng thành công trên Oracle VirtualBox. Kali Linux được sử dụng làm máy quét và Metasploitable 2 được sử dụng làm máy đích.

Hai máy được kết nối trong mạng Host-Only `192.168.56.0/24`, không sử dụng Bridged Network để đảm bảo môi trường thực hành được cách ly với mạng thật.

Kali Linux có địa chỉ IP `192.168.56.102/24` và Metasploitable 2 có địa chỉ IP `192.168.56.101/24`. Hai máy đã ping thành công với packet loss 0%.

Nmap phiên bản 7.99 hoạt động bình thường trên Kali Linux. Host Discovery đã được thực hiện thành công và phát hiện các host đang hoạt động trong mạng Host-Only.

