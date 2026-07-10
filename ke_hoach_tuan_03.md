# KẾ HOẠCH TUẦN 3 & PHÂN TÍCH MỐI ĐE DỌA STRIDE
## DỰ ÁN: BẢO MẬT ZIGBEE TRONG NHÀ THÔNG MINH

Tài liệu này vạch ra kế hoạch hành động cụ thể cho **Tuần 03** của dự án nghiên cứu, thiết lập bảng kiểm bảo mật (Security Checklist) và phân tích mối đe dọa toàn diện bằng mô hình **STRIDE**.

---

### 1. Kế hoạch Hành động Tuần 3 (Action Plan - Week 3)

Mục tiêu chính của Tuần 3 là hoàn tất thực nghiệm kiểm thử, thu thập các bằng chứng kỹ thuật (tệp PCAP, file cấu hình) và hoàn thiện phân tích lý thuyết.

| Thời gian | Công việc cụ thể | Sản phẩm / Minh chứng cần thu thập | Người chịu trách nhiệm |
| :--- | :--- | :--- | :--- |
| **Ngày 1 - 2** | - Cài đặt môi trường ảo hóa Docker (MQTT + Zigbee2MQTT).<br>- Cắm và cấu hình USB Dongle CC2531 Sniffer trên máy ảo Kali Linux. | - File `docker-compose.yml` hoạt động.<br>- Lệnh kiểm tra driver phần cứng thành công. | Sinh viên thực hiện |
| **Ngày 3 - 4** | - Thực hiện Kịch bản Tấn công 1: Sniffing bắt gói tin Pairing để trích xuất khóa mạng.<br>- Thực hiện Kịch bản Tấn công 2: Tấn công phát lại (Replay Attack). | - Ảnh chụp màn hình Wireshark bắt được gói tin chứa khóa.<br>- File capture gói tin `zigbee_sniff.pcap`. | Sinh viên thực hiện |
| **Ngày 5** | - Thực hiện cấu hình an toàn hệ thống (Hardening) theo cấu hình đề xuất.<br>- Chạy lại quy trình tấn công để kiểm chứng kết quả phòng thủ. | - File `configuration.yaml` đã tối ưu.<br>- Ảnh chụp Wireshark ghi nhận dữ liệu bị mã hóa hoàn toàn (không đọc được payload). | Sinh viên thực hiện |
| **Ngày 6** | - Thực hiện phân tích rủi ro chi tiết và đánh giá checklist bảo mật. | - Bảng đánh giá rủi ro hoàn chỉnh. | Sinh viên thực hiện |
| **Ngày 7** | - Tổng hợp kết quả thực nghiệm, viết báo cáo phân tích tuần và chuẩn bị slide demo. | - Báo cáo PDF hoàn chỉnh gửi giảng viên hướng dẫn. | Nhóm/Sinh viên |

---

### 2. Mô hình phân tích mối đe dọa STRIDE cho Zigbee Smart Home

Mô hình STRIDE được áp dụng để xác định các rủi ro bảo mật tiềm tàng đối với các thành phần trong hệ thống Zigbee Smart Home.

| Mối đe dọa STRIDE | Nguy cơ cụ thể trong mạng Zigbee | Cơ chế giảm thiểu rủi ro (Mitigation) |
| :--- | :--- | :--- |
| **Spoofing** (Giả mạo) | - Kẻ tấn công giả danh một thiết bị cảm biến Zigbee hợp lệ để gửi dữ liệu giả mạo (ví dụ: gửi tín hiệu cửa mở giả).<br>- Giả mạo Coordinator để lừa các thiết bị đầu cuối join vào mạng giả. | - Sử dụng khóa liên kết duy nhất (Unique Link Key) được tạo ra từ Install Code của từng thiết bị.<br>- Xác thực thực thể nghiêm ngặt trước khi cho phép truyền nhận dữ liệu. |
| **Tampering** (Can thiệp) | - Kẻ tấn công sửa đổi nội dung gói tin điều khiển trên đường truyền không dây (ví dụ: thay đổi lệnh từ "Tắt" thành "Bật"). | - Mã hóa dữ liệu ở lớp mạng bằng thuật toán mã hóa đối xứng AES-128 CCM.<br>- Sử dụng mã xác thực thông điệp (MIC - Message Integrity Code) để đảm bảo gói tin không bị sửa đổi. |
| **Repudiation** (Chối bỏ) | - Một thiết bị thực hiện hành động (ví dụ: khóa cửa được mở) nhưng hệ thống không thể chứng minh được ai hay thiết bị nào đã ra lệnh mở do thiếu log hoặc log bị giả mạo. | - Ghi nhật ký tập trung tại MQTT Broker và Home Assistant.<br>- Đảm bảo tất cả lệnh gửi đi từ Home Assistant đều được gắn kèm User ID và lưu trữ trong cơ sở dữ liệu có cơ chế bảo vệ ghi đè. |
| **Information Disclosure** (Tiết lộ thông tin) | - Kẻ tấn công nghe lén tần số 2.4 GHz, giải mã lưu lượng truyền tải để thu thập trạng thái sinh hoạt của gia chủ (khi nào có người ở nhà, trạng thái camera). | - Không sử dụng khóa liên kết mặc định toàn cầu.<br>- Đảm bảo tất cả lưu lượng dữ liệu Zigbee luôn được mã hóa.<br>- Mã hóa TLS 1.3 đối với các giao tiếp tầng trên (MQTT và Web Frontend). |
| **Denial of Service** (Từ chối dịch vụ) | - Gây nhiễu sóng vật lý (Jamming) tần số 2.4 GHz khiến các thiết bị mất kết nối.<br>- Tấn công làm cạn kiệt pin (Battery-Drain attack) bằng cách liên tục gửi các yêu cầu yêu cầu thiết bị đầu cuối thức dậy (Wake-up) và phản hồi. | - Cấu hình chuyển kênh tần số linh hoạt.<br>- Cấu hình giới hạn tốc độ yêu cầu (Rate limiting) tại Coordinator đối với các yêu cầu từ thiết bị đầu cuối.<br>- Lắp đặt thiết bị lọc nhiễu. |
| **Elevation of Privilege** (Leo thang đặc quyền) | - Kẻ tấn công khai thác lỗ hổng tràn bộ đệm trên firmware Coordinator để thực thi mã từ xa, chiếm quyền kiểm soát toàn bộ mạng Zigbee và các thiết bị kết nối. | - Cập nhật firmware mới nhất cho Coordinator và thiết bị đầu cuối định kỳ.<br>- Áp dụng nguyên tắc đặc quyền tối thiểu: Không cho phép giao diện quản trị Zigbee2MQTT chạy quyền root trực tiếp trên hệ thống máy chủ. |

---

### 3. Checklist Kiểm toán Bảo mật Zigbee (Security Audit Checklist)

Sử dụng checklist này để đánh giá nhanh mức độ an toàn của một hệ thống Zigbee Smart Home đang triển khai thực tế.

* **Nhóm 1: Quản lý Khóa và Xác thực**
  - [ ] Khóa mạng (Network Key) đã được cấu hình sinh ngẫu nhiên thay vì sử dụng mặc định? (Đạt/Không)
  - [ ] Hệ thống sử dụng Install Codes (Unique Link Keys) đối với các thiết bị Zigbee 3.0 mới ghép nối thay vì dùng `ZigBeeAlliance09`? (Đạt/Không)
  - [ ] Khóa mạng có được định kỳ thay đổi hoặc tự động đổi khi xóa một thiết bị khỏi mạng không? (Đạt/Không)

* **Nhóm 2: Kiểm soát Truy cập và Gia nhập Mạng**
  - [ ] Thông số `permit_join` đã được cấu hình là `false` mặc định chưa? (Đạt/Không)
  - [ ] Có thiết lập thời gian tự động tắt (Timeout) cho Permit Join khi kích hoạt thủ công không? (Đạt/Không)
  - [ ] Mật khẩu và Token của Web Frontend quản trị Zigbee2MQTT có độ phức tạp cao và đã được kích hoạt chưa? (Đạt/Không)

* **Nhóm 3: Bảo mật Kênh truyền và Tầng ứng dụng**
  - [ ] Giao thức kết nối đến MQTT Broker có được mã hóa qua TLS (Port 8883) chưa? (Đạt/Không)
  - [ ] MQTT Broker đã tắt tính năng cho phép kết nối vô danh (Anonymous connection)? (Đạt/Không)
  - [ ] Giao diện Web Frontend có được bọc qua HTTPS/TLS không? (Đạt/Không)

* **Nhóm 4: Quản lý Bản vá và Phần cứng**
  - [ ] Firmware của thiết bị Coordinator và các thiết bị đầu cuối đã được cập nhật phiên bản mới nhất chưa? (Đạt/Không)
  - [ ] Vị trí đặt Coordinator vật lý có an toàn, tránh việc kẻ xấu tiếp cận trực tiếp và tháo gỡ phần cứng? (Đạt/Không)
