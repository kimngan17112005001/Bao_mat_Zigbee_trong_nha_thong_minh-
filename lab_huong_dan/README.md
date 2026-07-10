# HƯỚNG DẪN DỰNG MÔI TRƯỜNG MÔ PHỎNG (LAB SIMULATION) & KIỂM CHỨNG BẢO MẬT ZIGBEE

Tài liệu này hướng dẫn chi tiết cách thiết lập một phòng thí nghiệm mô phỏng bảo mật Zigbee (Security Simulation Lab) nhằm thực hiện các kịch bản kiểm chứng lỗ hổng: **Nghe lén dữ liệu (Sniffing)**, **Trích xuất khóa mạng (Network Key Extraction)** và **Tấn công phát lại (Replay Attack)** trong môi trường phần mềm mô phỏng, đồng thời đưa ra cấu hình khắc phục lỗi.

---

## 1. Sơ đồ Kiến trúc Môi trường Mô phỏng (Simulation Architecture)

Hệ thống mô phỏng được triển khai hoàn toàn bằng phần mềm, sử dụng trình mô phỏng mạng **Cooja (Contiki-NG)** để giả lập các nút mạng Zigbee/IEEE 802.15.4 và môi trường truyền dẫn không dây ảo, kết hợp với các công cụ phân tích an toàn thông tin.

```mermaid
graph TD
    subgraph Môi trường Mô phỏng Cooja (Cooja Simulator)
        A[Node Coordinator ảo] <-->|Kênh truyền sóng 2.4GHz ảo| B[Node Cảm biến ảo]
        A <-->|Kênh truyền sóng 2.4GHz ảo| C[Node Bóng đèn ảo]
        D[Node Tấn công ảo - Attacker Node] -.->|Nghe lén lưu lượng sóng ảo| A
        D -.->|Phát lại gói tin giả mạo| A
    end

    subgraph Phân tích & Kiểm chứng (Analysis Tools)
        A -->|Xuất tệp lưu vết ảo| E[Tệp tin zigbee_sim.pcap]
        E -->|Phân tích & Giải mã| F[Wireshark]
        D -->|Sử dụng bộ công cụ| G[KillerBee / Scapy ảo]
    end

    style Môi trường Mô phỏng Cooja (Cooja Simulator) fill:#f5f5f5,stroke:#333,stroke-width:2px
    style D fill:#f9f,stroke:#333,stroke-width:2px
    style F fill:#9f9,stroke:#333,stroke-width:2px
```

### Thành phần phần mềm yêu cầu:
1. **Trình mô phỏng mạng Cooja (Contiki-NG)**: Dùng để thiết lập mạng lưới thiết bị Zigbee/IEEE 802.15.4 ảo.
2. **Wireshark**: Công cụ phân tích gói tin, được cấu hình bộ giải mã giao thức Zigbee để phân tích tệp `.pcap` xuất ra từ Cooja.
3. **Môi trường Docker**: (Tùy chọn) Chạy bộ tích hợp Zigbee2MQTT ảo với cổng nối tiếp ảo (`socat` hoặc dummy adapter) kết hợp với MQTT Broker để mô phỏng tầng ứng dụng.

---

## 2. Thiết lập Môi trường Mô phỏng mạng không dây với Cooja

### Bước 1: Khởi động Cooja
Khởi chạy Cooja thông qua môi trường Contiki-NG Docker hoặc chạy trực tiếp trên máy ảo Ubuntu:
```bash
cd contiki-ng/tools/cooja
./gradlew run
```

### Bước 2: Tạo Mô phỏng Mới (New Simulation)
1. Chọn **File** -> **New Simulation**. Đặt tên là `Zigbee_Security_Simulation`.
2. Thiết lập cấu hình vô tuyến (Radio Medium) sử dụng mô hình **UDGM (Unit Disk Graph Medium)** để mô phỏng vùng phủ sóng RF 2.4GHz.

### Bước 3: Thêm các Node mạng mô phỏng
1. Thêm 1 node đóng vai trò **Coordinator** (chạy firmware tạo lập mạng Zigbee ảo).
2. Thêm 1-2 node đóng vai trò **End Device** (giả lập cảm biến nhiệt độ hoặc bóng đèn).
3. Thêm 1 node đóng vai trò **Attacker (Sniffer/Injector)** nằm trong vùng phủ sóng không dây của Coordinator.

---

## 3. Kịch bản Mô phỏng Tấn công Bảo mật (PoC Simulation)

### Kịch bản 1: Mô phỏng Sniffing và Giải mã khóa Network Key (Khóa mạng)

*Lỗ hổng mô phỏng: Giao thức truyền khóa Network Key được mã hóa bằng khóa liên kết mặc định (`ZigBeeAlliance09`) khi thiết bị ảo gia nhập mạng.*

#### Bước 1: Thu thập gói tin từ sóng ảo Cooja
1. Trong giao diện Cooja, kích hoạt tính năng **Radio Logger** (chọn **Tools** -> **Radio Logger**).
2. Tại Radio Logger, chọn định dạng xuất dữ liệu là **PCAP** (định dạng Wireshark). Ghi lưu lượng ra tệp `zigbee_sim.pcap`.
3. Nhấp nút **Start** trong Cooja để bắt đầu chạy mô phỏng.
4. Kích hoạt pha ghép đôi (Pairing) bằng cách cho phép Node End Device ảo gửi yêu cầu gia nhập mạng (Association Request) tới Node Coordinator ảo.

#### Bước 2: Cấu hình Wireshark giải mã tệp PCAP mô phỏng
1. Mở tệp `zigbee_sim.pcap` bằng Wireshark.
2. Chọn **Edit** -> **Preferences** -> **Protocols** -> **ZigBee**.
3. Tại mục **Pre-configured Keys**, nhấn **Edit** và thêm khóa liên kết mặc định toàn cầu của Zigbee Alliance:
   - Key: `5A:69:67:42:65:65:41:6C:6C:69:61:6E:63:65:30:39` (Chuỗi ASCII của `ZigBeeAlliance09`).
   - Byte Order: `Normal`.
4. Nhấn OK và tích chọn **Enable Decryption**.
5. Wireshark sẽ tự động giải mã gói tin **"Transport Key"** trong pha bắt tay ảo, làm lộ rõ khóa mạng **Network Key** dạng clear text dưới dạng Hex.
6. Copy khóa mạng này dán lại vào danh sách Keys của Wireshark để giải mã 100% các dữ liệu cảm biến ảo tiếp theo.

---

### Kịch bản 2: Mô phỏng tấn công phát lại (Replay Attack) điều khiển thiết bị

*Lỗ hổng mô phỏng: Node tấn công ảo phát lại gói tin điều khiển hợp lệ đã bắt được để thay đổi trạng thái thiết bị ảo mà không cần biết khóa mạng.*

#### Bước 1: Trích xuất gói tin điều khiển ảo
Từ tệp `zigbee_sim.pcap`, tìm gói tin ra lệnh bật/tắt của Coordinator ảo gửi đến bóng đèn ảo. Trích xuất gói tin này dưới dạng hex hoặc file pcap thô (`control_cmd.pcap`).

#### Bước 2: Thực hiện phát lại lệnh trong Cooja
1. Sử dụng tính năng gửi gói tin tùy chỉnh (Packet Sender) của Node Attacker ảo hoặc viết script python tích hợp Scapy để tiêm (inject) gói tin vào Radio Medium của Cooja.
2. Script Python mô phỏng phát lại:
   ```python
   from scapy.all import *
   # Đọc gói tin điều khiển đã lưu từ môi trường mô phỏng
   packet = rdpcap("control_cmd.pcap")[0]
   # Phát lại gói tin qua interface mạng ảo của Cooja
   send(packet, iface="tun0")
   ```
3. Kết quả mô phỏng: Trình mô phỏng Cooja hiển thị Node Bóng đèn nhận được gói tin, chấp nhận lệnh và đổi trạng thái (On/Off) mà không phát hiện ra gói tin này do kẻ tấn công phát lại.

---

## 4. Hướng dẫn cấu hình Phòng thủ trên hệ thống Mô phỏng (Hardening)

Để kiểm chứng hiệu quả của các cơ chế phòng thủ trong mô phỏng, thực hiện các cấu hình sau trên tệp tin `configuration.yaml` của Gateway ảo hoặc trong mã nguồn firmware của các node:

1. **Sinh khóa mạng ngẫu nhiên**: Trong cấu hình `configuration.yaml` của Zigbee2MQTT ảo:
   ```yaml
   advanced:
     network_key: generate
   ```
   Hệ thống mô phỏng sẽ tự sinh khóa mạng mới ngẫu nhiên thay vì sử dụng khóa cố định.
2. **Kích hoạt Install Code (Unique Link Key)**:
   - Trên bộ điều phối ảo, chỉ định khóa liên kết cụ thể cho từng địa chỉ MAC ảo của Node End Device thay vì chấp nhận khóa liên kết mặc định `ZigBeeAlliance09`.
3. **Mặc định đóng Permit Join**:
   ```yaml
   permit_join: false
   ```
   Chỉ cho phép mở trong thời gian giới hạn (ví dụ: 60 giây) qua lệnh điều khiển.
