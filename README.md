# GPIB2USB_AR488-ESP32

Dự án tự làm bộ chuyển đổi GPIB (IEEE-488) sang USB để kết nối multimeter
(và các thiết bị đo khác có cổng GPIB) với máy tính, dựa trên firmware mã
nguồn mở AR488 (bản fork cho ESP32).

## Mục tiêu

- Thay thế bộ chuyển đổi thương mại (Prologix GPIB-USB, NI GPIB-USB-HS...)
  bằng mạch tự làm, chi phí thấp hơn nhiều.
- Tương thích phần mềm với các công cụ đã hỗ trợ sẵn giao thức Prologix:
  PyVISA, LabVIEW, MATLAB Instrument Control Toolbox...
- Đọc số liệu từ multimeter GPIB (SCPI) qua cổng USB-serial ảo trên máy tính.

## Phần cứng đã chọn

- Vi điều khiển: **ESP32-WROOM-32 DevKit (38 chân)** — đã có sẵn.
- Kết nối vật lý: đầu nối GPIB 24 chân (tận dụng cáp GPIB cũ, cắt lấy đầu).
- Chưa chốt: dùng IC transceiver GPIB thật (SN75160/75161/75162) hay bản
  gọn lái thẳng bằng GPIO open-drain của ESP32 — xem phân tích trong
  `docs/pin_mapping_esp32_wroom32.md` mục 4 ("Việc còn mở").

## Cấu trúc thư mục trên Drive

```
GPIB2USB_AR488-ESP32/
├── README.md                              — file này
├── firmware/
│   └── ar488-esp32/                       — (xem docs/SOURCES.md để clone)
└── docs/
    ├── pin_mapping_esp32_wroom32.md       — ghi chú bố trí chân đã thống nhất
    └── SOURCES.md                          — link + lệnh tải firmware, tài liệu
```

## Tóm tắt các quyết định thiết kế (từ quá trình trao đổi)

1. **Vì sao không dùng PLC**: mức điện áp I/O của PLC (24V) không tương
   thích với TTL 5V của GPIB — rủi ro cháy cổng GPIB của multimeter; chu kỳ
   quét PLC (ms) cũng chậm hơn nhiều so với timing bắt tay GPIB cần. Nếu
   muốn tích hợp vào hệ PLC sẵn có, nên để board ESP32 làm cầu GPIB thật,
   xuất dữ liệu ra PLC qua Modbus thay vì PLC nói GPIB trực tiếp.
2. **Open-collector/open-drain**: có thể làm bằng transistor rời hoặc IC
   đệm hex open-collector (7406/7407/74LS06/74LS07) thay cho 75160/161 nếu
   khó mua; hoặc dùng chế độ open-drain phần cứng có sẵn trên hầu hết
   STM32 và mọi ESP32 (không cần transistor rời).
3. **STM32F411 vs ESP32-WROOM-32**: F411 (Black Pill) có nhiều chân hơn,
   có USB device thật, nhiều chân 5V-tolerant, nhưng AR488 **chưa có bản
   port sẵn cho STM32** — phải tự viết firmware. ESP32 đã có bản fork
   AR488-ESP32 sẵn dùng được ngay, đổi lại phải né kỹ các chân
   strapping/flash/input-only (xem mục 1 trong `docs/pin_mapping_esp32_wroom32.md`).
   → Đã chốt dùng ESP32-WROOM-32 DevKit sẵn có.

## Bước tiếp theo

- [ ] Quyết định có dùng IC SN7516x hay bản không-transceiver.
- [ ] Nếu dùng bản không-transceiver: thêm environment mới trong
      `platformio.ini`, map đúng 16 chân trong tập an toàn, review
      `src/gpib.cpp` và `src/controller.cpp` để đảm bảo không còn phụ
      thuộc `SN7516X_TE`/`SN7516X_DC`.
- [ ] Kiếm đầu nối/cáp GPIB 24 chân.
- [ ] Build và nạp thử firmware bằng PlatformIO, kiểm tra với multimeter
      thật (đặt địa chỉ GPIB trên máy đo, thử lệnh SCPI qua `++addr`,
      `++read`).

## Nguồn tham khảo

- AR488 (bản gốc, AVR): https://github.com/Twilight-Logic/AR488
- AR488-ESP32 (fork dùng cho dự án này): https://github.com/douardda/AR488-ESP32
- Tài liệu build AR488: https://sdfa3.org/david/ar488/build.html
