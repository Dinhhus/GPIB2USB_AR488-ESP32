# Ghi chú bố trí chân — ESP32-WROOM-32 DevKit (38 chân) cho GPIB2USB

Tài liệu này tóm tắt lại các ràng buộc và phương án bố trí chân đã thống nhất
trong quá trình trao đổi thiết kế dự án GPIB2USB_AR488-ESP32, dùng board
DevKit ESP32-WROOM-32 (38 chân, loại phổ thông hay gọi "ESP32 DevKit V1").

## 1. Các nhóm chân cần tránh

| Nhóm | Chân | Lý do |
|---|---|---|
| Flash nội bộ | GPIO6–GPIO11 | Nối trực tiếp flash SPI trong module, không đưa ra ngoài, không dùng được |
| Input-only | GPIO34, GPIO35, GPIO36 (VP), GPIO39 (VN) | Không có tầng driver ra, không cấu hình open-drain output được |
| UART0 (nối PC qua chip USB-UART trên board) | GPIO1 (TX0), GPIO3 (RX0) | Nên giữ lại để giao tiếp serial với máy tính |
| Strapping rủi ro cao | GPIO0, GPIO2, GPIO12 | Ảnh hưởng chế độ boot / điện áp flash; dễ gây lỗi nạp firmware hoặc treo boot nếu bị kéo sai mức bởi pull-up của bus GPIB |
| Strapping rủi ro thấp (dùng được) | GPIO5, GPIO15 | Trạng thái mặc định lúc boot (mức cao) trùng với trạng thái nghỉ tự nhiên của bus GPIB (được kéo lên cao) |

## 2. Tập chân "an toàn" đề xuất (17 chân, cần 16 cho GPIB)

```
GPIO4, GPIO5, GPIO13, GPIO14, GPIO15, GPIO16, GPIO17, GPIO18, GPIO19,
GPIO21, GPIO22, GPIO23, GPIO25, GPIO26, GPIO27, GPIO32, GPIO33
```

Dư đúng 1 chân so với 16 đường cần thiết (8 DIO + 3 bắt tay DAV/NRFD/NDAC +
5 quản lý ATN/IFC/REN/SRQ/EOI) — có thể dùng chân dư làm LED báo trạng thái
hoặc bắt ngắt SRQ.

Đây là phương án **không dùng IC transceiver GPIB rời** (SN75160/75161),
GPIO của ESP32 lái thẳng vào bus, dựa vào chế độ open-drain phần cứng của
ESP32 (xem phần 3 bên dưới về cách firmware AR488-ESP32 thực sự làm việc này).

## 3. Đối chiếu với cấu hình mặc định của firmware AR488-ESP32

Khi tải firmware `ar488-esp32` về, `platformio.ini` có sẵn nhiều environment
cho ESP32. Điểm quan trọng cần lưu ý trước khi build:

- Environment mặc định (`esp32dev`, chọn khi build không chỉ định env) và
  cả `ttgo-t8-161`, `lolin32-161` đều bật cờ `SN7516X`, tức là **giả định
  bạn vẫn dùng IC transceiver GPIB thật** (SN75160/75161, hoặc SN75162 theo
  ghi chú `SN7516X_SC=0 # for 75162` trong file). Hai chân thêm `TE`
  (Talk Enable) và `DC` (Direction Control) dùng để điều khiển IC này,
  không phải đường tín hiệu bus.
- Bảng chân mặc định của `esp32dev`:

  ```
  TE=15  DC=2
  DIO1=33 DIO2=32 DIO3=26 DIO4=25 DIO5=14 DIO6=27 DIO7=13 DIO8=12
  REN=23  IFC=22  NDAC=21 NRFD=19 DAV=18  EOI=17  ATN=4   SRQ=16
  ```

  Chú ý: cấu hình gốc này **có dùng GPIO2 và GPIO12** — hai chân mình xếp
  vào nhóm "rủi ro cao" ở trên. Tác giả firmware vẫn dùng được trong thực
  tế (nhiều người đã build thành công), nhưng nếu gặp lỗi nạp firmware/khởi
  động chập chờn, nên thử đổi hai chân này sang chân khác qua build flag.
- Khi **không** định nghĩa `SN7516X` (xem ví dụ STM32 `f303k8` — không dùng
  transceiver — trong `platformio.ini`), firmware chuyển sang kiểu lái trực
  tiếp: đọc `AR488_Layouts.cpp` thấy khi "nhả" một đường bus, code gọi
  `pinMode(pin, INPUT_PULLUP)` (dùng pull-up nội yếu của MCU, không cần
  điện trở ngoài); khi "kéo xuống" thì gọi `pinMode(pin, OUTPUT)` rồi ghi
  mức thấp. Đây chính là kiểu "open-drain giả lập bằng phần mềm" đã nói ở
  phần trò chuyện trước — không phải open-drain phần cứng thật của STM32,
  nhưng hiệu quả tương đương cho một bus tải nhẹ, cáp ngắn, 1 thiết bị.
- **Việc cần làm nếu muốn bỏ hẳn IC transceiver rời** (theo hướng tối giản
  linh kiện đã bàn): thêm một environment mới trong `platformio.ini`, không
  đặt cờ `SN7516X`, và map `DIOx`/`REN`/`IFC`/`NDAC`/`NRFD`/`DAV`/`EOI`/`ATN`/`SRQ`
  vào đúng 16 chân trong tập "an toàn" ở mục 2. Nên thêm điện trở pull-up
  ngoài (~3.3kΩ lên 3.3V) trên mỗi đường thay vì chỉ dựa vào pull-up nội
  yếu, để tăng tốc độ sườn lên và độ tin cậy khi cáp dài hơn vài chục cm.

## 4. Việc còn mở (chưa quyết định)

- Có build theo đúng mẫu `esp32dev` sẵn có (dùng IC SN75160/75161/75162
  thật, bám sát chuẩn điện GPIB) hay tự chế bản không-transceiver theo
  mục 2/3 (gọn hơn, ít linh kiện hơn, nhưng chưa có ai xác nhận độ ổn định
  với thiết bị GPIB thật)?
- Nếu build bản không-transceiver: cần review lại phần `gpib.cpp`/
  `controller.cpp` để chắc chắn không có chỗ nào giả định sự tồn tại của
  chân `SN7516X_TE`/`SN7516X_DC`.
