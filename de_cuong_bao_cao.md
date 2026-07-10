# ĐỀ CƯƠNG CHI TIẾT
## ĐỀ TÀI: ĐÁNH GIÁ VÀ TĂNG CƯỜNG BẢO MẬT GIAO THỨC ZIGBEE QUA MÔ PHỎNG HỆ THỐNG NHÀ THÔNG MINH (SMART HOME)

---

### 1. Giới thiệu & Tính cấp thiết của Đề tài
Giao thức Zigbee là một trong những tiêu chuẩn truyền thông không dây tầm ngắn, tiêu thụ năng lượng thấp phổ biến nhất hiện nay trong các hệ thống IoT và Nhà thông minh (Smart Home). Được xây dựng trên nền tảng chuẩn IEEE 802.15.4, Zigbee kết nối hàng triệu thiết bị từ bóng đèn, cảm biến, ổ cắm cho đến các khóa cửa thông minh.

Tuy nhiên, do hạn chế về tài nguyên phần cứng (CPU, bộ nhớ, năng lượng), việc triển khai các cơ chế mã hóa phức tạp trên các thiết bị Zigbee đầu cuối gặp nhiều khó khăn. Thực tế cho thấy, nhiều hệ thống lắp đặt thương mại hoặc tự chế (DIY) vẫn đang sử dụng các cấu hình mặc định (default configurations) thiếu an toàn như: khóa liên kết mặc định (Default Link Key), cho phép thiết bị mới gia nhập mạng vô hạn hạn (Permit Join luôn mở), hoặc không mã hóa dữ liệu truyền tải.

Để nghiên cứu các lỗ hổng bảo mật của Zigbee mà không cần đầu tư nhiều thiết bị phần cứng chuyên dụng đắt tiền, việc sử dụng các công cụ **mô phỏng phần mềm (Software-based Simulation)** là một phương pháp tối ưu, an toàn và dễ tiếp cận. Mô phỏng giúp thiết lập nhanh chóng các kịch bản mạng khác nhau, phân tích lưu lượng gói tin ảo và kiểm chứng các véc-tơ tấn công như nghe lén (Sniffing), giải mã gói tin, tấn công giả mạo (Replay Attack) trong môi trường kiểm soát. Do đó, việc nghiên cứu các lỗ hổng bảo mật của Zigbee thông qua mô phỏng và đề xuất các giải pháp cấu hình an toàn là vô cùng cần thiết.

---

### 2. Mục tiêu nghiên cứu
- **Mục tiêu tổng quát**: Đánh giá thực trạng an toàn thông tin của giao thức Zigbee trong môi trường Smart Home thông qua phương pháp mô phỏng phần mềm, thiết lập các kịch bản tấn công ảo và đề xuất các phương án cấu hình tối ưu nhằm nâng cao năng lực phòng thủ cho hệ thống.
- **Mục tiêu cụ thể**:
  1. Phân tích kiến trúc bảo mật của Zigbee ở các tầng Physical, MAC, Network và Application.
  2. Xác định các lỗ hổng bảo mật phổ biến trong quá trình trao đổi khóa (Key Exchange), định tuyến và quản lý thiết bị gia nhập mạng.
  3. Xây dựng môi trường Lab mô phỏng (sử dụng phần mềm giả lập mạng Cooja/Contiki, NS-3 hoặc chạy giả lập virtual serial port kết hợp Zigbee2MQTT/MQTT Broker) để thực hiện mô phỏng nghe lén, trích xuất khóa bảo mật (Network Key) và mô phỏng tấn công Replay Frame điều khiển.
  4. Đề xuất quy trình cấu hình an toàn (Hardening Guide) cho các hệ thống điều phối Zigbee ảo hóa và thực tế.
  5. Xây dựng checklist đánh giá nhanh mức độ an toàn của một hệ thống Zigbee Smart Home.

---

### 3. Phạm vi nghiên cứu
- **Phạm vi công nghệ & mô phỏng**:
  - Giao thức Zigbee phiên bản Zigbee 3.0 và Zigbee Home Automation (HA 1.2).
  - Sử dụng phần mềm mô phỏng mạng không dây (ví dụ: **Cooja Simulator** giả lập các node IEEE 802.15.4 / Zigbee, hoặc dùng cổng nối tiếp ảo **socat** để mô phỏng bộ điều phối trung tâm chạy **Zigbee2MQTT** với dữ liệu đầu vào giả lập).
  - Các công cụ kiểm thử mã nguồn mở chạy trong môi trường ảo hóa Linux (KillerBee, Wireshark).
- **Phạm vi mô phỏng tấn công**:
  - Mô phỏng tấn công thụ động: Nghe lén (Sniffing) lưu lượng mạng Zigbee ảo trên các kênh mô phỏng, bắt gói tin chứa khóa Network Key khi có thiết bị ảo mới gia nhập mạng (Association phase).
  - Mô phỏng tấn công chủ động: Phát lại gói tin điều khiển (Replay Attack) đã capture từ tệp PCAP mô phỏng để thay đổi trạng thái của các thiết bị ảo (bật/tắt đèn hoặc giả lập trạng thái cảm biến).
- **Phạm vi phòng thủ**:
  - Tối ưu hóa cấu hình trên tệp cấu hình mô phỏng `configuration.yaml` của Zigbee2MQTT.
  - Quản lý khóa liên kết (Link Key) và chính sách gia nhập mạng (Join Policy) trên hệ thống mô phỏng.

---

### 4. Công cụ & Kho tài nguyên (Repository) tham khảo
Đề tài dựa trên việc ứng dụng và phân tích các công cụ mã nguồn mở uy tín sau:
1. **Zigbee2MQTT Repository** ([Koenkk/zigbee2mqtt](https://github.com/Koenkk/zigbee2mqtt)): Giải pháp kết nối thiết bị Zigbee vào mạng MQTT, đóng vai trò Coordinator chính trong mô hình mô phỏng.
2. **Cooja Simulator (Contiki-NG)** ([contiki-ng/contiki-ng](https://github.com/contiki-ng/contiki-ng)): Công cụ mô phỏng mạng cảm biến IoT không dây mạnh mẽ, cho phép mô phỏng các node IEEE 802.15.4 chạy Zigbee.
3. **KillerBee Framework** ([riverloopsec/killerbee](https://github.com/riverloopsec/killerbee)): Bộ công cụ kiểm thử bảo mật chuyên dụng cho chuẩn IEEE 802.15.4 và Zigbee, hỗ trợ phân tích gói tin capture được từ môi trường mô phỏng.
4. **SecBee Tool** ([iot-sec/secbee](https://github.com/iot-sec/secbee)): Công cụ khai thác lỗ hổng bảo mật của Zigbee, thích hợp chạy thử nghiệm trên các tệp pcap mô phỏng.
5. **Wireshark dissector for Zigbee**: Công cụ phân tích gói tin mạng hỗ trợ giải mã lưu lượng IEEE 802.15.4 ảo thu thập từ phần mềm mô phỏng.

---

### 5. Sản phẩm dự kiến
- **Báo cáo phân tích chi tiết**: Tài liệu PDF/Markdown trình bày cơ sở lý thuyết, sơ đồ kiến trúc hệ thống mô phỏng, phân tích lỗ hổng và kết quả mô phỏng.
- **Kịch bản & Môi trường Lab Mô phỏng**: Mã nguồn, file cấu hình (`configuration.yaml`, Docker Compose, file cấu hình mô phỏng Cooja `.csc`) dựng hệ thống mô phỏng phục vụ thử nghiệm bảo mật.
- **Tệp lưu vết gói tin mô phỏng (.pcap)**: Các mẫu gói tin thu giữ được từ môi trường mô phỏng trước và sau khi giải mã khóa mạng Zigbee phục vụ cho việc đối sánh chứng cứ.
- **Bảng so sánh cấu hình và Hướng dẫn phòng thủ**: Cung cấp cấu hình mẫu đã được tối ưu bảo mật.
- **Checklist kiểm toán**: File Excel/Markdown chứa danh sách các hạng mục cần kiểm tra để đảm bảo an toàn cho mạng Zigbee.

---

### 6. Danh mục tài liệu tham khảo (References)
1. **Contiki-NG & Cooja Contributors**. *Cooja Simulator for IoT/IEEE 802.15.4 Network Simulation*. Available at: [https://github.com/contiki-ng/contiki-ng](https://github.com/contiki-ng/contiki-ng).
2. **Wright, J. (2011)**. *KillerBee: Practical Zigbee Exploitation Framework*. SANS Technology Institute. Available at: [https://github.com/riverloopsec/killerbee](https://github.com/riverloopsec/killerbee).
3. **Koenkk**. *Zigbee2MQTT: Bridge Zigbee devices to MQTT*. Open-source documentation and repository. Available at: [https://github.com/Koenkk/zigbee2mqtt](https://github.com/Koenkk/zigbee2mqtt).
4. **Zigbee Alliance (2015)**. *Zigbee Specification Revision 21 (R21)*. Document 05-3474-21, Zigbee Alliance.
5. **Morgner, F., & Müller, T. (2017)**. *SecBee: Automated Zigbee Security Assessment*. Proceedings of the ACM Conference on Security and Privacy in Wireless and Mobile Networks (WiSec). Repo: [https://github.com/iot-sec/secbee](https://github.com/iot-sec/secbee).
6. **Z-Attack Contributors (2016)**. *Z-Attack: Zigbee Penetration Testing tool*. Source code repository. Available at: [https://github.com/SebaD/Z-Attack](https://github.com/SebaD/Z-Attack).
