# Bảng đấu dây GPIB ↔ ESP32-WROOM-32 DevKit (38 chân)

Phương án: **lái thẳng bằng GPIO**, không dùng IC transceiver SN75160/75161.
Cấu hình tương ứng đã nằm sẵn trong `arduino/AR488/AR488_Layouts.h`.

## 1. Bảng ánh xạ 16 đường tín hiệu

| GPIB (đầu 24 chân) | Tín hiệu | GPIO ESP32 | Vị trí trên board |
|---|---|---|---|
| 1  | DIO1 | 23 | hàng phải |
| 2  | DIO2 | 22 | hàng phải |
| 3  | DIO3 | 21 | hàng phải |
| 4  | DIO4 | 19 | hàng phải |
| 13 | DIO5 | 18 | hàng phải |
| 14 | DIO6 | 5  | hàng phải |
| 15 | DIO7 | 17 | hàng phải |
| 16 | DIO8 | 16 | hàng phải |
| 5  | EOI  | 27 | hàng trái |
| 6  | DAV  | 26 | hàng trái |
| 7  | NRFD | 25 | hàng trái |
| 8  | NDAC | 33 | hàng trái |
| 9  | IFC  | 32 | hàng trái |
| 10 | SRQ  | 14 | hàng trái |
| 11 | ATN  | 4  | hàng phải (gần cuối) |
| 17 | REN  | 13 | hàng trái |

Bus dữ liệu gom hết về hàng phải, các đường điều khiển về hàng trái, chỉ
ATN phải lấy từ hàng phải vì hàng trái đã hết chân an toàn.

## 2. Các chân nối đất

| GPIB | Nối vào |
|---|---|
| 18, 19, 20, 21, 22, 23, 24 | GND của ESP32 (nối chung) |
| 12 (shield) | vỏ cáp / GND |

Chân 18–23 là dây mass xoắn đôi của DAV, NRFD, NDAC, IFC, SRQ, ATN — cáp
ngắn thì chập chung vào GND cũng chạy, nhưng nối đủ vẫn tốt hơn cho nhiễu.

## 3. Điện trở ngoài

Firmware chỉ dùng pull-up nội của ESP32 (~45kΩ) — quá yếu để kéo bus lên
nhanh. Cần thêm trên mỗi trong 16 đường:

- **Pull-up 3.3kΩ lên 3.3V**, đặt phía ESP32.
- **Điện trở nối tiếp 220Ω** giữa chân ESP32 và đầu nối GPIB (xem mục 4).

## 4. Rủi ro cần biết trước khi cắm vào máy đo thật

GPIO của ESP32 chịu tối đa khoảng 3.6V, trong khi máy đo GPIB dùng driver
TTL 5V: khi multimeter ở vai talker và lái các đường DIO/DAV/EOI lên mức
cao, điện áp thực tế thường quanh 3.4V — sát ngưỡng chịu đựng của ESP32,
không còn biên an toàn. Đây là lý do firmware gốc mặc định đi qua IC
SN75160/75161.

Hai biện pháp giảm rủi ro khi vẫn lái thẳng:

- Điện trở nối tiếp **220Ω** trên mỗi đường (giới hạn dòng chảy vào diode
  bảo vệ của ESP32 khi máy đo lái cao). Giữ giá trị nhỏ, đừng dùng 1kΩ —
  với pull-up 3kΩ sẵn có trong máy đo, mức thấp sẽ bị đẩy lên quá ngưỡng
  0.8V và bắt tay sẽ lỗi.
- Bật máy đo **sau** khi ESP32 đã cấp nguồn, và rút cáp GPIB khi ESP32
  không chạy — vì lái thẳng không có tri-state Hi-Z như IC transceiver,
  điện áp ký sinh từ bus có thể chạy ngược vào chip khi board mất nguồn.

Thêm nữa, chuẩn IEEE-488 yêu cầu driver hút được 48mA; GPIO ESP32 chỉ hút
được cỡ 20–40mA. Với một máy đo và cáp ngắn (dưới ~1m) thì đủ, nhưng nếu
mắc nhiều thiết bị hoặc cáp dài mà thấy bắt tay chập chờn thì nguyên nhân
nằm ở đây, và cách sửa là quay lại dùng SN75160/75161.

## 5. Chân còn trống

- **GPIO15** — chưa dùng, để dành cho LED báo trạng thái hoặc mở rộng.
- **GPIO2** — LED sẵn trên board (LED_BUILTIN), firmware tự bật ở `setup()`.
- **GPIO0, GPIO12** — cố tình bỏ trống (strapping, dễ gây lỗi boot/nạp).
- **GPIO1, GPIO3** — UART0, dành cho nạp firmware và cổng COM ảo với PC.
