# Fruit Drop — QuanKA

> **Onet / Pikachu connect CÓ TRỌNG LỰC** | HTML5 single-file | Level vô hạn sinh theo công thức

## Thông tin game

| Trường | Giá trị |
|---|---|
| **Package** | `com.falcon.fruitdrop` |
| **Engine / Version** | HTML5 single-file · 1.0.0 |
| **Category** | PUZZLE |
| **Loại** | Game **có level** — vô hạn, thắng/thua theo màn |
| **Kích thước** | `index.html` ~120 KB (đã gồm 20 KB sprite icon) |

## Gameplay

- Nối 2 tile **cùng loại quả** bằng đường ≤ 3 đoạn thẳng (**≤ 2 lần rẽ**) qua ô
  trống; đường được vòng ra ngoài biên (vành ngoài 1 ô luôn đi được).
- **KHÁC animal-connect: có TRỌNG LỰC.** Sau mỗi lần nối, mọi tile nằm trên ô
  trống trong cùng cột **rơi xuống** lấp chỗ — bàn tự sắp xếp lại sau *mỗi* nước.
- **Sạch bàn trước khi hết giờ = thắng màn** → sang màn kế; **hết giờ = THUA =
  endgame** (run kết thúc, chơi lại từ màn 1, giữ BEST).
- **Điểm CỘNG DỒN cả run** (lvl 1 → endgame). SCORE = điểm run hiện tại,
  BEST = điểm run cao nhất.
- Điểm bay lên ô SCORE là **ngôi sao giống hệt icon trên chính ô SCORE** (dùng
  chung path/màu), để thứ bay lên và chỗ nó đáp xuống đọc ra cùng một ý.
- **Cơ chế điểm đặc trưng — DROP COMBO**: nếu lần nối kế tiếp dùng ít nhất một
  tile **vừa rơi** ở lần trước thì combo tăng một bậc (×1.5 → ×2 → ×2.5, trần
  ×4) và hiện tem `DROP ×N`. Trọng lực vì thế là **động cơ tính điểm**, không
  phải hiệu ứng trang trí — nó thưởng đúng kỹ năng mà cơ chế này thêm vào:
  đọc bàn *sau khi* nó sập xuống.
- **Đồng hồ ẩn thưởng nối nhanh**: <1.1s → "Lightning!", <2.2s → "Fast!"
  (cộng vào trước khi nhân hệ số combo).
- **~5s không thao tác** → tự nháy một cặp nối được (auto-hint, vòng xanh dương).
  Hết nước đi → **tự xáo miễn phí** kèm toast.
- **Không có menu** — mở app vào thẳng gameplay. Lần đầu chơi tự vào **tutorial**.
- **Header chuẩn**: Back (trái) · SCORE ★ · BEST ♛ (2 pill bằng nhau, BEST vàng)
  · Volume (phải); dưới là **LEVEL | thanh giờ | số giây**.
- **SFX tổng hợp Web Audio** (không file ngoài, không nhạc nền), qua một bus
  `sfxOut`, mềm/tròn, năng lượng 250–3000 Hz. Tiếng rơi dùng **modal synthesis**
  (noise qua bandpass) và **phát 1 lần mỗi lần lắng**, không phải mỗi tile —
  không thì một cú sập dây chuyền nghe như súng máy.

### Tutorial — 1 bàn 6×4, không chuyển cảnh

4 bước, **mục tiêu từng bước được tìm trên bàn THẬT lúc chạy** (không hard-code
toạ độ), nên không bao giờ có bước yêu cầu thứ mà bàn không có:

1. **LINK** — nối cặp đang sáng (chọn cặp dọc có ≥2 tile phía trên để cú rơi thấy rõ).
2. **DROP** — sau cú rơi, game tự tìm **cặp do chính cú rơi tạo ra** và bắt dọn nó.
3. **BEND** — chạm cặp cùng loại **bị chặn**; game vẽ đường "quá vòng" **đúng 3
   lần rẽ** (đỏ nét đứt + chấm ở mỗi chỗ rẽ) cho thấy vì sao không nối được.
4. **CLEAR** — dọn nốt bàn, hiện **timer demo** + mũi tên chỉ lên nó (timer chỉ
   chạy sau thao tác đầu tiên, và **không bao giờ gây thua**).

`buildTutorialBoard()` sinh lại bàn tới khi **cả 4 bài học đều dạy được** và
greedy-solve (có trọng lực) còn **0 ô** — verify bằng máy, không phải bằng mắt.
Tutorial **không tính điểm** (score/best giữ 0), có Skip, `?reset=1` xem lại.

### Level sinh theo công thức — `levelConfig(idx)`

| Level | Bàn | Số loại quả | Giờ |
|---|---|---|---|
| 1 | 6×6 | 6 | 80s |
| 2 | 8×6 | 8 | 100s |
| 3 | 8×8 | 8 | 130s |
| 4 | 10×8 | 10 | 155s |
| 5 | 10×10 | 10 | 190s |
| 6+ | 12×10 | 12 | 215s → **giảm đều** → 120s (L16+) |

- Bàn to dần tới L6 rồi **giữ nguyên**, sau đó **timer siết đều mỗi màn** — giữ
  một biến đứng yên và cho biến kia đi một chiều là cách làm đường cong khó
  *đọc được* (bài học từ animal-connect).
- **Chỉ 12 loại quả trên một bàn, nhưng cửa sổ 12-loại XOAY theo level** nên cả
  **14 icon** đều xuất hiện trong một run — có đa dạng mà không trả giá deadlock.
- Vô hạn → **không có màn cuối, không bắn `victory`** (deviation có chủ ý so với
  game-common §1.5, giống animal-connect / runic-blaze).

### ⚠️ Phát hiện quan trọng: trọng lực làm DEADLOCK NHIỀU HƠN, không ít hơn

Giả định ban đầu (ghi trong plan) là trọng lực sẽ *giảm* deadlock. **Đo bằng
simulator thì ngược lại** — và đây là thứ định hình toàn bộ bảng level ở trên.

**Nguyên nhân cấu trúc:** Onet tĩnh, dọn một cặp để lại một **lỗ ở giữa bàn**, và
chính những cái lỗ đó là hành lang đi đường cho các cặp sau. Có trọng lực, mọi
lỗ **dồn hết lên đỉnh**, để lại một khối đặc mà chỉ **bề mặt** của nó là nối
được. Kết quả đo (chơi ngẫu-nhiên-đối-nghịch, qua đúng generator/pathfinder của
bản ship):

| Đòn bẩy | Kết quả đo |
|---|---|
| Thiết kế ban đầu (10×12, 14 loại) | **37–52%** ván bị kẹt ít nhất 1 lần |
| Số loại quả (cùng 1 bàn) | 6 loại **5%** · 10 loại 12.5% · 14 loại **37.5%** |
| Tỉ lệ khung (cùng số ô + số loại) | **12×10 → 7.5%** · **10×12 → 20%** |
| Hệ số kề `f` | Gần như không đổi deadlock, chỉ làm bàn "lộ cặp" hơn → giữ **thấp (0.15)** |

→ Hai đòn bẩy thật là **KHUNG (rộng thắng cao)** và **SỐ LOẠI** (giữ ~3+ cặp mỗi
loại trên bàn), **không** phải hệ số kề. Sau khi sửa: **trung bình 0.111 lần xáo
mỗi màn** (3–16% tuỳ màn), **0 bàn không giải được**, adjacency ~34%. Đây là số
*đối-nghịch*; animal-connect đo chơi thường tốt hơn ~4×, tức người chơi thật gặp
auto-xáo khoảng **1 lần mỗi ~30 màn** — đúng nghĩa lưới an toàn.

## Nghệ thuật — nguồn stock, KHÔNG generate

- **Icon: Fluent Emoji "Flat" (Microsoft) — license MIT.** 14 quả/rau vector
  32×32, không viền, **không gradient/filter/defs** (đã grep xác nhận) nên render
  cực rẻ. Gộp thành **một sprite `<symbol>` 20 KB**; mỗi tile là một `<use>`.
  Nghĩa vụ license duy nhất là giữ notice — đã để trong comment đầu `index.html`.
- **Bộ 14 quả chốt bằng ảnh chụp thật ở cỡ tile thật**, không chọn bằng cảm tính:
  - `strawberry` bị loại — ở 30px nó và `apple` đều là đốm đỏ tròn, không phân biệt được.
  - `blueberries` (ứng viên thay) cũng bị loại — nó **đụng `grapes`** (cùng là chùm
    quả mọng tím), sửa được lỗi này lại đẻ ra lỗi tương đương.
  - → chọn **`mushroom`**: silhouette mũ-vòm-cộng-thân là duy nhất trong bộ, và là
    món hai tông đỏ/kem duy nhất.
- **Phong cách: neo-brutalist in ấn PHẲNG HOÀN TOÀN** — nền giấy kem + chấm
  halftone, viền mực đen 3px, khối màu phẳng, mọi animation dùng `steps()`.
  **Không có bóng đổ nổi/3D ở bất kỳ đâu** (theo yêu cầu 2026-08-04: bỏ hết UI
  kiểu nút 3D) — nút/pill/khung bàn/banner chỉ còn viền mực; nút bấm phản hồi
  bằng thu nhỏ + đổi nền, không phải "lún vào bóng". Các `box-shadow` còn lại
  đều là **vòng offset 0** (chọn / gợi ý), không phải bóng đổ.
  **Không blur, không `filter`, không `backdrop-filter`, không gradient màu** —
  `radial-gradient` duy nhất trong file là hoạ tiết **chấm halftone** (một
  pattern lặp, compositor vẽ một lần rồi thôi), không phải chuyển sắc.
- **Trạng thái chọn cố tình KHÔNG dùng màu**: 14 quả 14 màu nên bất kỳ vòng accent
  nào cũng đụng một quả — vòng đỏ tomato trên tile táo từng gộp thành một khối đỏ
  đặc không đọc được. Nay chọn = **phóng to + viền mực dày**, đọc được trên mọi
  icon. Hint dùng **xanh dương** vì trong bộ **không có quả nào màu xanh dương**.

## Tối ưu hiệu năng

| Hạng mục | Cách làm |
|---|---|
| **Không có vòng lặp rAF** | Nền là CSS, tile rơi là transition của compositor, link fade bằng CSS → **không có gì cần 60Hz**. Việc định kỳ duy nhất là đồng hồ ở **5Hz** (`setInterval` 200ms). |
| **Nền không tốn gì** | Nền **chỉ còn chấm halftone** vẽ một lần (các khối tròn/vuông trôi đã bỏ theo yêu cầu 2026-08-04). Không canvas, không animation nền → animal-connect phải tối ưu vòng vẽ nền xuống 14 lệnh canvas/frame; ở đây là **0**, và giờ không còn cả element nào động. |
| **Tile rơi** | Đổi `transform` + `transition` → compositor lo. **Không một dòng JS nào chạy mỗi frame trong lúc rơi.** |
| **Tìm cặp** | BFS **theo nguồn**: một lần quét tia đánh dấu mọi tile nối được từ một nguồn → nhóm k tile cùng loại tốn **k** lần quét thay vì **k(k−1)/2** lần tìm đường. Thoát sớm + cache cặp tìm được cho auto-hint. |
| **Cấp phát** | `Int32Array` cấp **một lần mỗi level**, **đóng dấu bằng `scanId`** thay vì `fill(0)` → một lần quét không tốn gì cho việc dọn buffer. |
| **DOM tile** | Pool cấp sẵn ở cỡ bàn max (120), tái dùng vĩnh viễn; đổi loại = 1 `setAttribute`. Không tạo/huỷ node lúc chơi. |
| **`backdrop-filter`** | **Không dùng ở bất kỳ đâu** (bị ban vì perf trên WebView; style phẳng cũng không cần). |
| **Bridge** | `save_data` từ **~260 → 40 message**/phiên: điểm cộng dồn nên qua kỷ lục cũ là *mỗi* nước đều lập best mới. Nay localStorage ghi ngay (miễn phí, trong WebView) còn native bị **throttle 4s + trailing send**; hết màn/hết run vẫn gửi ngay không throttle. |

## Tuân thủ quy ước chung

- ✅ **game-common.md** — `sendMessage` đủ 5 trường, level 1-indexed;
  `game_result` win/lose + `showModal:true`; Back → `quit`; `ads` mỗi 3 ván;
  `save_data` chỉ data nhẹ, ván dở lưu `localStorage`; đọc đủ
  `statusBarHeight`/`currentLevel`/`data`/`language` đúng thứ tự ưu tiên;
  `waitForNativeInjection`; 3 callback native; font Google Sans Flex qua
  `--ui-font` (ngoài Latinh → `system-ui`); reset CSS + tắt tap-highlight.
- ✅ **popup-common.md** — game theo level nên **không tự vẽ popup kết quả**
  (app lo, `showModal:true`).
- ✅ **zip-common.md** — single-file, `game.json` đủ field & pass script check,
  3 cover đúng **1920×1080 / 800×1200 / 800×800** (verify từ byte 16–23 của PNG).
- ✅ **i18n** — inline; **chỉ TUTORIAL dịch đủ 23 ngôn ngữ** (theo tiền lệ
  animal-connect / QA "thừa localize"), HUD & toast dùng tiếng Anh qua fallback
  `en` — deviation chủ đích so với game-common §5.
- ℹ️ `button-common.md` / `background-common.md` được các spec khác tham chiếu
  nhưng **không tồn tại trong repo** (git history xác nhận chưa từng có) →
  `btn3d` và pattern nền lấy từ runic-blaze/animal-connect như các game trước.
- ℹ️ Mở bằng browser thường: tự giả lập popup app (next/retry sau 1.4s) để
  playtest — trong app thật không chạy.

## Đã kiểm thử (headless Edge)

- **Bot 9 màn: 404 lần nối, 1 lần auto-xáo, 0 lỗi JS, 0 vi phạm bất biến.** Sau
  **mỗi lần lắng** đều assert: `grid[][]` khớp `tiles[][]`, mọi cột liền mạch từ
  đáy, `transform` DOM khớp ô lưới, số tile = 2×`pairsLeft`, và **không loại nào
  còn số lẻ** (parity cặp). Bridge: `game_result/win` ×9, `ads` ×3 (mỗi 3 ván).
- **Tutorial**: `bendTurns = 3` (đúng bài học "thừa 1 lần rẽ"), `dropFromFall =
  true` (cặp bước 2 **thật sự** do cú rơi tạo ra), score/best **giữ 0** suốt và
  sau tutorial (bug rò điểm từng làm phồng best ở animal-connect), rồi vào màn 1.
- **28 assertion edge-case pass**: race (`loadLevel` **giữa lúc tile đang rơi**
  → `levelEpoch` huỷ continuation cũ, bàn vẫn nhất quán), pause (đồng hồ đứng,
  veil hiện, nền dừng) → resume (chạy lại), snapshot (khôi phục **đúng từng ô**,
  từ chối snapshot sai level), thua do hết giờ (đúng **một** `game_result/lose`,
  best thăng hạng, run reset về `levelIdx:0/score:0`), retry (về màn 1, giữ best).
- **Boot matrix**: `en` / `ar` (→ `dir=rtl` + `system-ui`) / `ja` (→ `system-ui`)
  / `vi` (giữ Google Sans Flex, chữ có dấu **đúng UTF-8, không mojibake**);
  `data.levelIdx` khôi phục đúng màn + score/best; `currentLevel` từ native
  **thắng** levelIdx đã lưu; `?reset=1` xoá localStorage.
- **Layout**: `#board-outer` `overflow:hidden`, dò scroll trên **đúng container
  cuộn** (không dùng `documentElement` — cái bẫy từng giấu bug của
  animal-connect cả một session) → **0 scroll cả hai trục**.
- ⚠️ **Chưa kiểm được**: Edge trên máy này **bỏ qua `--window-size` cho CSS
  viewport** (đo được: xin 412/360/320 đều ra **492×822**), nên **layout ở màn
  hẹp thật chưa reproduce headless được**. Công thức cỡ ô có sàn 16px và ở
  492px cho `cellPx = 35`; ước tính ở 360px là ~27px và ở 320px là ~23px —
  **cần playtest trên cửa sổ Edge app-mode 420×800 hoặc máy thật để xác nhận**.

## 📋 Backlog

- [ ] **3 cover PNG vẫn vẽ tile CÓ bóng đổ cứng** (sinh trước khi bỏ UI 3D) →
      lệch với bản game phẳng hiện tại. Chưa sinh lại: cover là art cần anh
      duyệt. Nói một tiếng là em render lại trong 1 lệnh.
- [ ] Kiểm layout ở màn hẹp thật (320/360px) — xem cảnh báo ở trên.
- [ ] Hướng trọng lực biến thể theo level (lên / trái / phải / vào giữa).
- [ ] Dịch nốt HUD/toast cho 23 ngôn ngữ nếu mentor muốn (hiện chủ đích để `en`).
- [ ] Confetti khi thắng màn; hiệu ứng riêng cho DROP COMBO ×3 trở lên.
