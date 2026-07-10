# HƯỚNG DẪN DỰNG PHÒNG THÍ NGHIỆM (LAB) & KIỂM CHỨNG BẢO MẬT ZIGBEE

Tài liệu này hướng dẫn chi tiết cách thiết lập một phòng thí nghiệm bảo mật Zigbee tối thiểu (Minimal Security Lab) để thực nghiệm các lỗ hổng bảo mật: **Nghe lén dữ liệu (Sniffing)**, **Trích xuất khóa mạng (Network Key Extraction)** và **Tấn công phát lại (Replay Attack)**, đồng thời đưa ra phương án cấu hình khắc phục lỗi.

---

## 1. Sơ đồ Kiến trúc Lab (Lab Architecture)

Mô hình lab sử dụng một mạng nội bộ kết hợp giữa phần cứng thực tế và các container ảo hóa qua Docker.

```mermaid
graph TD
    subgraph Kẻ tấn công (Attacker)
        A[Laptop chạy Kali Linux] <-->|USB| B[USB Sniffer CC2531 / KillerBee]
        B -.->|Sniff Sóng RF 2.4GHz| E
    end

    subgraph Hệ thống Smart Home (Victim)
        C[Home Assistant Core / Docker] <-->|MQTT over TLS| D[Zigbee2MQTT Bridge]
        D <-->|USB Serial| E[Zigbee Coordinator - Dongle CC2652P]
        E -.->|Mạng Zigbee mã hóa yếu| F[Bóng đèn Zigbee Smart Light]
        E -.->|Mạng Zigbee mã hóa yếu| G[Cảm biến cửa Zigbee Sensor]
    end

    style A fill:#f9f,stroke:#333,stroke-width:2px
    style B fill:#ff9,stroke:#333,stroke-width:2px
    style E fill:#9f9,stroke:#333,stroke-width:2px
```

### Thành phần phần cứng và phần mềm yêu cầu:
1. **Thiết bị Coordinator (Nạn nhân)**: USB Dongle CC2652P hoặc CC2531 nạp firmware Koenkk.
2. **Thiết bị Sniffer/Attacker (Kẻ tấn công)**: USB Dongle CC2531 nạp firmware sniffer (`ccsniffer` của Texas Instruments hoặc firmware tương thích với KillerBee).
3. **Môi trường giả lập**: Máy tính chạy Linux/Windows Docker chứa:
   - Eclipse Mosquitto (MQTT Broker).
   - Zigbee2MQTT.
   - Home Assistant (tùy chọn, để xem giao diện trực quan).
4. **Hệ điều hành Attacker**: Kali Linux hoặc Ubuntu cài đặt **Wireshark** và **KillerBee/SecBee**.

---

## 2. Thiết lập Môi trường chạy Lab (Docker-Compose)

Tạo tệp tin `docker-compose.yml` tại thư mục Lab để khởi chạy các dịch vụ MQTT và Zigbee2MQTT.

```yaml
version: '3.8'

services:
  mqtt:
    image: eclipse-mosquitto:2.0
    container_name: mosquitto
    restart: always
    ports:
      - "1883:1883" # Cổng mặc định không mã hóa (phục vụ test)
      - "8883:8883" # Cổng an toàn TLS
    volumes:
      - ./mosquitto/config:/mosquitto/config
      - ./mosquitto/data:/mosquitto/data
      - ./mosquitto/log:/mosquitto/log

  zigbee2mqtt:
    image: koenkk/zigbee2mqtt:latest
    container_name: zigbee2mqtt
    restart: always
    volumes:
      - ./configuration.yaml:/app/data/configuration.yaml
      - /run/udev:/run/udev:ro
    ports:
      - "8080:8080" # Cổng Frontend
    devices:
      - "/dev/ttyUSB0:/dev/ttyUSB0" # Map USB Coordinator thực tế vào container
    depends_on:
      - mqtt
```

> [!NOTE]
> Tạo thư mục cấu hình cho Mosquitto: `./mosquitto/config/mosquitto.conf` với thiết lập cho phép anonymous phục vụ cho việc giả lập pha cấu hình mặc định (kém an toàn) trước.

---

## 3. Kịch bản Kiểm chứng Bảo mật (PoC Security Verification)

### Kịch bản 1: Sniffing và Giải mã khóa Network Key (Khóa mạng)

*Lỗ hổng: Giao thức Zigbee truyền khóa Network Key được mã hóa bằng khóa liên kết mặc định (`ZigBeeAlliance09`) khi có thiết bị mới gia nhập mạng.*

#### Bước 1: Cấu hình USB Sniffer
Cắm USB CC2531 nạp firmware Sniffer vào máy chạy Kali Linux. Sử dụng công cụ `zbdump` từ bộ công cụ **KillerBee** để dò kênh mạng:
```bash
# Liệt kê các thiết bị sniffer đang cắm
zbusb

# Kiểm tra sóng RF trên kênh 11 (kênh mặc định) và ghi file pcap
zbdump -i hw:1:1 -c 11 -w zigbee_sniff.pcap
```

#### Bước 2: Cấu hình Wireshark giải mã
1. Mở Wireshark, chọn **Edit** -> **Preferences** -> **Protocols** -> **ZigBee**.
2. Tại mục **Pre-configured Keys**, nhấn **Edit** và thêm khóa liên kết mặc định của Zigbee Alliance:
   - Key: `5A:69:67:42:65:65:41:6C:6C:69:61:6E:63:65:30:39` (Chuỗi ASCII của `ZigBeeAlliance09`).
   - Byte Order: `Normal` hoặc `Reverse` tùy theo firmware.
3. Kích hoạt tính năng **Decryption** trong cài đặt ZigBee.

#### Bước 3: Bẫy pha ghép đôi (Pairing Phase)
1. Trên giao diện Zigbee2MQTT, nhấn nút "Permit Join" (Cho phép gia nhập).
2. Nhấn nút Reset trên cảm biến cửa Zigbee hoặc bóng đèn thông minh để thiết bị tiến hành Pairing lại.
3. Gói tin trao đổi khóa mạng **"Transport Key"** từ Coordinator tới Device sẽ bị bắt lại bởi Wireshark.
4. Xem chi tiết gói tin thuộc tầng bảo mật APS (Application Support Sublayer). Bạn sẽ thấy trường `Network Key` được giải mã rõ ràng dạng Hex trong Wireshark nhờ khóa liên kết mặc định.
5. Sao chép Khóa mạng này và dán thêm vào danh sách **Pre-configured Keys** của Wireshark. Kể từ thời điểm này, Wireshark có thể giải mã 100% gói tin dữ liệu cảm biến và lệnh điều khiển chuyển trạng thái (On/Off).

---

### Kịch bản 2: Tấn công phát lại (Replay Attack) điều khiển thiết bị

*Lỗ hổng: Kẻ tấn công bắt được khung tin mã hóa ra lệnh bật/tắt thiết bị và phát lại chính xác khung tin đó để điều khiển thiết bị mà không cần giải mã, nếu lớp mạng không kiểm tra chặt chẽ số thứ tự khung tin (Frame Counter).*

#### Bước 1: Xác định Frame điều khiển
Sử dụng bộ lọc Wireshark để lọc các khung tin điều khiển bật/tắt bóng đèn (thường là gói tin ứng dụng ZCL - Zigbee Cluster Library):
```text
zbee_nwk && zbee_aps.cmd.id == 0x01
```
Ghi nhận giá trị Sequence Number và Frame Counter của gói tin đó.

#### Bước 2: Thực hiện phát lại gói tin
Sử dụng công cụ `zbreplay` từ framework **KillerBee**:
```bash
# Phát lại gói tin thu được từ file PCAP để kích hoạt bật đèn
zbreplay -i hw:1:1 -f zigbee_sniff.pcap -d 0x12ab -s 5
```
*Giải thích*: `-d` là PAN ID mục tiêu, `-s` là số giây trễ. Nếu thành công, bóng đèn thông minh sẽ tự động chuyển trạng thái ngay cả khi kẻ tấn công không biết nội dung giải mã của khóa mạng.

---

## 4. Hướng dẫn cấu hình Phòng thủ (Hardening Guide)

Để ngăn chặn các kịch bản tấn công trên, quản trị viên cần thay đổi tệp tin `configuration.yaml` tại thư mục cài đặt của Zigbee2MQTT như sau:

1. **Khóa mạng ngẫu nhiên**: Đảm bảo cấu hình `network_key: generate` để hệ thống tự sinh khóa mới ngẫu nhiên, không dùng khóa mặc định.
2. **Khóa liên kết thiết bị (Zigbee 3.0 Install Codes)**: Thay vì cho phép dùng `ZigBeeAlliance09` để truyền khóa mạng, hãy kích hoạt cơ chế Install Code cho các thiết bị hỗ trợ.
   - Truy cập giao diện Web của Zigbee2MQTT -> Chọn **Settings** -> **Install Codes**.
   - Nhập khóa cài đặt từ mã QR/nhãn in trên thiết bị trước khi cho ghép đôi.
3. **Hạn chế Permit Join**:
   ```yaml
   permit_join: false
   ```
   Chỉ mở bằng tay và sử dụng hệ thống giám sát tự động tắt sau 60 giây.
