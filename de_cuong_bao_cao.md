# ĐỀ CƯƠNG CHI TIẾT
## ĐỀ TÀI: ĐÁNH GIÁ VÀ TĂNG CƯỜNG BẢO MẬT GIAO THỨC ZIGBEE TRONG HỆ THỐNG NHÀ THÔNG MINH (SMART HOME)

---

### 1. Giới thiệu & Tính cấp thiết của Đề tài
Giao thức Zigbee là một trong những tiêu chuẩn truyền thông không dây tầm ngắn, tiêu thụ năng lượng thấp phổ biến nhất hiện nay trong các hệ thống IoT và Nhà thông minh (Smart Home). Được xây dựng trên nền tảng chuẩn IEEE 802.15.4, Zigbee kết nối hàng triệu thiết bị từ bóng đèn, cảm biến, ổ cắm cho đến các khóa cửa thông minh.

Tuy nhiên, do hạn chế về tài nguyên phần cứng (CPU, bộ nhớ, năng lượng), việc triển khai các cơ chế mã hóa phức tạp trên các thiết bị Zigbee đầu cuối gặp nhiều khó khăn. Thực tế cho thấy, nhiều hệ thống lắp đặt thương mại hoặc tự chế (DIY) vẫn đang sử dụng các cấu hình mặc định (default configurations) thiếu an toàn như: khóa liên kết mặc định (Default Link Key), cho phép thiết bị mới gia nhập mạng vô hạn hạn (Permit Join luôn mở), hoặc không mã hóa dữ liệu truyền tải. Điều này tạo cơ hội cho kẻ tấn công thực hiện nghe lén (Sniffing), giải mã gói tin, tấn công giả mạo (Replay Attack) hoặc thậm chí kiểm soát hoàn toàn hệ thống Smart Home. Do đó, việc nghiên cứu các lỗ hổng bảo mật của Zigbee, xây dựng mô hình giả lập/thực nghiệm tấn công và đề xuất các giải pháp cấu hình an toàn là vô cùng cấp thiết.

---

### 2. Mục tiêu nghiên cứu
- **Mục tiêu tổng quát**: Đánh giá thực trạng an toàn thông tin của giao thức Zigbee trong môi trường Smart Home, thực nghiệm các kịch bản tấn công điển hình và đề xuất các phương án cấu hình tối ưu nhằm nâng cao năng lực phòng thủ cho hệ thống.
- **Mục tiêu cụ thể**:
  1. Phân tích kiến trúc bảo mật của Zigbee ở các tầng Physical, MAC, Network và Application.
  2. Xác định các lỗ hổng bảo mật phổ biến trong quá trình trao đổi khóa (Key Exchange), định tuyến và quản lý thiết bị gia nhập mạng.
  3. Xây dựng môi trường Lab tối thiểu (giả lập hoặc sử dụng phần cứng SDR/USB Sniffer) để thực hiện sniffing, trích xuất khóa bảo mật (Network Key) và thực hiện tấn công Replay Frame điều khiển.
  4. Đề xuất quy trình cấu hình an toàn (Hardening Guide) cho các hệ thống điều phối Zigbee phổ biến (như Zigbee2MQTT, Home Assistant).
  5. Xây dựng checklist đánh giá nhanh mức độ an toàn của một hệ thống Zigbee Smart Home.

---

### 3. Phạm vi nghiên cứu
- **Phạm vi công nghệ**:
  - Giao thức Zigbee phiên bản Zigbee 3.0 và Zigbee Home Automation (HA 1.2).
  - Bộ điều phối trung tâm (Coordinator) chạy mã nguồn mở **Zigbee2MQTT** kết hợp với phần cứng USB Dongle (CC2531 hoặc CC2652P).
  - Các công cụ kiểm thử mã nguồn mở trên nền tảng Linux (KillerBee, Wireshark).
- **Phạm vi tấn công thực nghiệm**:
  - Tấn công thụ động: Nghe lén (Sniffing) lưu lượng mạng Zigbee trên tần số 2.4 GHz, bắt gói tin chứa khóa Network Key khi có thiết bị mới gia nhập mạng (Association phase).
  - Tấn công chủ động: Tấn công phát lại (Replay Attack) để gửi lệnh điều khiển thiết bị (bật/tắt đèn hoặc giả lập trạng thái cảm biến).
- **Phạm vi phòng thủ**:
  - Tối ưu hóa cấu hình trên tệp tin `configuration.yaml` của Zigbee2MQTT.
  - Quản lý khóa liên kết (Link Key) và chính sách gia nhập mạng (Join Policy).

---

### 4. Công cụ & Kho tài nguyên (Repository) tham khảo
Đề tài dựa trên việc ứng dụng và phân tích các công cụ mã nguồn mở uy tín sau:
1. **Zigbee2MQTT Repository** ([Koenkk/zigbee2mqtt](https://github.com/Koenkk/zigbee2mqtt)): Giải pháp kết nối thiết bị Zigbee vào mạng MQTT, đóng vai trò Coordinator chính trong Lab.
2. **KillerBee Framework** ([riverloopsec/killerbee](https://github.com/riverloopsec/killerbee)): Bộ công cụ kiểm thử bảo mật chuyên dụng cho chuẩn IEEE 802.15.4 và Zigbee, hỗ trợ nghe lén, tiêm gói tin (injection) và quét thiết bị.
3. **SecBee Tool** ([iot-sec/secbee](https://github.com/iot-sec/secbee)): Công cụ khai thác lỗ hổng bảo mật của Zigbee, giúp kiểm tra tính năng bảo mật của các thiết bị thông minh.
4. **Z-Attack** ([SebaD/Z-Attack](https://github.com/SebaD/Z-Attack)): Bộ công cụ hỗ trợ tấn công từ chối dịch vụ (DoS), replay và giả mạo trong mạng Zigbee.
5. **Wireshark dissector for Zigbee**: Công cụ phân tích gói tin mạng hỗ trợ giải mã giao thức IEEE 802.15.4 và giải mã payload Zigbee khi có Network Key.

---

### 5. Sản phẩm dự kiến
- **Báo cáo phân tích chi tiết**: Tài liệu PDF/Markdown trình bày cơ sở lý thuyết, sơ đồ kiến trúc, phân tích lỗ hổng và kết quả thực nghiệm.
- **Kịch bản & Môi trường Lab**: Mã nguồn, file cấu hình (`configuration.yaml`, Docker Compose) dựng hệ thống Zigbee2MQTT, Home Assistant phục vụ thử nghiệm bảo mật.
- **Tệp lưu vết gói tin (.pcap)**: Các mẫu gói tin thu giữ được trước và sau khi giải mã khóa mạng Zigbee phục vụ cho việc đối sánh chứng cứ.
- **Bảng so sánh cấu hình và Hướng dẫn phòng thủ**: Cung cấp cấu hình mẫu đã được tối ưu bảo mật.
- **Checklist kiểm toán**: File Excel/Markdown chứa danh sách các hạng mục cần kiểm tra để đảm bảo an toàn cho mạng Zigbee.

---

### 6. Danh mục tài liệu tham khảo (References)
1. **Wright, J. (2011)**. *KillerBee: Practical Zigbee Exploitation Framework*. SANS Technology Institute. Available at: [https://github.com/riverloopsec/killerbee](https://github.com/riverloopsec/killerbee).
2. **Koenkk**. *Zigbee2MQTT: Bridge Zigbee devices to MQTT*. Open-source documentation and repository. Available at: [https://github.com/Koenkk/zigbee2mqtt](https://github.com/Koenkk/zigbee2mqtt).
3. **Zigbee Alliance (2015)**. *Zigbee Specification Revision 21 (R21)*. Document 05-3474-21, Zigbee Alliance.
4. **NIST (2018)**. *Guide to Bluetooth Security (SP 800-121 Rev. 2)* - Tham khảo cấu trúc đánh giá không dây IoT. National Institute of Standards and Technology.
5. **Morgner, F., & Müller, T. (2017)**. *SecBee: Automated Zigbee Security Assessment*. Proceedings of the ACM Conference on Security and Privacy in Wireless and Mobile Networks (WiSec). Repo: [https://github.com/iot-sec/secbee](https://github.com/iot-sec/secbee).
6. **Z-Attack Contributors (2016)**. *Z-Attack: Zigbee Penetration Testing tool*. Source code repository. Available at: [https://github.com/SebaD/Z-Attack](https://github.com/SebaD/Z-Attack).
