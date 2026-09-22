# Smart-Home-IoT-System
# DALN
# 🏠 Smart Home IoT System (Hệ Thống Nhà Thông Minh)

> **Đồ án môn học / Đồ án tốt nghiệp Công nghệ Thông tin**  
> **Sinh viên thực hiện:** Phạm Khắc Hùng  
> **Nền tảng triển khai:** Vi điều khiển ESP32, RESTful API / MQTT Gateway, Cloud Backend & Dashboard/Mobile App  

---

## 📌 1. Giới thiệu tổng quan

Hệ thống **Nhà thông minh IoT (Internet of Things)** cung cấp giải pháp toàn diện từ thu thập dữ liệu phần cứng, điều hướng dữ liệu qua tầng API trung gian lên nền tảng Cloud, đến điều khiển thiết bị và tự động hóa cảnh báo an toàn thời gian thực.

Hệ thống giải quyết 3 bài toán trọng tâm:
- **Thu thập & Điều hướng dữ liệu (Data Routing):** Xây dựng tầng API tiếp nhận dữ liệu đo lường từ phần cứng (ESP32), phân loại và định tuyến thông minh lên Cloud Database và dịch vụ cảnh báo.
- **Điều khiển 2 chiều thời gian thực:** Bật/tắt đèn, quạt, bơm nước từ xa qua Internet/LAN với độ trễ phản hồi thấp (< 100ms) kèm cơ chế xác thực trạng thái.
- **Bảo vệ an toàn biên (Hardware Failsafe):** Tự động phát hiện rò rỉ khí gas/khói, kích hoạt còi báo động tại chỗ và ngắt rơ-le phụ tải ngay tại vi điều khiển mà không cần chờ lệnh từ Cloud.

---

## 🚀 2. Tính năng chính

- [x] **Thu thập dữ liệu môi trường:** Đọc và chuẩn hóa dữ liệu từ cảm biến nhiệt độ - độ ẩm DHT22 và cảm biến khí gas MQ-2.
- [x] **API điều hướng lên Cloud:** Tự động đóng gói JSON, xác thực Device Token và điều hướng gói tin từ phần cứng lên Cloud Server.
- [x] **Điều khiển thời gian thực:** Điều khiển rơ-le qua giao thức MQTT / HTTP RESTful API với cơ chế phản hồi xác nhận trạng thái (Status Feedback).
- [x] **Cảnh báo khẩn cấp:** Cảm biến gas MQ-2 kích hoạt còi hú (Buzzer) tại chỗ và kích hoạt luồng cảnh báo khẩn cấp đẩy về điện thoại/Web.
- [x] **Biểu đồ trực quan:** Trực quan hóa dữ liệu chuỗi thời gian (Time-series data) dạng biểu đồ đường trên Dashboard.
- [x] **Cơ chế Local Fallback:** Cho phép thiết bị và bảng điều khiển tiếp tục tương tác trong mạng nội bộ (LAN) khi mất kết nối Internet.

---

## 🏗️ 3. Kiến trúc hệ thống

```text
[ Cảm biến & Thiết bị ]
 (DHT22, MQ-2, Relay, Buzzer)
              │
              ▼ (GPIO / ADC)
       [ ESP32 Edge Node ]
              │
              ▼ (HTTP POST / MQTT Publish kèm Device Token)
┌─────────────────────────────────────────────────────────────┐
│             HỆ THỐNG API GATEWAY ĐIỀU HƯỚNG                 │
│  - Tiếp nhận & Giải mã gói tin (Payload Ingestion)           │
│  - Xác thực thiết bị (Authentication & Rate Limiting)       │
│  - Bộ phân luồng & Điều hướng dữ liệu (Data Dispatcher)     │
└───┬─────────────────────────┬───────────────────────────┬───┘
    │ Luồng cảnh báo khẩn     │ Luồng dữ liệu định kỳ     │ Trạng thái phản hồi
    ▼                         ▼                           ▼
[ Cloud Alert Service ]   [ Cloud Database ]       [ Dashboard / Mobile ]
(FCM / Push Notification) (Lưu trữ lịch sử)        (Giám sát & Điều khiển)
```

---

## 🛠️ 4. Ngăn xếp công nghệ (Tech Stack)

| Phân hệ | Công nghệ / Linh kiện sử dụng | Vai trò kỹ thuật |
| :--- | :--- | :--- |
| **Phần cứng** | ESP32 DevKit V1, DHT22, MQ-2, Relay 4 kênh, Active Buzzer | Thu thập dữ liệu cảm biến và đóng ngắt tải điện |
| **Firmware** | C/C++ (VS Code + PlatformIO / Arduino IDE) | Xử lý logic nhúng, lọc nhiễu, kết nối Wi-Fi & gọi API/MQTT |
| **Thư viện nhúng** | `PubSubClient`, `DHT sensor library`, `ArduinoJson`, `HTTPClient` | Đóng gói JSON và gửi nhận dữ liệu mạng |
| **Message Broker** | Eclipse Mosquitto (MQTT v3.1.1 / v5.0) | Điều phối bản tin publish/subscribe tốc độ cao |
| **API / Backend** | Node.js (Express / NestJS) hoặc Python, Docker | Tiếp nhận request từ ESP32, xác thực token và điều hướng dữ liệu |
| **Cơ sở dữ liệu** | PostgreSQL / MongoDB / InfluxDB | Lưu trữ tài khoản, thiết bị và dữ liệu môi trường chuỗi thời gian |
| **Giao diện (UI)** | Web Dashboard (HTML5, TailwindCSS, Chart.js) / Flutter | Trực quan hóa dữ liệu và điều khiển thiết bị |

---

## 🔌 5. Sơ đồ nối dây phần cứng (Pinout Reference)

| Linh kiện / Module | Chân Module | Chân GPIO ESP32 | Chế độ / Ghi chú |
| :--- | :--- | :--- | :--- |
| **DHT22** | DATA | `GPIO 4` | Kéo điện trở 10kΩ lên nguồn 3.3V |
| **MQ-2** | A0 (Analog) | `GPIO 34` | Đọc mức tín hiệu qua kênh ADC1 (Input Only) |
| **Relay Kênh 1 (Đèn)** | IN1 | `GPIO 18` | Kích mức thấp (Active LOW 0V) |
| **Relay Kênh 2 (Quạt)** | IN2 | `GPIO 19` | Kích mức thấp (Active LOW 0V) |
| **Active Buzzer** | Chân tín hiệu | `GPIO 21` | Phát còi hú cảnh báo rò rỉ khí gas |
| **Nguồn hệ thống** | 5V / GND | `VIN` / `GND` | Nguồn cấp ngoài từ mạch Buck hạ áp DC 5V/3A |

---

## 📡 6. Thiết kế API & MQTT Topic Điều hướng

### 6.1. Cấu trúc Payload JSON từ phần cứng gửi lên Cloud
```json
{
  "device_id": "ESP32_SMARTHOME_01",
  "token": "sec_token_9f8a2b3c4d",
  "timestamp": 1727002800,
  "data": {
    "temperature": 28.5,
    "humidity": 65.2,
    "gas_level": 125,
    "relay_status": {
      "relay_1": 1,
      "relay_2": 0
    }
  }
}
```

### 6.2. Danh sách Endpoints API điều hướng
| Phương thức | Endpoint | Chức năng điều hướng |
| :---: | :--- | :--- |
| `POST` | `/api/v1/devices/telemetry` | Tiếp nhận dữ liệu định kỳ, bóc tách và ghi vào Cloud Database |
| `POST` | `/api/v1/devices/alert` | Kênh ưu tiên: Bỏ qua hàng đợi, kích hoạt dịch vụ thông báo khẩn |
| `GET` | `/api/v1/devices/config/{id}` | Phần cứng lấy cấu hình ngưỡng đo và chu kỳ gửi từ Cloud |

### 6.3. Cấu trúc MQTT Topics
* **Dữ liệu cảm biến:** `smarthome/telemetry` (Payload: JSON số liệu đo)
* **Lệnh điều khiển:** `smarthome/relay1/set` (Payload: `"ON"` hoặc `"OFF"`)
* **Xác nhận trạng thái:** `smarthome/relay1/state` (Payload: `"ON"` hoặc `"OFF"`)
* **Cảnh báo khẩn cấp:** `smarthome/alert` (Payload: JSON sự cố gas/cháy)

---

## 📁 7. Cấu trúc thư mục dự án

```text
smart-home-iot/
├── hardware/                  # Mã nguồn vi điều khiển ESP32
│   ├── src/
│   │   ├── main.cpp           # Luồng đọc cảm biến, gửi dữ liệu & nhận lệnh
│   │   ├── api_client.cpp     # Module đóng gói JSON và gọi API lên Cloud
│   │   └── relay_control.cpp  # Xử lý kích hoạt rơ-le và ngắt Failsafe
│   ├── include/
│   │   └── config.h           # Cấu hình Wi-Fi, Cloud API URL và chân GPIO
│   └── platformio.ini         # Cấu hình môi trường nạp code PlatformIO
├── server/                    # API Gateway & Dịch vụ Cloud
│   ├── src/
│   │   ├── routes/            # Khai báo các API Endpoints điều hướng
│   │   ├── controllers/       # Tiếp nhận và xử lý gói tin từ phần cứng
│   │   └── services/          # Phân luồng dữ liệu sang Database & Notification
│   ├── mosquitto/
│   │   └── mosquitto.conf     # File cấu hình cổng và quyền Broker
│   ├── Dockerfile
│   └── docker-compose.yml     # Khởi chạy tự động Broker, API Server & Database
├── client/                    # Giao diện người dùng (Dashboard / Mobile App)
│   ├── index.html             # Giao diện Web Dashboard
│   ├── css/style.css
│   └── js/app.js              # Kết nối API & WebSocket hiển thị thời gian thực
├── docs/                      # Sơ đồ mạch nguyên lý, tài liệu báo cáo
└── README.md                  # Tài liệu thuyết minh dự án
```

---

## ⚙️ 8. Hướng dẫn cài đặt & Triển khai

### Bước 1: Khởi chạy API Gateway & MQTT Broker trên Cloud/Server
Yêu cầu máy chủ đã cài đặt **Docker** và **Docker Compose**:
```bash
cd server
docker-compose up -d
```
*(Hệ thống sẽ khởi chạy API Gateway tại cổng `5000`, Mosquitto Broker tại cổng `1883` và WebSocket tại cổng `9001`).*

### Bước 2: Nạp mã nguồn cho ESP32
1. Mở thư mục `hardware` bằng **VS Code** (đã cài đặt tiện ích mở rộng **PlatformIO IDE**).
2. Điều chỉnh thông số Wi-Fi và Endpoint Cloud trong file `hardware/include/config.h`:
   ```cpp
   const char* ssid = "TEN_WIFI_CUA_BAN";
   const char* password = "MAT_KHAU_WIFI";
   const char* cloud_api_url = "http://dia-chi-cloud-ip:5000/api/v1/devices/telemetry";
   const char* device_token = "sec_token_9f8a2b3c4d";
   ```
3. Cắm cáp Micro-USB kết nối ESP32 với máy tính và nhấn nút **Upload** trên thanh trạng thái PlatformIO.

### Bước 3: Mở giao diện điều khiển (Dashboard)
1. Mở trực tiếp file `client/index.html` trên trình duyệt hoặc chạy qua tiện ích **Live Server**.
2. Nhập địa chỉ Cloud IP và mã Token của thiết bị.
3. Bấm **Kết nối** để theo dõi biểu đồ cảm biến và điều khiển thiết bị thời gian thực.

---

## 👥 9. Thông tin tác giả

* **Sinh viên thực hiện:** Phạm Khắc Hùng  
* **Ngành:** Công nghệ Thông tin  
* **Đề tài:** Hệ thống Nhà thông minh IoT (Smart Home System)  

---

## 📄 10. Giấy phép (License)

Dự án được phân phối dưới giấy phép **MIT License**.
