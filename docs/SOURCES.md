# Nguồn firmware và tài liệu

## Vì sao mã nguồn firmware không được chép nguyên văn lên đây

Firmware là mã nguồn nhiều file (gần 20 file .cpp/.h cho riêng bản ESP32).
Việc gõ/dán lại toàn bộ nội dung này qua chat vào Google Drive không đáng
tin cậy bằng việc tự `git clone` trực tiếp — dễ sai sót, thiếu file, và
không giữ được lịch sử commit. Vì vậy tài liệu này chỉ chứa đường link và
lệnh tải chính xác; hai bộ firmware đã được clone thành công trong phiên
làm việc (dùng để tham khảo/kiểm tra), còn bản đầy đủ, đáng tin cậy nhất để
build là clone trực tiếp về máy bạn theo hướng dẫn dưới đây.

## 1. Firmware chính — AR488-ESP32 (dùng cho dự án này)

```bash
git clone https://github.com/douardda/AR488-ESP32.git
cd AR488-ESP32
pio run -e esp32dev        # build thử với cấu hình mặc định (dùng SN75160/75161)
pio run -e esp32dev -t upload   # nạp vào board qua USB
```

- Yêu cầu: [PlatformIO](https://platformio.org/) (cài qua VSCode extension
  hoặc `pip install platformio --break-system-packages`).
- Cấu hình chân mặc định của environment `esp32dev` nằm trong
  `platformio.ini` (xem `pin_mapping_esp32_wroom32.md` để đối chiếu với
  bảng chân đã chọn cho board ESP32-WROOM-32 DevKit 38 chân).
- Tài liệu build/command reference: https://douardda.srht.site/

## 2. Firmware gốc — AR488 (Twilight-Logic, chỉ để tham khảo)

```bash
git clone https://github.com/Twilight-Logic/AR488.git
```

- Có sẵn `AR488-manual.pdf` trong repo — tài liệu tham chiếu đầy đủ nhất về
  bộ lệnh Prologix ++ mà cả hai bản firmware đều dùng chung cú pháp.
- Tài liệu online: https://sdfa3.org/david/ar488/

## 3. Tài liệu khác

- AR488-ESP32 GPIB Controller (tổng quan): https://douardda.srht.site/
- AR488 build guide gốc: https://sdfa3.org/david/ar488/build.html
- sigrok wiki về AR488: https://sigrok.org/wiki/AR488
