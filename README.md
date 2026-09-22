# Smart-Home-IoT-System
# DALN
# HỆ THỐNG NHÀ THÔNG MINH IOT GIÁM SÁT MÔI TRƯỜNG VÀ ĐIỀU KHIỂN TỰ ĐỘNG HÓA AN TOÀN

> **Đồ án môn học / Đồ án liên ngành**  
> **Sinh viên thực hiện:** Phạm Khắc Hùng  
> **Giáo viên hướng dẫn:** TS. Nguyễn Lệ Thu  
> **Nền tảng triển khai:** Vi điều khiển ESP32, Mosquitto MQTT Broker & Ứng dụng điều khiển đa nền tảng  

---

## 1. Giới thiệu bài toán & Mục tiêu đề tài

### 1.1. Bối cảnh và Thách thức
Trong xu hướng phát triển đô thị thông minh và nâng cao chất lượng đời sống gia đình, các hệ sinh thái điện gia dụng truyền thống bộc lộ nhiều điểm nghẽn:
- **Thiếu khả năng cảnh báo sớm:** Các nguy cơ cháy nổ, rò rỉ khí gas sinh hoạt (LPG) thường chỉ được phát hiện khi thiệt hại đã phát sinh, thiếu cơ chế ngắt mạch và báo động tức thì từ xa.
- **Lãng phí năng lượng:** Thiết bị chiếu sáng, làm mát vận hành thụ động theo thao tác cơ học của con người, không tự động tối ưu theo biến thiên thông số môi trường thực tế (nhiệt độ, độ ẩm).
- **Phụ thuộc kết nối đám mây (Cloud Latency & Privacy):** Nhiều giải pháp thương mại phụ thuộc hoàn toàn vào máy chủ nước ngoài, dẫn đến độ trễ cao, tiềm ẩn rủi ro lộ lọt dữ liệu sinh hoạt và ngừng trệ điều khiển khi mất đường truyền Internet quốc tế.

### 1.2. Mục tiêu đề tài
Xây dựng một hệ thống **Nhà thông minh IoT (Internet of Things)** toàn diện từ biên đến giao diện người dùng, hoạt động tin cậy theo thời gian thực:
- Thiết kế hệ thống mạng cảm biến thu thập thông số môi trường liên tục, phản ứng tức thì khi vượt ngưỡng an toàn.
- Xây dựng cơ chế truyền thông hai chiều thời gian thực (Real-time Full-duplex) với độ trễ thấp thông qua giao thức truyền thông nhẹ **MQTT**.
- Phát triển kịch bản tự động hóa biên (**Edge Automation**): Vi điều khiển tự động đóng/cắt rơ-le và hú còi cảnh báo độc lập ngay cả khi mất liên lạc với máy chủ trung tâm.
- Triển khai giao diện Dashboard/Mobile App trực quan cho phép giám sát dữ liệu chuỗi thời gian (Time-series) và điều khiển thiết bị mọi lúc, mọi nơi.

---

## 2. Kiến trúc phần cứng hệ thống

Hệ thống được module hóa, chia thành các khối chấp hành và giám sát chuyên biệt:
- **Bộ điều khiển trung tâm tại trạm (IoT Edge Node):** Bo mạch vi điều khiển **ESP32 DevKit V1** (Chip ESP32 32-bit Xtensa Dual-Core LX6 @ 240 MHz, tích hợp Wi-Fi 802.11 b/g/n và BLE 4.2).
- **Khối cảm biến môi trường & khí độc:**
  - Cảm biến nhiệt độ, độ ẩm kỹ thuật số **DHT22 / AM2302** (Dải đo $-40^\circ\text{C} \div 80^\circ\text{C}$, sai số $\pm 0.5^\circ\text{C}$; độ ẩm $0 \div 100\% \text{ RH}$, sai số $\pm 2\%$).
  - Cảm biến khí gas / khói quang hóa **MQ-2** (Độ nhạy cao với LPG, Propane, Smoke, dải nồng độ phát hiện $300 \div 10.000\text{ ppm}$).
- **Khối đóng cắt & Cảnh báo:**
  - Module **Relay 4 kênh cách ly quang (Optocoupler)** 5V/10A điều khiển phụ tải 220VAC (đèn chiếu sáng, quạt làm mát).
  - Còi báo động áp điện **Active Buzzer 5V** (cường độ âm thanh $\ge 85\text{ dB}$).
- **Khối nguồn:** Mạch hạ áp ổn định điện áp xung DC-DC 5V/3A bảo đảm dòng tải tức thời khi đóng/ngắt các cuộn hút rơ-le và truyền tải sóng Wi-Fi công suất cực đại (TX Power 20 dBm).

---

## 3. Pipeline truyền thông và Thuật toán điều khiển

### 3.1. Tiền xử lý tín hiệu cảm biến tại biên (Edge Pre-processing)
Dữ liệu từ các cổng GPIO được chuẩn hóa trước khi đóng gói truyền thông:
1. **Lọc trung bình trượt (Moving Average Filter):** Thực hiện lấy mẫu $N = 10$ chu kỳ đối với tín hiệu ADC 12-bit từ cảm biến MQ-2, loại bỏ hoàn toàn các đỉnh nhiễu đột biến do xung nguồn.
2. **Kỹ thuật định thời bất đồng bộ (Non-blocking Timer):** Sử dụng hàm thời gian vi mạch `millis()` thay thế toàn bộ lệnh tạm dừng tĩnh `delay()`, bảo đảm chu kỳ quét cảm biến $2000\text{ ms/lần}$ không làm gián đoạn vòng lặp lắng nghe bản tin điều khiển.
3. **Cơ chế Failsafe nội tại:** Vi điều khiển tự động kích hoạt còi Buzzer và ngắt tải liên quan ngay tại tầng phần cứng nếu nồng độ gas vượt ngưỡng $T_{\text{threshold}} \ge 400\text{ ppm}$ mà không cần chờ lệnh xác nhận từ Backend Server.

### 3.2. Cấu trúc đóng gói dữ liệu & Chuẩn truyền thông MQTT
- **Phương thức truyền nhận:** Định dạng chuẩn **JSON** gói gọn tải trọng (payload) nhỏ gọn, tối ưu hóa băng thông mạng Wi-Fi nội bộ.
- **Định danh Topic phân cấp:**
  - Trạng thái cảm biến: `home/livingroom/sensors/data`  
    Payload mẫu: `{"temp": 28.4, "hum": 65.2, "gas": 128, "status": "SAFE"}`
  - Lệnh điều khiển rơ-le: `home/livingroom/relay/{id}/set` (Payload: `ON` / `OFF`)
  - Báo cáo phản hồi xác nhận: `home/livingroom/relay/{id}/status` (Payload: `ON` / `OFF`)
- **Chất lượng dịch vụ (QoS):** Sử dụng **QoS 1 (At least once)** cho lệnh điều khiển bảo đảm thiết bị luôn nhận được chỉ thị; sử dụng **QoS 0** cho luồng telemetry để giảm tải mạng.

---

## 4. Thiết kế Giao diện giám sát & Điều khiển (UI/UX)

Hệ thống giao diện được thiết kế hiện đại, hỗ trợ tương tác trực quan thời gian thực:
- **Khu vực hiển thị thẻ trạng thái môi trường (Dashboard Cards):**
  - Đồng hồ đo nhiệt độ, độ ẩm hiển thị trực quan dải giá trị an toàn/nguy hiểm bằng dải màu Gradient thích ứng.
  - Đồ thị biến thiên nhiệt độ và nồng độ khí gas theo thời gian thực (Time-series Chart) cập nhật chu kỳ từng giây.
- **Khu vực điều khiển phụ tải (Device Control Panel):**
  - Hệ thống công tắc chuyển đổi (Switch Toggle) cảm ứng điều khiển bật/tắt tức thì từng kênh Relay.
  - Đèn LED trạng thái ảo (Feedback Indicator) phản ánh chính xác trạng thái thực của thiết bị tại hiện trường thông qua kênh subscribe MQTT.
- **Trung tâm thông báo & Báo động khẩn:**
  - Popup cảnh báo màu đỏ toàn màn hình kèm âm thanh khi phát hiện nồng độ gas/khói vượt mức báo động.
  - Nhật ký sự kiện (Event Logger) ghi lại chính xác thời điểm bật/tắt thiết bị và lịch sử phát sinh sự cố an toàn.

---

## 5. Kết quả thực nghiệm đạt được

| Chỉ số đánh giá | Kết quả đạt được | Ý nghĩa kỹ thuật |
| :--- | :---: | :--- |
| **Độ trễ truyền nhận lệnh (MQTT)** | **$42.5\text{ ms}$** | Phản hồi điều khiển đèn/quạt gần như tức thì trong mạng nội bộ |
| **Tỷ lệ truyền gói tin thành công** | **$99.8\%$** | Đảm bảo tính toàn vẹn dữ liệu trong môi trường Wi-Fi dân dụng |
| **Thời gian phản ứng Failsafe** | **$< 50\text{ ms}$** | Kích hoạt còi báo động ngay lập tức khi phát hiện khí gas vượt ngưỡng |
| **Sai số đo nhiệt độ / độ ẩm** | **$\le \pm 0.5^\circ\text{C} \ / \ \pm 2\%$** | Đáp ứng tiêu chuẩn giám sát vi khí hậu phòng ở |
| **Thời gian hoạt động liên tục (Uptime)**| **$168\text{ giờ}$** | Hệ thống chạy thử nghiệm 7 ngày liên tục không phát sinh lỗi tràn bộ nhớ (Memory Leak) |
| **Điện năng tiêu thụ trạm biên** | **$\sim 0.65\text{W}$** | Vi điều khiển vận hành tối ưu năng lượng ở chế độ thu phát bình thường |

---

## 6. Cấu trúc thư mục mã nguồn

```text
├── hardware/
│   ├── src/
│   │   ├── main.cpp            # Vòng lặp chính, kết nối Wi-Fi & MQTT Client
│   │   ├── sensor_dht.cpp      # Module đọc và chuẩn hóa dữ liệu DHT22
│   │   ├── sensor_mq2.cpp      # Module tiền xử lý, lọc nhiễu analog khí gas
│   │   └── relay_control.cpp   # Quản lý kích hoạt rơ-le và cơ chế Failsafe
│   ├── include/
│   │   └── config.h            # Cấu hình GPIO Pin, ngưỡng báo động & MQTT broker
│   └── platformio.ini          # Quản lý thư viện phụ thuộc (PubSubClient, ArduinoJson)
├── server/
│   ├── broker/
│   │   └── mosquitto.conf      # Tệp cấu hình cổng và xác thực Mosquitto Broker
│   ├── backend_service/        # API điều khiển và ghi log cơ sở dữ liệu
│   └── docker-compose.yml      # Đóng gói triển khai nhanh Broker & Database
├── client_app/
│   ├── src/
│   │   ├── components/         # Thẻ hiển thị cảm biến, nút gạt điều khiển
│   │   ├── services/           # Xử lý kết nối WebSocket / MQTT qua giao diện
│   │   └── App.vue / main.dart # Giao diện ứng dụng điều khiển trung tâm
│   └── package.json
└── README.md                   # Tài liệu thuyết minh dự án
