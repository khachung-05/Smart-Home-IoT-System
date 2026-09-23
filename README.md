# DALN
# 🏠 Smart Home IoT System (Hệ Thống Nhà Thông Minh)

> **Đồ án môn học / Đồ án liên ngành Công nghệ Thông tin**  
> **Sinh viên thực hiện:** Phạm Khắc Hùng 
> **Lĩnh vực:** Nhúng IoT – API điều hướng dữ liệu từ thiết bị phần cứng lên Cloud 
> **Nền tảng triển khai:** Vi điều khiển ESP8266, Node.js RESTful API Gateway, SQLite Time-Series Database & Web Dashboard

---

## 📌 1. Giới thiệu tổng quan[cite: 9]

Hệ thống **Nhà thông minh IoT (Internet of Things)** cung cấp giải pháp toàn diện từ thu thập dữ liệu phần cứng, điều hướng dữ liệu qua tầng API trung gian lên máy chủ, đến điều khiển thiết bị và tự động hóa cảnh báo an toàn thời gian thực[cite: 9].

Hệ thống giải quyết 3 bài toán trọng tâm[cite: 9]:
- **Thu thập & Điều hướng dữ liệu (Data Ingestion & Routing):** Xây dựng tầng API RESTful tiếp nhận dữ liệu đo lường từ phần cứng nhúng (ESP8266), bóc tách JSON và định tuyến ghi vào cơ sở dữ liệu chuỗi thời gian (SQLite)[cite: 9].
- **Điều khiển 2 chiều thời gian thực (Bi-directional Control):** Đóng/ngắt tải điện dân dụng (Đèn, Quạt) từ xa qua giao diện Web Dashboard với cơ chế phản hồi xác nhận trạng thái (Status Feedback)[cite: 9].
- **Bảo vệ an toàn tại biên (Edge Failsafe):** Tự động phát hiện rò rỉ khí gas/khói (ngưỡng >= 200 PPM) hoặc phát hiện ngập nước mưa, kích hoạt còi hú tại chỗ (Active Buzzer) ngay trên vi điều khiển mà không phụ thuộc vào kết nối mạng[cite: 9].

---

## 🚀 2. Tính năng chính[cite: 9]

- [x] **Thu thập dữ liệu môi trường:** Đọc và chuẩn hóa dữ liệu từ cảm biến khí gas MQ-2, cảm biến lượng mưa MH-RD và thông số vi khí hậu[cite: 9].
- [x] **API điều hướng lên Server/Cloud:** Tự động đóng gói JSON từ vi điều khiển và điều hướng gói tin qua HTTP REST API[cite: 9].
- [x] **Điều khiển thiết bị 2 chiều:** Bật/tắt rơ-le 2 kênh (Đèn, Quạt) qua Web Dashboard với cơ chế cập nhật trạng thái tức thì.
- [x] **Bảo vệ an toàn tại biên (Edge Computing):** Tự động hú còi báo động khi nồng độ gas >= 200 PPM hoặc phát hiện trời mưa.
- [x] **Lưu trữ dữ liệu thời gian thực:** Cơ sở dữ liệu tự động đồng bộ mốc thời gian chuẩn GMT+7 (Việt Nam).
- [x] **Trực quan hóa chỉ số:** Hiển thị trực quan dữ liệu chuỗi thời gian (Time-series data) dạng biểu đồ đường qua Chart.js.

---

## 🏗️ 3. Kiến trúc hệ thống

```text
[ Cảm biến & Thiết bị ]
 (MQ-2 Gas, MH-RD Rain, 2-Ch Relay, Buzzer)
              │
              ▼ (Analog A0 / GPIO)
      [ ESP8266 Edge Node ]
              │
              ▼ (HTTP REST API: POST /telemetry)
┌─────────────────────────────────────────────────────────────┐
│               HỆ THỐNG API GATEWAY ĐIỀU HƯỚNG               │
│  - Tiếp nhận & Bóc tách gói tin (Payload Ingestion)         │
│  - Bộ phân luồng & Điều hướng dữ liệu (Data Dispatcher)     │
└─────────────┬─────────────────────────────┬─────────────────┘
              │                             │
              ▼                             ▼
     [ Database SQLite3 ]          [ Web Dashboard (UI) ]
 (Lưu trữ telemetry chuỗi t/g)    (Giám sát & Điều khiển 2 chiều)

```
## 🛠️ 4. Ngăn xếp công nghệ (Tech Stack)

| Phân hệ | Công nghệ / Linh kiện sử dụng | Vai trò kỹ thuật |
| :--- | :--- | :--- |
| **Phần cứng** | NodeMCU ESP8266, MQ-2, MH-RD (LM393), Relay 2 kênh, Active Buzzer | Thu thập dữ liệu cảm biến và đóng ngắt tải điện |
| **Firmware** | C/C++ (VS Code + PlatformIO IDE) | Xử lý logic nhúng, kiểm soát ngưỡng biên & gọi REST API |
| **Thư viện nhúng** | ESP8266WiFi, ESP8266HTTPClient, ArduinoJson | Đóng gói JSON và gửi nhận dữ liệu mạng |
| **API / Backend** | Node.js (Express.js), CORS, Path | Tiếp nhận request từ ESP8266, điều hướng và lưu trữ dữ liệu |
| **Cơ sở dữ liệu** | SQLite3 (data.db) | Lưu trữ dữ liệu cảm biến chuỗi thời gian và trạng thái Relay |
| **Giao diện (UI)** | Web Dashboard (HTML5, Tailwind CSS, Chart.js, Lucide Icons) | Trực quan hóa dữ liệu và điều khiển thiết bị |


## 🔌 5. Sơ đồ nối dây phần cứng (Pinout Reference)

| Linh kiện / Module | Chân Module | Chân NodeMCU ESP8266 | Chế độ / Ghi chú |
| :--- | :--- | :--- | :--- |
| **MQ-2 Gas Sensor** | A0 (Analog) | A0 | Đọc mức điện áp analog nồng độ khí gas |
| **MH-RD Rain Sensor** | D0 (Digital) | D6 | Kéo trở phát hiện nước mưa (Active LOW) |
| **Relay Kênh 1 (Đèn)** | IN1 | D1 | Kích mức thấp (Active LOW) |
| **Relay Kênh 2 (Quạt)** | IN2 | D2 | Kích mức thấp (Active LOW) |
| **Active Buzzer** | Chân tín hiệu | D5 | Phát còi hú cảnh báo rò rỉ gas / mưa |
| **Nguồn hệ thống** | 5V / 3.3V / GND | VIN / 3V3 / GND | Nguồn cấp cho vi điều khiển và các module |

## 📡 6. Thiết kế API Điều hướng dữ liệu

### 6.1. Cấu trúc Payload JSON từ phần cứng gửi lên Gateway

```json
{
  "device_id": "ESP8266_SMARTHOME",
  "temperature": 28.5,
  "humidity": 65.2,
  "gas_level": 125,
  "rain": 0
}
```
### 6.2. Danh sách Endpoints API điều hướng

| Phương thức | Endpoint | Chức năng điều hướng |
| :--- | :--- | :--- |
| `POST` | `/api/v1/devices/telemetry` | Tiếp nhận dữ liệu định kỳ, bóc tách và ghi vào SQLite Database |
| `GET` | `/api/v1/devices/telemetry/recent` | Truy vấn 20 bản ghi mới nhất phục vụ biểu đồ Dashboard |
| `GET` | `/api/v1/devices/relays` | Truy vấn trạng thái hoạt động hiện tại của các rơ-le |
| `POST` | `/api/v1/devices/relays` | Nhận lệnh điều khiển bật/tắt rơ-le từ Dashboard và cập nhật DB |

## 📁 7. Cấu trúc thư mục dự án

```text
Smart-Home-IoT/
├── client/                     # Giao diện người dùng Web Dashboard
│   ├── index.html              # Trang giao diện chính (Tailwind CSS)
│   ├── css/style.css           # Định kiểu và hiệu ứng chuyển động
│   └── js/app.js               # Kết nối API, cập nhật biểu đồ thời gian thực
├── server/                     # API Gateway Backend (Node.js)
│   ├── server.js               # Express API & SQLite Data Gateway
│   ├── package.json
│   └── data.db                 # File cơ sở dữ liệu SQLite (tự động tạo)
├── src/                        # Mã nguồn vi điều khiển ESP8266
│   └── main.cpp                # Đọc cảm biến, gọi API & xử lý Failsafe tại biên
├── include/
│   └── config.h                # Cấu hình chân kết nối và thông số mạng
├── platformio.ini              # Cấu hình biên dịch môi trường PlatformIO
└── README.md                   # Tài liệu thuyết minh dự án
```

## ⚙️ 8. Hướng dẫn cài đặt & Triển khai

### Bước 1: Khởi chạy API Gateway Server

Yêu cầu máy tính/máy chủ đã cài đặt **Node.js**:

```bash
cd server
npm install
node server.js
```

*(Server sẽ khởi chạy tại địa chỉ: `http://localhost:5000`)*.

### Bước 2: Nạp mã nguồn cho ESP8266

1. Mở dự án bằng **VS Code** (đã cài đặt tiện ích mở rộng **PlatformIO IDE**).
2. Điều chỉnh thông số Wi-Fi và địa chỉ IP Gateway trong file `src/main.cpp`:

   ```cpp
   const char* WIFI_SSID = "TEN_WIFI_CUA_BAN";
   const char* WIFI_PASSWORD = "MAT_KHAU_WIFI";
   const char* SERVER_BASE_URL = "http://<IP_MAY_TINH>:5000/api/v1/devices";
   ```

3. Cắm cáp kết nối bo mạch **NodeMCU ESP8266** với máy tính và nhấn nút **Upload** trên thanh trạng thái PlatformIO.
### Bước 3: Mở giao diện điều khiển (Dashboard)

1. Mở trình duyệt và truy cập: `http://localhost:5000`.
2. Hoặc mở trực tiếp file `client/index.html` trên trình duyệt.



