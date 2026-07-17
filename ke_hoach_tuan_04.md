# BÁO CÁO TIẾN ĐỘ TUẦN 04 & KẾ HOẠCH BẢO VỆ TIỂU LUẬN
## ĐỀ TÀI 16: BẢO MẬT ZIGBEE TRONG NHÀ THÔNG MINH (SMART HOME)

---

**THÔNG TIN CHUNG:**
* **Dự án**: Bảo mật Zigbee trong nhà thông minh
* **Sinh viên thực hiện**: Bùi Thị Kim Ngân
* **MSSV**: 231A010257
* **Mã học phần**: 253INT441001
* **Giảng viên hướng dẫn**: PGS. TS. Nguyễn Văn A

---

## 1. Kế hoạch Hành động Tuần 04 (Action Plan - Week 4)

Mục tiêu cốt lõi của Tuần 04 là tiến hành kiểm tra hồi quy bảo mật (Security Regression Testing) trên hệ thống đã củng cố, hoàn thiện tài liệu báo cáo tiểu luận cuối cùng, tối ưu hóa slide thuyết trình và đóng gói sản phẩm mã nguồn đưa lên GitHub.

| Ngày thực hiện | Nội dung công việc chi tiết | Sản phẩm / Minh chứng kỹ thuật | Người thực hiện | Trạng thái |
| :--- | :--- | :--- | :--- | :--- |
| **Ngày 1 - 2** | - Thực hiện kiểm định lại (Re-test) các kịch bản tấn công trên cấu hình an toàn mới.<br>- Xác minh xem kẻ tấn công còn có thể sniffing lấy Network Key hay thực hiện Replay Attack thành công hay không. | - Nhật ký log Wireshark cho thấy gói tin Transport Key bị mã hóa hoàn toàn bởi Unique Link Key.<br>- Log bóng đèn từ chối lệnh Replay do lỗi lệch bộ đếm Frame Counter. | Bùi Thị Kim Ngân | **Hoàn thành** |
| **Ngày 3** | - Cấu hình chi tiết bộ lọc phân quyền truy cập **ACL (Access Control List)** trên Mosquitto MQTT Broker để áp dụng nguyên tắc đặc quyền tối thiểu cho các client. | - File cấu hình ACL `mosquitto.acl`. | Bùi Thị Kim Ngân | **Hoàn thành** |
| **Ngày 4** | - Rà soát và hoàn thiện báo cáo tiểu luận chi tiết ([bao_cao_bao_mat_zigbee.md](file:///c:/Users/Dell/.gemini/antigravity-ide/scratch/zigbee-security-analysis/bao_cao_bao_mat_zigbee.md)).<br>- Kiểm tra lại cấu trúc trình bày, lỗi chính tả và các trích dẫn khoa học. | - Báo cáo tiểu luận bản cuối cùng sẵn sàng chuyển đổi sang PDF. | Bùi Thị Kim Ngân | **Hoàn thành** |
| **Ngày 5** | - Hoàn thiện tệp Slide thuyết trình ([slide_thuyet_trinh.md](file:///c:/Users/Dell/.gemini/antigravity-ide/scratch/zigbee-security-analysis/slide_thuyet_trinh.md)).<br>- Tập dượt thuyết trình thử để đảm bảo thời gian bảo vệ đúng quy định (10-15 phút). | - File slide thuyết trình hoàn chỉnh. | Bùi Thị Kim Ngân | **Hoàn thành** |
| **Ngày 6** | - Đánh giá tổng kết các kết quả đạt được, ghi nhận 3 bài học học được và 2 điểm hạn chế của đề tài.<br>- Xây dựng định hướng phát triển tiếp theo. | - Phần kết luận và tự đánh giá trong báo cáo. | Bùi Thị Kim Ngân | **Hoàn thành** |
| **Ngày 7** | - Đẩy toàn bộ mã nguồn, cấu hình và báo cáo lên GitHub Repository.<br>- Đóng gói tệp nộp bài gửi giảng viên hướng dẫn. | - Link Repository GitHub hoạt động công khai.<br>- Biên bản nghiệm thu báo cáo. | Bùi Thị Kim Ngân | **Hoàn thành** |

---

## 2. Kịch bản Kiểm thử Bảo mật sau khi Hardening (Security Verification Scenarios)

Để chứng minh tính hiệu quả của cấu hình an toàn đã đề xuất ở Tuần 03, nhóm tiến hành 3 kịch bản kiểm thử đối chứng sau:

### Kịch bản 1: Thử nghiệm Sniffing trên cấu hình sử dụng Install Code (Zigbee 3.0)
* **Mục tiêu**: Xác minh kẻ tấn công không thể lấy được Network Key.
* **Cách thực hiện**:
  1. Nạp mã Install Code của End Device vào Coordinator trước khi ghép cặp.
  2. Bật Permit Join và cho thiết bị kết nối vào mạng.
  3. Kẻ tấn công sử dụng Cooja Radio Logger thu giữ tệp `.pcap`.
* **Kết quả kiểm thử**: Wireshark không thể tự động giải mã gói tin `Transport Key` bằng khóa liên kết mặc định `ZigBeeAlliance09`. Khóa Network Key truyền đi được mã hóa an toàn bằng Unique Link Key tạo từ Install Code, kẻ tấn công chỉ thu được dữ liệu đã mã hóa (Ciphertext).

### Kịch bản 2: Thử nghiệm Replay Attack khi bộ đếm Frame Counter hoạt động
* **Mục tiêu**: Xác minh thiết bị đầu cuối ngăn chặn các gói tin điều khiển phát lại.
* **Cách thực hiện**:
  1. Kẻ tấn công ghi lại gói tin điều khiển tắt bóng đèn (Ví dụ: Frame Counter lớp mạng là `105`).
  2. Kẻ tấn công phát lại gói tin này vào kênh truyền khi Frame Counter hiện tại của mạng đã tăng lên `120`.
* **Kết quả kiểm thử**: Thiết bị bóng đèn nhận được gói tin, kiểm tra bộ đếm Frame Counter của gói tin nhận được (`105`) nhỏ hơn Frame Counter hiện tại lưu trên thiết bị (`120`). Bóng đèn tự động loại bỏ (drop) gói tin này và ghi nhận cảnh báo bảo mật, cuộc tấn công phát lại thất bại.

### Kịch bản 3: Thử nghiệm phân quyền truy cập ACL trên MQTT Broker
* **Mục tiêu**: Xác minh client lạ không thể chèn lệnh điều khiển qua MQTT.
* **Cách thực hiện**:
  1. Cấu hình file `mosquitto.acl` giới hạn quyền:
     ```text
     user gateway_coordinator_node
     topic readwrite zigbee2mqtt/#

     user homeassistant_client
     topic readwrite homeassistant/#
     topic readwrite zigbee2mqtt/#

     user guest_attacker
     topic read homeassistant/status
     ```
  2. Kẻ tấn công đăng nhập tài khoản `guest_attacker` và cố gắng publish lệnh tắt đèn vào topic `zigbee2mqtt/den_phong_khach/set`.
* **Kết quả kiểm thử**: MQTT Broker chặn quyền ghi của `guest_attacker` trên topic `zigbee2mqtt/#`. Lệnh điều khiển giả mạo không được truyền đi, đảm bảo an toàn cho Gateway.

---

## 3. Checklist Rà soát Cuối cùng (Final Release Checklist)

Checklist đánh giá chất lượng sản phẩm trước khi nộp bài và bảo vệ đề tài:

- [x] **Tính đầy đủ**: Đã có đầy đủ báo cáo tiểu luận (Markdown/PDF), slide thuyết trình, file cấu hình bảo mật mẫu, và hướng dẫn Lab thực hành.
- [x] **Tính nhất quán**: Các tham số bảo mật đề xuất trong báo cáo thống nhất hoàn toàn với file cấu hình `configuration.yaml` thực tế.
- [x] **Tính đúng đắn**: Sơ đồ Mermaid biểu diễn chính xác kiến trúc mạng lưới và luồng gói tin MQTT.
- [x] **Trích dẫn khoa học**: Danh mục tài liệu tham khảo được trình bày rõ ràng, trích dẫn chính xác link dự án mã nguồn mở Zigbee2MQTT, zigpy và OWASP ISVS.
- [x] **Đóng gói mã nguồn**: Thư mục dự án sạch sẽ, đã xóa bỏ các tệp tin nháp dư thừa và có tệp hướng dẫn đọc README rõ ràng.
