# Smart-Home-IoT-System
# DALN
# HỆ THỐNG NHÀ THÔNG MINH IOT GIÁM SÁT MÔI TRƯỜNG, ĐIỀU KHIỂN THỜI GIAN THỰC VÀ TỰ ĐỘNG HÓA CẢNH BÁO AN TOÀN

> **Đồ án môn học / Đồ án liên ngành**  
> **Sinh viên thực hiện:** Phạm Khắc Hùng  
> **Nền tảng triển khai:** Vi điều khiển ESP32, Mosquitto MQTT Broker, Docker & Web/Mobile Dashboard  

---

## 1. Giới thiệu bài toán & Mục tiêu đề tài

### 1.1. Bối cảnh và Thách thức
Trong xu hướng xây dựng không gian sống thông minh và an toàn, các hệ thống điện dân dụng truyền thống tồn tại nhiều hạn chế:
- **Thiếu cảnh báo sớm về an toàn cháy nổ:** Nguy cơ rò rỉ khí gas sinh hoạt (LPG/Propane) hoặc chập cháy nổ thường chỉ được nhận biết khi thiệt hại đã xảy ra, thiếu cơ chế cảnh báo từ xa tức thời và tự động ngắt tải cục bộ.
- **Vận hành phân tán, tiêu tốn năng lượng:** Thiết bị chiếu sáng, làm mát phụ thuộc hoàn toàn vào thao tác bật/tắt thủ công tại chỗ, không có khả năng tự động hóa theo biến thiên vi khí hậu môi trường (nhiệt độ, độ ẩm).
- **Hạn chế của các nền tảng đám mây đóng (Closed Cloud):** Các thiết bị Smart Home thương mại giá rẻ thường phụ thuộc hoàn toàn vào server nước ngoài (Tuya, eWeLink), gây trễ phản hồi, nguy cơ mất quyền riêng tư dữ liệu sinh hoạt và tê liệt hoạt động khi mất kết nối Internet quốc tế.

### 1.2. Mục tiêu đề tài
Nghiên cứu, thiết kế và chế tạo một hệ thống **Nhà thông minh IoT (Internet of Things)** toàn diện từ biên phần cứng đến ứng dụng người dùng cuối:
- **Nguyên lý Edge-First & Privacy:** Dữ liệu và logic điều khiển được xử lý trực tiếp tại mạng nội bộ gia đình (Local LAN), bảo mật thông tin và không phụ thuộc dịch vụ đám mây bên thứ ba.
- **Truyền thông hai chiều thời gian thực (Full-duplex Realtime):** Ứng dụng giao thức truyền thông nhẹ **MQTT** kết nối hai chiều với độ trễ phản hồi cực thấp ($< 50\text{ ms}$).
- **Cơ chế bảo vệ biên tự chủ (Hardware Failsafe):** Tự động phát hiện rò rỉ khí gas bằng thuật toán lọc nhiễu, kích hoạt còi báo động tại chỗ và ngắt rơ-le phụ tải ngay ở tầng vi điều khiển mà không cần phụ thuộc vào trạng thái kết nối mạng hay lệnh từ server.
- **Giao diện quản lý tập trung:** Xây dựng Dashboard điều khiển trực quan, trực quan hóa dữ liệu đo môi trường dạng biểu đồ chuỗi thời gian (Time-series) và hỗ trợ quản trị trạng thái đa thiết bị.

---

## 2. Kiến trúc phần cứng hệ thống

Hệ thống sử dụng các linh kiện phần cứng công nghiệp/thực nghiệm có độ tin cậy cao, kết nối dạng module hóa:

```text
       ┌───────────────────────────────┐
       │     Nguồn DC 5V/3A (Buck)     │
       └──────────────┬────────────────┘
                      │ Nguồn nuôi
                      ▼
 ┌─────────────────────────────────────────────────────────────┐
 │                      ESP32 DevKit V1                        │
 │  (Dual-Core Xtensa LX6 @ 240MHz, Wi-Fi 802.11 b/g/n, BLE)   │
 └───▲─────────────▲──────────────┬──────────────┬───────────┬─┘
     │ GPIO 4      │ GPIO 34 (ADC)│ GPIO 18      │ GPIO 19   │ GPIO 21
     │             │              │              │           │
┌────┴────┐   ┌────┴────┐    ┌────▼────┐    ┌────▼────┐ ┌────▼────┐
│  DHT22  │   │  MQ-2   │    │ Relay 1 │    │ Relay 2 │ │ Active  │
│(Nhiệt/Ẩm│   │ (Khí gas│    │ (Đèn)   │    │ (Quạt)  │ │ Buzzer  │
└─────────┘   └─────────┘    └─────────┘    └─────────┘ └─────────┘
