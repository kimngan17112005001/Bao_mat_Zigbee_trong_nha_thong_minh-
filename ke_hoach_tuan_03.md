# KẾ HOẠCH TUẦN 3 & PHÂN TÍCH MỐI ĐE DỌA STRIDE
## DỰ ÁN MÔ PHỎNG: BẢO MẬT ZIGBEE TRONG NHÀ THÔNG MINH

Tài liệu này vạch ra kế hoạch hành động cụ thể cho **Tuần 03** của dự án nghiên cứu, thiết lập bảng kiểm bảo mật (Security Checklist) và phân tích mối đe dọa toàn diện bằng mô hình **STRIDE** cho môi trường mô phỏng.

---

### 1. Kế hoạch Hành động Tuần 3 (Action Plan - Week 3)

Mục tiêu chính của Tuần 3 là hoàn tất các kịch bản mô phỏng kiểm thử, thu thập các bằng chứng kỹ thuật ảo (tệp PCAP từ trình mô phỏng Cooja, file cấu hình) và hoàn thiện phân tích lý thuyết.

| Thời gian | Công việc cụ thể | Sản phẩm / Minh chứng cần thu thập | Người chịu trách nhiệm |
| :--- | :--- | :--- | :--- |
| **Ngày 1 - 2** | - Cài đặt môi trường ảo hóa Docker và trình mô phỏng Cooja (Contiki-NG).<br>- Tạo lập mô hình mạng ảo gồm Node Coordinator, Node cảm biến và Node tấn công. | - File cấu hình mô phỏng Cooja (`.csc`) hoạt động.<br>- Sơ đồ bố trí các node ảo trong vùng phủ sóng. | Sinh viên thực hiện |
| **Ngày 3 - 4** | - Chạy Kịch bản Mô phỏng Tấn công 1: Trích xuất gói tin bắt tay ghép đôi để tìm Network Key.<br>- Chạy Kịch bản Mô phỏng Tấn công 2: Thực hiện tấn công phát lại (Replay Attack) gửi lệnh bật/tắt thiết bị ảo. | - Ảnh chụp màn hình Wireshark giải mã khóa mạng từ tệp pcap mô phỏng.<br>- File capture gói tin ảo `zigbee_sim.pcap`. | Sinh viên thực hiện |
| **Ngày 5** | - Thực hiện cấu hình an toàn hệ thống mô phỏng (Hardening) theo cấu hình đề xuất.<br>- Chạy lại mô phỏng tấn công để kiểm chứng kết quả phòng thủ trong môi trường ảo. | - File `configuration.yaml` đã tối ưu cho Gateway ảo.<br>- Ảnh chụp Wireshark ghi nhận dữ liệu mô phỏng bị mã hóa hoàn toàn (không đọc được payload). | Sinh viên thực hiện |
| **Ngày 6** | - Thực hiện phân tích rủi ro chi tiết và đánh giá checklist bảo mật cho hệ thống mô phỏng. | - Bảng đánh giá rủi ro hoàn chỉnh. | Sinh viên thực hiện |
| **Ngày 7** | - Tổng hợp kết quả mô phỏng, viết báo cáo phân tích tuần và chuẩn bị slide demo chạy mô phỏng. | - Báo cáo PDF hoàn chỉnh gửi giảng viên hướng dẫn. | Nhóm/Sinh viên |

---

### 2. Mô hình phân tích mối đe dọa STRIDE cho hệ thống Zigbee mô phỏng

Mô hình STRIDE được áp dụng để xác định các rủi ro bảo mật tiềm tàng đối với các thành phần trong hệ thống Zigbee Smart Home ảo hóa.

| Mối đe dọa STRIDE | Nguy cơ cụ thể trong mạng Zigbee mô phỏng | Cơ chế giảm thiểu rủi ro (Mitigation) |
| :--- | :--- | :--- |
| **Spoofing** (Giả mạo) | - Node tấn công ảo giả danh một thiết bị cảm biến ảo để gửi dữ liệu giả mạo (ví dụ: gửi tín hiệu cửa mở giả).<br>- Giả mạo Coordinator ảo để lừa các thiết bị đầu cuối join vào mạng mô phỏng giả. | - Sử dụng khóa liên kết duy nhất (Unique Link Key) được tạo ra từ Install Code của từng thiết bị ảo.<br>- Xác thực thực thể nghiêm ngặt trước khi cho phép truyền nhận dữ liệu ảo. |
| **Tampering** (Can thiệp) | - Node tấn công ảo sửa đổi nội dung gói tin điều khiển trên đường truyền không dây mô phỏng (ví dụ: thay đổi lệnh từ "Tắt" thành "Bật"). | - Mã hóa dữ liệu ở lớp mạng bằng thuật toán mã hóa đối xứng AES-128 CCM.<br>- Sử dụng mã xác thực thông điệp (MIC - Message Integrity Code) để đảm bảo gói tin ảo không bị sửa đổi. |
| **Repudiation** (Chối bỏ) | - Một thiết bị ảo thực hiện hành động (ví dụ: khóa cửa được mở) nhưng hệ thống không thể chứng minh được ai hay thiết bị nào đã ra lệnh mở do thiếu log ảo hoặc log bị giả mạo. | - Ghi nhật ký tập trung tại MQTT Broker ảo và Home Assistant.<br>- Đảm bảo tất cả lệnh gửi đi từ Home Assistant ảo đều được gắn kèm User ID và lưu trữ trong cơ sở dữ liệu có cơ chế bảo vệ ghi đè. |
| **Information Disclosure** (Tiết lộ thông tin) | - Node tấn công ảo nghe lén kênh truyền mô phỏng, giải mã lưu lượng truyền tải để thu thập trạng thái sinh hoạt ảo của gia chủ (khi nào có người ở nhà, trạng thái camera). | - Không sử dụng khóa liên kết mặc định toàn cầu.<br>- Đảm bảo tất cả lưu lượng dữ liệu Zigbee ảo luôn được mã hóa.<br>- Mã hóa TLS 1.3 đối với các giao tiếp tầng trên (MQTT và Web Frontend ảo). |
| **Denial of Service** (Từ chối dịch vụ) | - Gây nhiễu sóng mô phỏng (Jamming) tần số không dây ảo khiến các thiết bị ảo mất kết nối.<br>- Tấn công làm cạn kiệt pin ảo (Battery-Drain attack) bằng cách liên tục gửi các yêu cầu yêu cầu thiết bị đầu cuối ảo thức dậy (Wake-up) và phản hồi. | - Cấu hình chuyển kênh tần số linh hoạt.<br>- Cấu hình giới hạn tốc độ yêu cầu (Rate limiting) tại Coordinator ảo đối với các yêu cầu từ thiết bị đầu cuối ảo.<br>- Thiết lập cơ chế lọc nhiễu sóng trong mô phỏng. |
| **Elevation of Privilege** (Leo thang đặc quyền) | - Node tấn công ảo khai thác lỗ hổng tràn bộ đệm trên firmware Coordinator ảo để thực thi mã từ xa, chiếm quyền kiểm soát toàn bộ mạng Zigbee mô phỏng và các thiết bị kết nối. | - Cập nhật firmware mới nhất cho Coordinator và thiết bị đầu cuối ảo định kỳ.<br>- Áp dụng nguyên tắc đặc quyền tối thiểu: Không cho phép giao diện quản trị Zigbee2MQTT ảo chạy quyền root trực tiếp trên hệ thống máy chủ ảo. |

---

### 3. Checklist Kiểm toán Bảo mật Zigbee (Security Audit Checklist)

Sử dụng checklist này để đánh giá nhanh mức độ an toàn của một hệ thống Zigbee mô phỏng đang triển khai thực tế.

* **Nhóm 1: Quản lý Khóa và Xác thực**
  - [ ] Khóa mạng (Network Key) đã được cấu hình sinh ngẫu nhiên thay vì sử dụng mặc định trong mô phỏng? (Đạt/Không)
  - [ ] Hệ thống sử dụng Install Codes (Unique Link Keys) đối với các thiết bị Zigbee ảo mới ghép nối thay vì dùng `ZigBeeAlliance09`? (Đạt/Không)
  - [ ] Khóa mạng có được định kỳ thay đổi hoặc tự động đổi khi xóa một thiết bị ảo khỏi mạng không? (Đạt/Không)

* **Nhóm 2: Kiểm soát Truy cập và Gia nhập Mạng**
  - [ ] Thông số `permit_join` đã được cấu hình là `false` mặc định trong tệp cấu hình ảo chưa? (Đạt/Không)
  - [ ] Có thiết lập thời gian tự động tắt (Timeout) cho Permit Join ảo khi kích hoạt thủ công không? (Đạt/Không)
  - [ ] Mật khẩu và Token của Web Frontend quản trị Zigbee2MQTT ảo có độ phức tạp cao và đã được kích hoạt chưa? (Đạt/Không)

* **Nhóm 3: Bảo mật Kênh truyền và Tầng ứng dụng**
  - [ ] Giao thức kết nối đến MQTT Broker ảo có được mã hóa qua TLS (Port 8883) chưa? (Đạt/Không)
  - [ ] MQTT Broker ảo đã tắt tính năng cho phép kết nối vô danh (Anonymous connection)? (Đạt/Không)
  - [ ] Giao diện Web Frontend ảo có được bọc qua HTTPS/TLS không? (Đạt/Không)

* **Nhóm 4: Quản lý Bản vá và Phần cứng**
  - [ ] Firmware của thiết bị Coordinator ảo và các thiết bị đầu cuối ảo đã được cập nhật phiên bản mới nhất chưa? (Đạt/Không)
  - [ ] Quyền truy cập vật lý/ảo vào máy chủ chạy phần mềm mô phỏng đã được cấu hình chặt chẽ chưa? (Đạt/Không)
