# Find The Cow — HaND

> **Color-block logic puzzle (chủ đề bò)** | HTML5 single-file | Level vô hạn, câu đố sinh ngẫu nhiên

## Thông tin game

| Trường | Giá trị |
|---|---|
| **Package** | `com.falcon.findthecow` |
| **Engine / Version** | HTML5 single-file · 1.0.0 |
| **Category** | PUZZLE |
| **Loại** | Game **có level** — vô hạn, thắng/thua theo màn |

## Gameplay

- Lưới **N×N** chia thành **N vùng màu**. Đặt **N con bò** sao cho **mỗi hàng, mỗi
  cột và mỗi vùng màu có ĐÚNG 1 con bò**; hai con bò **không được kề nhau** (kể cả
  chéo).
- Mỗi màn có **một lời giải duy nhất** và **giải được bằng suy luận** từng bước
  (không đoán mò). Người chơi phải đặt bò **đúng y hệt** lời giải mới thắng.
- **3 mạng**: đặt bò vào ô **không** thuộc lời giải → nháy đỏ, **−1 mạng**, không cho
  đặt. Hết mạng → thua màn.
- **Cỡ bàn tăng dần**: `LEVEL_RAMP = [5,5,5,6,6,7,7,8,8,9,9,10]` — 3 màn đầu 5×5 dễ,
  rồi lớn dần tới **10×10** (tối đa 10 màu/màn) và giữ nguyên.

### Điều khiển

- **Nhấp 1 lần / chạm** = đánh dấu **X** (nhấp lại để bỏ X).
- **Giữ & kéo** = đánh (hoặc tẩy) **X nhiều ô** liền.
- **Nhấp/chạm ĐÔI** = **đặt bò** — đặt rồi **KHÓA**, không gỡ được.

### Sinh câu đố — duy nhất & suy luận được

Nhanh ở mọi cỡ (median < 0.1ms tới 10×10), khác kiểu sinh-rồi-thử chậm với N lớn.

- **`makeSolution(N)`** — backtracking dựng một nghiệm hợp lệ (1 bò/hàng/cột, không
  kề nhau).
- **`growRegions(N, sol)`** — vẽ vùng theo **thứ tự mở** con bò: mỗi vùng chỉ "ăn" ô
  đã bị loại (chết) TRƯỚC khi con bò của vùng tới lượt → con bò luôn bị **ép buộc**
  (vùng chỉ còn 1 ứng viên). Nuôi **cân bằng** (vùng nhỏ nhất lớn thêm) rồi vùng của
  con bò mở cuối dọn nốt → lấp kín & liền mạch.
- **`solvableByLogic(N, region)`** — xác nhận câu đố giải được thuần suy luận: lặp
  đặt bò ở hàng/cột/vùng nào chỉ còn đúng 1 ứng viên, loại ứng viên liên quan, tới
  khi đủ N bò. Nhờ vậy câu đố **vừa duy nhất vừa không phải đoán**.

### Gợi ý (Hint)

- Bắt đầu **3 lượt** từ màn 1 (lưu qua `save_data`).
- Mỗi lượt đặt giúp **1 con bò đúng** (gỡ các bò đặt sai đang xung đột) — **không mất
  mạng**.
- **Còn lượt** → dùng & trừ 1; **hết lượt** → badge đổi thành `▶`, bấm bắn event
  `ads` (app hiện quảng cáo) rồi tặng 1 gợi ý dùng luôn.

### Âm thanh (Web Audio — tổng hợp, không cần file)

Module `Sound` tự sinh nốt bằng `AudioContext` (khớp phong cách game anh em
**skydom**: master gain + limiter). Hiệu ứng cho từng sự kiện:

| Sự kiện | Âm |
|---|---|
| Đánh / tẩy X | "tick" gọn (1 tiếng/lần chạm, không kêu dồn khi kéo) |
| Đặt bò đúng | "boop-beep" trưởng vui + "moo" ấm ở trầm |
| Đặt sai (−1 mạng) | buzz đi xuống |
| Gợi ý | lấp lánh đi lên |
| Thắng màn | kèn chúc mừng (hợp âm rải) + "moo" |
| Hết mạng | chuỗi nốt đi xuống buồn |

- **Nút bật/tắt tiếng** trong thanh công cụ (loa ↔ loa gạch chéo); lựa chọn **lưu qua
  `save_data`** (`{ muted }`).
- **Mở khoá AudioContext** ở lần chạm đầu (autoplay mobile); `onAppPause` tạm dừng,
  `onAppResume` phát tiếp.

## Tuân thủ quy ước chung

- ✅ **game-common.md** — `sendMessage` 5 trường, level 1-indexed; `game_result`
  win/lose + `showModal:true`; `quit`; nút **Restart** nội bộ không bắn event; `ads`
  khi hết lượt gợi ý; `save_data` chỉ **data nhẹ** (`{ hints, muted }`); đọc đủ
  `statusBarHeight`/`currentLevel`/`data`/`language` đúng thứ tự ưu tiên;
  `waitForNativeInjection`; callback native `onAppPause`/`onNextLevel`/
  `onRetryLevel` (+ `onAppResume` cho audio); font Google Sans Flex qua `--ui-font`
  (ngoài Latinh → `system-ui`); reset CSS + tắt tap-highlight.
- ✅ **popup-common.md** — **không tự vẽ modal kết quả** (app lo popup nhờ
  `showModal:true`); trong game chỉ có **toast** báo thoáng (phạm luật / giải xong /
  hết mạng).
- ✅ **zip-common.md** — single-file `index.html`, `game.json` đủ field (5 tags,
  packageName reverse-DNS), 3 cover đúng kích thước 1920×1080 / 800×1200 / 800×800.
- ✅ **i18n** — inline trong `index.html`, fallback `en`. Đủ chuỗi cho **en/vi**;
  các ngôn ngữ khác (zh-Hans, zh-Hant, ja, ko, es, pt-BR, fr, de, ru, tr, id) dịch
  các chuỗi chính (mục tiêu, phạm luật, thắng, thua) — phần còn lại rơi về `en`.
- ℹ️ **Level vô hạn → không có màn cuối, không bắn `victory`**; thắng luôn gửi
  `game_result win` (`showModal:true`).
- ℹ️ Mở bằng browser thường (không có `ReactNativeWebView`): tự sang màn kế sau khi
  thắng (~750ms) / tự chơi lại sau khi thua (~900ms) để playtest — trong app thật
  không chạy nhánh này.

## Lưu trữ

- **`save_data` (native)** — chỉ data nhẹ: `{ hints, muted }`. Ghi khi dùng gợi ý,
  bật/tắt tiếng, thắng màn.
- **Không dùng localStorage** — câu đố sinh mới mỗi màn; Restart/Retry dùng lại đúng
  đề qua `puzzleCache` (giữ trong bộ nhớ, không cần persist ván dở).

## 📋 Backlog

- [ ] Nhạc nền (BGM) nhẹ, bật/tắt chung nút Sound.
- [ ] Hiệu ứng ăn mừng (confetti) khi thắng màn.
- [ ] Dịch đủ 23 ngôn ngữ (hiện đủ en/vi, còn lại một phần).
- [ ] Chế độ "no-mistake" / tính sao theo số mạng còn lại → meta progression qua
  `save_data`.
