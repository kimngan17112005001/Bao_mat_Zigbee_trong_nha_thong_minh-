# Bảo mật Giao thức Zigbee trong Hệ thống Nhà thông minh (Smart Home)

Dự án này tập trung vào nghiên cứu, phân tích các lỗ hổng bảo mật của giao thức truyền thông không dây Zigbee trong nhà thông minh và đề xuất các giải pháp nâng cao cấu hình an toàn (Hardening). Nghiên cứu sử dụng phương pháp mô phỏng và thực nghiệm cấu hình trên phần mềm trung gian mã nguồn mở phổ biến.

---

## 📢 TUYÊN BỐ VỀ ĐIỀU KIỆN PHẦN CỨNG
* **Yêu cầu phần cứng**: **KHÔNG YÊU CẦU PHẦN CỨNG THỰC TẾ.**
* **Phương pháp triển khai**: Dự án được xây dựng hoàn toàn dựa trên **mô phỏng phần mềm (Software-based Simulation)** thông qua trình giả lập mạng cảm biến không dây **Cooja (Contiki-NG)**, công cụ bắt/phân tích gói tin **Wireshark**, kết hợp thiết lập cấu hình an toàn trên bộ cầu nối ảo **Zigbee2MQTT** và **MQTT Broker (Mosquitto)**. Việc này đảm bảo tính khả thi, an toàn và có thể tái lập trong môi trường phòng thí nghiệm ảo.

---

## 📁 Cấu trúc Thư mục Dự án

```text
zigbee-security-analysis/
├── README.md                      # Tệp tin giới thiệu này (tuyên bố phần cứng, hướng dẫn đọc)
├── bao_cao_bao_mat_zigbee.md      # Báo cáo tiểu luận chi tiết và đầy đủ nhất (Sơ đồ, bảng rủi ro, checklist)
├── slide_thuyet_trinh.md          # Đề cương nội dung Slide thuyết trình bảo vệ đề tài
├── ke_hoach_tuan_03.md            # Báo cáo tiến độ tuần 03 & Mô hình phân tích đe dọa STRIDE
└── lab_huong_dan/                 # Thư mục tài liệu thực hành / Lab
    ├── README.md                  # Hướng dẫn chi tiết các bước chạy mô phỏng Cooja & giải mã Wireshark
    └── configuration.yaml         # File cấu hình mẫu đã được bảo mật (Hardened) cho Zigbee2MQTT
```

---

## 🛠️ Tóm tắt các công cụ và dự án tham chiếu chính
* **[Zigbee2MQTT](https://github.com/Koenkk/zigbee2mqtt)**: Cầu nối truyền tải thông tin từ mạng Zigbee sang các topic MQTT, đóng vai trò quản lý chính các kết nối và là trung tâm của các chính sách bảo mật cấu hình.
* **[zigpy](https://github.com/zigpy/zigpy)**: Thư viện mã nguồn mở Python triển khai Zigbee protocol stack, được tham chiếu để hiểu rõ cấu trúc các frame truyền nhận ở tầng mạng.
* **[OWASP IoT Security Verification Standard (ISVS)](https://github.com/OWASP/IoT-Security-Verification-Standard-ISVS)**: Tiêu chuẩn kiểm toán bảo mật IoT làm cơ sở xây dựng bảng checklist đánh giá an toàn thông tin cho hệ thống.

---

## 🚀 Hướng dẫn Đọc và Khai thác dự án

1. **Đọc báo cáo tiểu luận chính**: Xem tại [bao_cao_bao_mat_zigbee.md](file:///c:/Users/Dell/.gemini/antigravity-ide/scratch/zigbee-security-analysis/bao_cao_bao_mat_zigbee.md) để nắm toàn bộ cơ sở lý thuyết, phân tích rủi ro quá trình kết nối thiết bị và các biện pháp giảm thiểu.
2. **Kiểm tra file cấu hình an toàn**: Xem tại [lab_huong_dan/configuration.yaml](file:///c:/Users/Dell/.gemini/antigravity-ide/scratch/zigbee-security-analysis/lab_huong_dan/configuration.yaml) để tìm hiểu các tham số phòng thủ cụ thể.
3. **Thực hành Lab**: Làm theo các bước hướng dẫn tại [lab_huong_dan/README.md](file:///c:/Users/Dell/.gemini/antigravity-ide/scratch/zigbee-security-analysis/lab_huong_dan/README.md) để tự giả lập việc sniffing và replay gói tin.
4. **Chuẩn bị slide thuyết trình**: Xem tại [slide_thuyet_trinh.md](file:///c:/Users/Dell/.gemini/antigravity-ide/scratch/zigbee-security-analysis/slide_thuyet_trinh.md) để chuẩn bị cho buổi bảo vệ đồ án/tiểu luận.
