# BÁO CÁO TIẾN ĐỘ TUẦN 03 & PHÂN TÍCH MỐI ĐE DỌA STRIDE
## ĐỀ TÀI 16: BẢO MẬT ZIGBEE TRONG NHÀ THÔNG MINH (SMART HOME)

---

**THÔNG TIN CHUNG:**
* **Dự án**: Đánh giá và tăng cường bảo mật giao thức Zigbee qua mô phỏng nhà thông minh
* **Sinh viên thực hiện**: Bùi Thị Kim Ngân
* **MSSV**: 20260002
* **Mã học phần**: IoT-SEC-2026
* **Lớp**: An toàn Thông tin IoT - K15
* **Giảng viên hướng dẫn**: PGS. TS. Nguyễn Văn A

---

## 1. Kế hoạch Hành động Tuần 03 (Action Plan - Week 3)

Mục tiêu cốt lõi của Tuần 03 là hoàn thiện việc xây dựng môi trường Lab mô phỏng phần mềm, chạy thử nghiệm kịch bản tấn công (Sniffing, Replay) để thu thập dữ liệu gói tin ảo, triển khai cấu hình an toàn (Hardening) và lập bảng checklist đánh giá bảo mật.

| Ngày thực hiện | Nội dung công việc chi tiết | Sản phẩm / Minh chứng kỹ thuật | Người thực hiện | Trạng thái |
| :--- | :--- | :--- | :--- | :--- |
| **Ngày 1 - 2** | - Cài đặt máy ảo Linux, Docker, và trình giả lập mạng vô tuyến **Cooja (Contiki-NG)**.<br>- Thiết lập mạng lưới node ảo bao gồm: 1 Coordinator (Trust Center), 2 End Devices (Cảm biến cửa, Bóng đèn) và 1 Attacker Node. | - Tệp tin cấu hình mô phỏng Cooja (`.csc`).<br>- Ảnh chụp sơ đồ phân bổ vị trí địa lý của các node ảo trong Cooja. | Bùi Thị Kim Ngân | **Hoàn thành** |
| **Ngày 3 - 4** | - Thực hiện kịch bản tấn công 1: Nghe lén pha bắt tay ghép cặp bằng khóa liên kết mặc định (`ZigBeeAlliance09`), xuất file pcap.<br>- Thực hiện kịch bản tấn công 2: Ghi lại gói tin điều khiển và phát lại (Replay Attack) bằng Scapy. | - Tệp tin bắt gói tin ảo `zigbee_sim.pcap` và `control_cmd.pcap`.<br>- Ảnh chụp màn hình Wireshark giải mã được khóa mạng **Network Key**. | Bùi Thị Kim Ngân | **Hoàn thành** |
| **Ngày 5** | - Cấu hình an toàn cho bộ điều phối ảo qua file `configuration.yaml` của Zigbee2MQTT (sinh khóa ngẫu nhiên, tắt permit join).<br>- Thiết lập TLS 1.3 và mật khẩu mạnh trên Mosquitto MQTT Broker. Chạy lại kịch bản tấn công để đối chứng. | - File cấu hình [configuration.yaml](file:///c:/Users/Dell/.gemini/antigravity-ide/scratch/zigbee-security-analysis/lab_huong_dan/configuration.yaml) đã tối ưu.<br>- Nhật ký Wireshark ghi nhận toàn bộ gói tin bị mã hóa (không giải mã được payload). | Bùi Thị Kim Ngân | **Hoàn thành** |
| **Ngày 6** | - Thực hiện phân tích rủi ro hệ thống thông qua mô hình mối đe dọa STRIDE.<br>- Điền thông tin đánh giá checklist bảo mật hệ thống dựa trên tiêu chuẩn OWASP ISVS. | - Bảng phân tích STRIDE hoàn chỉnh.<br>- Checklist kiểm toán an toàn IoT. | Bùi Thị Kim Ngân | **Hoàn thành** |
| **Ngày 7** | - Tổng hợp báo cáo tiến độ tuần 3, viết đề cương slide thuyết trình bảo vệ và tối ưu hóa tài liệu hướng dẫn. | - Tệp báo cáo [ke_hoach_tuan_03.md](file:///c:/Users/Dell/.gemini/antigravity-ide/scratch/zigbee-security-analysis/ke_hoach_tuan_03.md) hoàn chỉnh. | Bùi Thị Kim Ngân | **Hoàn thành** |

---

## 2. Mô hình phân tích mối đe dọa STRIDE cho mạng Zigbee-MQTT

Mô hình STRIDE được áp dụng để đánh giá toàn diện các rủi ro bảo mật tiềm tàng ở cả hai tầng: mạng không dây Zigbee và cầu nối MQTT trung gian trong nhà thông minh.

| Mối đe dọa | Nguy cơ cụ thể trong mạng mô phỏng | Cơ chế giảm thiểu rủi ro (Mitigation) | Trạng thái kiểm chứng |
| :--- | :--- | :--- | :--- |
| **S**poofing<br>(Giả mạo) | - Node tấn công ảo giả dạng cảm biến thật gửi gói tin báo trạng thái giả (ví dụ: báo cửa mở ảo).<br>- Kẻ tấn công giả danh Coordinator để kết nối với thiết bị con. | - Kích hoạt cơ chế **Install Code (Zigbee 3.0)** để tạo ra khóa liên kết duy nhất (Unique Link Key) cho từng thiết bị thay vì dùng khóa mặc định chung.<br>- Sử dụng TLS Client Certificates trên MQTT Broker. | Đã cấu hình kiểm chứng thành công bằng Install Code trên file cấu hình ảo. |
| **T**ampering<br>(Can thiệp) | - Can thiệp, sửa đổi nội dung gói tin điều khiển trên đường truyền không dây ảo (như đổi lệnh OFF thành ON). | - Mã hóa dữ liệu lớp mạng bằng thuật toán mã hóa đối xứng **AES-128 CCM** tích hợp mã xác thực thông điệp **MIC** để đảm bảo tính toàn vẹn gói tin. | Đạt tiêu chuẩn giao thức vô tuyến Zigbee. |
| **R**epudiation<br>(Chối bỏ) | - Người dùng hoặc thiết bị thực hiện hành động (như mở khóa cửa) nhưng hệ thống không ghi nhận nguồn gốc do thiếu log hoặc log bị chỉnh sửa. | - Bật log chi tiết trên Zigbee2MQTT và Mosquitto Broker.<br>- Ghi nhật ký tập trung về Home Assistant Core, phân quyền truy cập ghi đè log nghiêm ngặt. | Đã cấu hình lưu trữ nhật ký tập trung trên Hub. |
| **I**nformation Disclosure<br>(Tiết lộ thông tin) | - Nghe lén (Sniffing) lưu lượng sóng vô tuyến ảo để trích xuất khóa Network Key và giải mã toàn bộ dữ liệu đời sống cá nhân của gia chủ. | - Tuyệt đối không dùng khóa liên kết mặc định toàn cầu (`ZigBeeAlliance09`) khi ghép cặp.<br>- Sử dụng giao thức mã hóa đường truyền **TLS 1.3** (`mqtts://` cổng 8883) giữa Gateway và Broker. | Đã thực nghiệm bắt gói tin vô tuyến bằng Wireshark và mã hóa thành công. |
| **D**enial of Service<br>(Từ chối dịch vụ) | - Gây nhiễu sóng vô tuyến ảo (Jamming) làm tê liệt hệ thống.<br>- Tấn công làm cạn kiệt pin ảo (Battery-Drain) bằng cách liên tục gửi gói tin đánh thức thiết bị đầu cuối. | - Cấu hình đổi kênh tần số linh hoạt (Kênh 15, 20, 25, 26 ít trùng lặp Wi-Fi).<br>- Coordinator thiết lập giới hạn tốc độ yêu cầu (Rate Limiting) đối với các yêu cầu ghép cặp ảo. | Đã cấu hình chuyển kênh sang kênh 25. |
| **E**levation of Privilege<br>(Leo thang đặc quyền) | - Kẻ tấn công khai thác lỗi tràn bộ đệm trên firmware Coordinator ảo để thực thi mã từ xa, chiếm quyền kiểm soát toàn mạng LAN hoặc Hub trung tâm. | - Cập nhật bản vá firmware định kỳ cho USB Dongle Coordinator.<br>- Không chạy dịch vụ Zigbee2MQTT với quyền quản trị cao nhất (`root`) trên hệ điều hành máy chủ. | Đã cấu hình chạy dưới tài khoản user thường. |

---

## 3. Checklist Kiểm toán Bảo mật Hệ thống (Dựa trên chuẩn OWASP ISVS)

Sử dụng checklist này để kiểm tra nhanh mức độ củng cố bảo mật trên hệ thống Smart Home Zigbee-MQTT:

### 3.1. Nhóm 1: An toàn mạng vô tuyến Zigbee
* [x] **Khóa mạng ngẫu nhiên**: Tham số `network_key` trong file cấu hình Zigbee2MQTT đã được đặt là `generate` thay vì dùng khóa mặc định.
* [x] **Tắt Permit Join mặc định**: Tham số `permit_join` đã được cấu hình mặc định là `false`. Chỉ mở thủ công khi pairing và tự động đóng sau tối đa 120 giây.
* [x] **Sử dụng Install Codes (Unique Link Keys)**: Hệ thống đã kích hoạt việc ghép cặp qua mã Install Code của thiết bị Zigbee 3.0, loại bỏ hoàn toàn khóa mặc định toàn cầu `ZigBeeAlliance09`.
* [x] **Kênh truyền tránh nhiễu**: Hệ thống sử dụng kênh tần số 25 hoặc 26 để tránh xung đột với các dải kênh Wi-Fi 2.4GHz thông dụng lân cận.

### 3.2. Nhóm 2: An toàn đường truyền trung gian và MQTT Broker
* [x] **Mã hóa TLS đường truyền**: Kết nối đến Mosquitto Broker bắt buộc chạy qua giao thức bảo mật `mqtts://` ở cổng 8883 có chứng chỉ SSL/TLS.
* [x] **Tắt Anonymous Connection**: Cấu hình MQTT Broker đã đặt `allow_anonymous false` để cấm các kết nối vô danh.
* [x] **Xác thực tài khoản mạnh**: Mỗi client kết nối vào Broker (như Zigbee2MQTT, Home Assistant) đều có tài khoản và mật khẩu độ phức tạp cao riêng biệt.
* [x] **Kiểm soát truy cập (ACL)**: Đã thiết lập phân quyền ACL trên Mosquitto Broker để hạn chế quyền subscribe/publish của từng client trong phạm vi topic cần thiết.

### 3.3. Nhóm 3: An toàn giao diện điều khiển (Frontend UI)
* [x] **Bảo vệ cổng Web Frontend**: Giao diện quản trị của Zigbee2MQTT được đặt token xác thực mạnh (`auth_token`).
* [x] **Giới hạn IP lắng nghe**: Đã đổi IP lắng nghe của Web UI về `127.0.0.1` thay vì `0.0.0.0`, chỉ cho phép truy cập cục bộ hoặc bọc qua Reverse Proxy (Nginx) có HTTPS.

---

## 4. Kết luận tiến độ Tuần 03
Báo cáo tiến độ Tuần 03 đã thực hiện đầy đủ các mục tiêu đề ra trong đề cương nghiên cứu. Qua việc thực nghiệm các kịch bản tấn công giả lập trên môi trường phần mềm và đối chiếu các cấu hình phòng thủ, nhóm đã chứng minh được tính khả thi của các biện pháp bảo mật trên hệ thống thực tế. Các kết quả này là cơ sở vững chắc để hoàn thiện báo cáo tiểu luận cuối cùng của đề tài.
