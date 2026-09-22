LAB 3 – NHẬN DIỆN VÀ ỨNG PHÓ CÁC MỐI ĐE DỌA ĐẾN AN TOÀN THÔNG TIN
LINK YTB : https://www.youtube.com/watch?v=NLp0mEGP0gg
1. Thông tin sinh viên

Họ và tên: Hoàng Công Trường Lộc

Mã số sinh viên: 1150080063

Môn học: An toàn và Bảo mật Hệ thống Thông tin

Bài thực hành: Lab 3 – Nhận diện và ứng phó các mối đe dọa đến an toàn thông tin

Repository: https://github.com/Cacao123-web/LAB_AT_BMHTTT



2. Mục tiêu

Phân biệt tài sản, lỗ hổng, mối đe dọa, rủi ro và tấn công.

Nhận diện các nguy cơ liên quan đến mã độc, mật khẩu, persistence, lưu lượng mạng và lừa đảo.

Thu thập ảnh chụp, log và kết quả thực hành để đối chiếu.

Thực hiện quy trình: Baseline → Observe → Detect → Contain → Recover → Verify.



3. Môi trường thực hành

Thành phần

Thông tin thực tế đã xác nhận

Thông tin cần bổ sung

Máy thật

Linux Pop!_OS

Phiên bản hệ điều hành

Phần mềm ảo hóa

VMware Workstation

Phiên bản trong Help → About

Máy ảo

LAB3_Windows11, chạy Windows 11

Edition, version và OS build trong winver

Shell

Windows PowerShell, sử dụng quyền Administrator ở bước yêu cầu

Phiên bản PowerShell

Bảo vệ endpoint

Microsoft Defender; Antivirus, Real-time protection và Tamper Protection đều bật tại thời điểm kiểm tra

Phiên bản cơ sở dữ liệu nhận diện

Thư mục làm việc

C:\LAB3

—

Python, Wireshark/Npcap

Phiên bản thực tế và ảnh kiểm tra

Sysmon, Autoruns, Process Explorer

Phiên bản thực tế và ảnh kiểm tra

Mạng và snapshot


4. Cách dựng lại môi trường


Chuẩn bị máy ảo Windows trên VMware, cấu hình mạng Host-only và tạo snapshot sạch.

Trong Windows, mở PowerShell bằng Run as administrator; tạo các thư mục C:\LAB3\Evidence, Tools, Downloads, Assets và lưu thời điểm bắt đầu.

Chép gói LAB3_Threats_Assets.zip do giảng viên cung cấp vào C:\LAB3\Downloads. Kiểm tra SHA-256 với manifest của đúng gói được cấp trước khi giải nén; kiểm tra thư mục C:\LAB3\lab3_assets sau khi giải nén.

Chuẩn bị Python, Wireshark/Npcap, Sysmon, Autoruns và Process Explorer theo đề; ghi lại phiên bản thực tế.

Giữ Microsoft Defender và Tamper Protection bật; thu baseline hệ điều hành, Defender, firewall, mạng và tiến trình trước khi tạo tình huống.

Thực hiện từng tình huống theo đề, lưu ảnh và output vào C:\LAB3\Evidence. Sau khi thu đủ bằng chứng, cleanup, kiểm tra lại và lập danh sách SHA-256.



