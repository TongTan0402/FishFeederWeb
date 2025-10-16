# 🌐 Hướng Dẫn Sử Dụng Web Interface MQTT

## 📋 Tổng Quan

Hệ thống bể cá IoT bây giờ được tách thành 2 phần:

1. **ESP32**: Chỉ kết nối WiFi và MQTT, điều khiển phần cứng
2. **Web Interface**: File HTML độc lập, kết nối MQTT qua WebSocket

## 🎯 Ưu Điểm

✅ **Không cần ESP32 làm web server** - Tiết kiệm tài nguyên  
✅ **Truy cập từ xa dễ dàng** - Chỉ cần broker MQTT public  
✅ **Nhiều người dùng cùng lúc** - Không giới hạn kết nối  
✅ **Không cần cùng mạng WiFi** - Truy cập từ bất kỳ đâu  
✅ **Dễ tùy chỉnh** - Chỉnh sửa HTML không cần upload lại ESP32

## 🚀 Cách Sử Dụng

### Bước 1: Chuẩn Bị MQTT Broker

Bạn có 2 lựa chọn:

#### Option A: Sử dụng Broker Công Khai (Đơn Giản)

**HiveMQ Public Broker:**
- Broker: `broker.hivemq.com`
- WebSocket Port: `8000`
- Không cần username/password

**Mosquitto Test Server:**
- Broker: `test.mosquitto.org`
- WebSocket Port: `8080`
- Không cần username/password

⚠️ **Lưu ý:** Broker công khai không bảo mật, ai cũng có thể xem/điều khiển!

#### Option B: Cài Đặt Broker Riêng (Khuyến Nghị)

**Cài Mosquitto trên Windows:**

1. Tải Mosquitto: https://mosquitto.org/download/
2. Cài đặt
3. Tạo file cấu hình `mosquitto.conf`:

```conf
# Cho phép kết nối không xác thực (test)
allow_anonymous true

# MQTT port thông thường
listener 1883

# WebSocket listener
listener 8000
protocol websockets

# Nếu muốn bảo mật, thêm:
# password_file C:\mosquitto\passwd.txt
```

4. Chạy Mosquitto:
```cmd
mosquitto -c mosquitto.conf -v
```

5. Port forwarding router (nếu muốn truy cập từ xa):
   - Port 1883 (MQTT)
   - Port 8000 (WebSocket)

### Bước 2: Cấu Hình ESP32

Mở file `src/main.cpp` và sửa:

```cpp
const char* ssid = "TenWiFi";
const char* password = "MatKhauWiFi";
const char* mqtt_server = "broker.hivemq.com";  // Hoặc IP broker của bạn
```

Upload code lên ESP32:
```bash
pio run -t upload
pio device monitor
```

Kiểm tra Serial Monitor thấy:
```
WiFi đã kết nối!
Đang kết nối MQTT...đã kết nối!
Hệ thống sẵn sàng!
```

### Bước 3: Mở Web Interface

1. Mở file `web/index.html` bằng trình duyệt (Chrome, Firefox, Edge)
2. Điền thông tin MQTT:
   - **MQTT Broker**: `broker.hivemq.com` (hoặc IP broker của bạn)
   - **WebSocket Port**: `8000`
   - **Username/Password**: Để trống nếu không cần

3. Click **"Kết Nối MQTT"**

4. Thấy thông báo: "🟢 Đã kết nối MQTT - Sẵn sàng điều khiển"

### Bước 4: Sử Dụng

Bây giờ bạn có thể:
- ⚡ Bật/tắt 4 outlet
- 🍽️ Cho cá ăn
- ⏰ Cài đặt lịch trình tự động
- 📊 Xem trạng thái real-time

## 🌍 Truy Cập Từ Xa

### Cách 1: Dùng Broker Public
- Mở `web/index.html` từ bất kỳ đâu
- Kết nối đến `broker.hivemq.com:8000`
- Điều khiển bể cá ngay lập tức!

### Cách 2: Dùng Broker Riêng + DDNS

1. **Cài đặt DDNS** (NoIP, DuckDNS, etc.)
2. **Port Forward** router:
   - Port 8000 → IP máy chạy Mosquitto
3. **Truy cập:**
   - Broker: `yourname.ddns.net`
   - Port: `8000`

### Cách 3: Host Web Trên Server

Upload file `web/index.html` lên:
- GitHub Pages
- Netlify
- Vercel
- Hoặc server riêng

Rồi truy cập qua URL:
- `https://yourname.github.io/fishfeeder`

## 📱 Truy Cập Bằng Điện Thoại

### Android/iOS:

1. Mở trình duyệt (Chrome/Safari)
2. Vào URL file HTML (nếu đã host)
3. Hoặc copy file `index.html` vào điện thoại, mở bằng trình duyệt
4. Kết nối MQTT như bình thường

### Tạo Web App (PWA):

Thêm vào `<head>` của `index.html`:

```html
<link rel="manifest" href="manifest.json">
<meta name="theme-color" content="#667eea">
```

Tạo file `manifest.json`:

```json
{
  "name": "Bể Cá IoT",
  "short_name": "Fish Tank",
  "start_url": "index.html",
  "display": "standalone",
  "theme_color": "#667eea",
  "background_color": "#ffffff",
  "icons": [
    {
      "src": "icon-192.png",
      "sizes": "192x192",
      "type": "image/png"
    }
  ]
}
```

Bây giờ có thể "Add to Home Screen"!

## 🔒 Bảo Mật

### Bảo Vệ MQTT Broker

1. **Bật xác thực:**

```bash
# Tạo user
mosquitto_passwd -c passwd.txt admin
mosquitto_passwd passwd.txt user1
```

Sửa `mosquitto.conf`:
```conf
allow_anonymous false
password_file C:\mosquitto\passwd.txt
```

2. **Sử dụng SSL/TLS:**

```conf
listener 8883
protocol websockets
certfile C:\mosquitto\certs\server.crt
keyfile C:\mosquitto\certs\server.key
```

Trong web, đổi thành:
```javascript
const url = `wss://${broker}:8883/mqtt`;  // wss = WebSocket Secure
```

3. **Firewall Rules:**
- Chỉ cho phép IP tin cậy
- Hoặc dùng VPN

### Thay Đổi Topic Prefix

Trong ESP32 và Web, đổi:
```cpp
const char* TOPIC_PREFIX = "fishfeeder";
```
Thành:
```cpp
const char* TOPIC_PREFIX = "mysecrethash_12345";
```

## 🐛 Xử Lý Sự Cố

### ❌ Web không kết nối được MQTT

**Nguyên nhân:**
- Broker không hỗ trợ WebSocket
- Port sai
- Firewall chặn

**Giải pháp:**
1. Kiểm tra broker có port WebSocket:
   ```bash
   # Test bằng MQTT Explorer
   # Hoặc telnet
   telnet broker.hivemq.com 8000
   ```

2. Thử broker khác:
   - `broker.hivemq.com:8000`
   - `test.mosquitto.org:8080`

3. Mở F12 Console trong trình duyệt, xem lỗi

### ❌ ESP32 kết nối được nhưng Web không nhận data

**Giải pháp:**
1. Kiểm tra cùng broker:
   - ESP32: `broker.hivemq.com`
   - Web: `broker.hivemq.com`

2. Kiểm tra topic:
   - Subscribe: `fishfeeder/#`

3. Dùng MQTT Explorer để debug:
   - Xem ESP32 có publish không
   - Xem topic có đúng không

### ❌ CORS Error

Nếu host HTML trên server và gặp lỗi CORS:

**Giải pháp:**
1. Dùng HTTPS nếu broker dùng WSS
2. Hoặc host HTML và broker trên cùng domain
3. Hoặc cấu hình CORS trong broker

### ❌ WebSocket Connection Failed

**Giải pháp:**
1. Đảm bảo broker đang chạy:
   ```bash
   mosquitto -v
   ```

2. Kiểm tra port không bị chiếm:
   ```bash
   netstat -an | findstr :8000
   ```

3. Tắt antivirus/firewall tạm thời để test

## 📊 Giám Sát MQTT

### MQTT Explorer (Windows/Mac/Linux)

1. Download: http://mqtt-explorer.com/
2. Kết nối broker
3. Xem tất cả topics và messages real-time
4. Gửi message test

### MQTTX (Cross-platform)

1. Download: https://mqttx.app/
2. Modern UI, nhiều tính năng
3. Hỗ trợ nhiều broker cùng lúc

### Command Line

```bash
# Subscribe tất cả topics
mosquitto_sub -h broker.hivemq.com -t "fishfeeder/#" -v

# Publish message test
mosquitto_pub -h broker.hivemq.com -t "fishfeeder/outlet1" -m "ON"

# Với authentication
mosquitto_sub -h localhost -t "#" -u admin -P password -v
```

## 🎨 Tùy Chỉnh Web Interface

File `web/index.html` rất dễ chỉnh sửa:

### Thay Đổi Màu Sắc

```css
/* Gradient background */
background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);

/* Đổi thành */
background: linear-gradient(135deg, #4facfe 0%, #00f2fe 100%);
```

### Thêm Thiết Bị Mới

```html
<button class="outlet-btn off" id="outlet5" onclick="toggleOutlet(5)">
    🌊 Bơm Oxy: TẮT
</button>
```

### Thay Đổi Icon

Sử dụng emoji hoặc Font Awesome:
```html
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css">

<button>
    <i class="fas fa-lightbulb"></i> Đèn
</button>
```

### Thêm Ngôn Ngữ

Tạo file `lang.js`:
```javascript
const lang = {
  vi: {
    title: "Bể Cá IoT",
    connect: "Kết Nối"
  },
  en: {
    title: "IoT Fish Tank",
    connect: "Connect"
  }
};
```

## 📝 Tổng Kết

Bây giờ bạn có:
- ✅ ESP32 chỉ lo phần cứng và MQTT
- ✅ Web interface hiện đại, responsive
- ✅ Truy cập từ bất kỳ đâu có internet
- ✅ Nhiều người dùng cùng lúc
- ✅ Dễ tùy chỉnh và mở rộng

**Chúc bạn thành công! 🐠🌊**

## 🔗 Links Hữu Ích

- MQTT.js: https://github.com/mqttjs/MQTT.js
- HiveMQ: https://www.hivemq.com/
- Mosquitto: https://mosquitto.org/
- MQTT Explorer: http://mqtt-explorer.com/
