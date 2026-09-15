# LAB 1 - AN TOÀN VÀ BẢO MẬT HỆ THỐNG THÔNG TIN

## 1. Thông tin sinh viên

- **Họ và tên:** Hoàng Công Trường Lộc
- **Mã số sinh viên:** 1150080063
- **Môn học:** An toàn và Bảo mật Hệ thống Thông tin
- **Bài thực hành:** Lab 1

## 2. Nội dung đã thực hiện

Trong bài Lab 1, em đã thực hiện các nội dung sau:

- Chuẩn bị môi trường thực hành trên máy ảo.
- Sử dụng hệ điều hành Kali Linux và Ubuntu Server.
- Kiểm tra địa chỉ IP của các máy.
- Cấu hình mạng để các máy có thể giao tiếp với nhau.
- Sử dụng lệnh `ping` để kiểm tra kết nối mạng.
- Cài đặt và kiểm tra dịch vụ SSH.
- Thực hiện kết nối SSH giữa các máy trong hệ thống.
- Kiểm tra trạng thái hoạt động của dịch vụ SSH.
- Ghi lại hình ảnh và video quá trình thực hiện bài Lab.


## 3. Kết quả thực hiện

Sau khi hoàn thành bài thực hành:

- Các máy trong môi trường Lab đã kết nối mạng thành công.
- Các máy có thể `ping` qua lại với nhau.
- Dịch vụ SSH hoạt động bình thường.
- Có thể thực hiện kết nối SSH từ máy này sang máy khác.
- Hoàn thành các yêu cầu kiểm tra của bài Lab.
- Các bước thực hiện được lưu lại bằng hình ảnh và video để phục vụ việc kiểm tra.


## 4. Một số lệnh đã sử dụng

### Kiểm tra địa chỉ IP

```bash
ip a
```

### Kiểm tra kết nối mạng

```bash
ping <IP_máy_đích>
```

Ví dụ:

```bash
ping 192.168.1.10
```

### Kiểm tra trạng thái dịch vụ SSH

```bash
sudo systemctl status ssh
```

### Khởi động dịch vụ SSH

```bash
sudo systemctl start ssh
```

### Kết nối SSH đến máy khác

```bash
ssh username@IP
```

Ví dụ:

```bash
ssh user@192.168.1.10
```

