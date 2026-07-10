# Animal Connect — QuanKA

> **Onet / Pikachu connect puzzle** | HTML5 single-file | Level vô hạn sinh theo công thức

## Thông tin game

| Trường | Giá trị |
|---|---|
| **Package** | `com.falcon.animalconnect` |
| **Engine / Version** | HTML5 single-file · 1.0.0 |
| **Category** | PUZZLE |
| **Loại** | Game **có level** — vô hạn, thắng/thua theo màn |

## Gameplay

- Nối 2 tile **cùng con thú** bằng đường ≤ 3 đoạn thẳng (**≤ 2 lần rẽ**) qua ô
  trống; đường được vòng ra ngoài biên. Nối đúng → line phát sáng + tile nổ.
- Sau mỗi match, tile còn lại **dồn theo hướng của level** (none/down/up/left/
  right/in/out). **Sạch bàn = thắng màn** (không có đồng hồ → không thua).
- **Điểm = số NGÔI SAO** ★ (đặc trưng game): mỗi lần nối, sao xuất hiện **dọc
  đường connect** (mỗi ~1 cạnh tile 1 sao, từ điểm đầu → điểm cuối) rồi **bay
  lên ô SCORE**. Đường càng dài → càng nhiều sao → càng nhiều điểm (10đ/sao).
- **Không có Time / Hint / Shuffle** trong match. Thay vào đó **~5s không thao
  tác** mà chưa nối được thì game **tự nháy 1 cặp** nối được (auto-hint); hết
  nước đi vẫn tự xáo miễn phí.
- **14 mặt thú** vẽ inline SVG; theme đồng cỏ pastel, lá + dấu chân trôi trên canvas.
- **Không có menu** — mở app là vào thẳng scene gameplay. Lần đầu chơi tự vào
  **tutorial** trước rồi mới sang màn 1.
- **Header chuẩn** (theo `cursed-knives`): overlay `position:fixed` — **Back**
  (trái) · **SCORE ★ · BEST ♛** (2 pill bằng nhau, icon ghim trái, BEST accent
  vàng) · **Volume** (phải, bật/tắt SFX); dưới là **LEVEL**. Back = lưu ván dở +
  `quit`. `BEST` = điểm cao nhất, persist qua `save_data` + localStorage.
- **Layout**: board nằm gọn dưới header (padding-top = chiều cao header), **không
  cuộn ngang/dọc** (`#board-outer overflow:hidden`), chừa nửa ô cho đường vòng biên.
- **Độc lập refresh rate**: mọi chuyển động theo `dt` (nền lá chuẩn hoá 60fps,
  clamp dt 0.25s chống stall) — test 30fps vs 144fps kết quả giống nhau.
- **SFX tổng hợp Web Audio** (không file ngoài, không nhạc nền): chọn/nối (cao
  dần theo combo, bù loudness khi lên cao)/sai cặp/auto-hint/xáo/thắng + click
  nút. Nút loa ở header (cờ `muted` persist); mở khoá sau cử chỉ đầu (autoplay
  policy), app ra nền / pause là suspend toàn bộ.
- **Tutorial 1 BÀN, không chuyển cảnh** — bước 0 nối 2 con **giống nhau** → bước
  1 **thử nối 2 con khác nhau** (học rằng khác loại **không nối được**) → bước 2
  dọn sạch bàn. **Bắt buộc lần đầu mở game** (cờ `tutorialSeen` persist), có nút
  Skip. Tutorial **không tính điểm**. `?reset=1` xoá state để xem lại từ đầu.

### Level sinh theo công thức — `levelConfig(idx)`

Deterministic theo số màn (hash, retry ra đúng màn đó); xếp tile random mỗi lần chơi.

- **Bàn**: ramp 6×6 → 8×9 trong 8 màn đầu, sau đó xoay vòng bàn lớn (tối đa 8×10, tổng ô luôn chẵn).
- **Dồn tile**: 7 màn đầu đi đủ tour các mode, sau đó bốc theo hash (bỏ `none`).
- **Loại thú**: 12 → 13 → 14 từ màn 3 (nhiều loại để ít cặp trùng).
- (Trường `time`/`hints`/`shuffles` trong `levelConfig` còn nhưng **không dùng** — đã bỏ Time/Hint/Shuffle.)
- Vô hạn → **không có màn cuối, không bắn `victory`**; thắng luôn gửi
  `game_result win` + `save_data {levelIdx: n+1, best}`.

## Tuân thủ quy ước chung

- ✅ **game-common.md** — `sendMessage` 5 trường, level 1-indexed; `game_result`
  win/lose + `showModal:true`; Back → `quit`, `ads` mỗi
  3 ván; `save_data` chỉ data nhẹ (`levelIdx`/`tutorialSeen`/`muted`/`best`), ván
  dở lưu localStorage; đọc đủ
  `statusBarHeight`/`currentLevel`/`data`/`language` đúng thứ tự ưu tiên;
  `waitForNativeInjection`; 3 callback native; font Google Sans Flex qua
  `--ui-font` (ngoài Latinh → `system-ui`); reset CSS + tắt tap-highlight.
- ✅ **popup-common.md** — không tự vẽ popup kết quả (app lo, `showModal:true`).
- ✅ **zip-common.md** — single-file, `game.json` đủ field, 3 cover đúng kích thước.
- ✅ **i18n** — inline, đủ 23 ngôn ngữ, fallback `en`.
- ℹ️ Mở bằng browser thường (không có `ReactNativeWebView`): tự giả lập popup
  app (next/retry sau 1.4s) để playtest — trong app thật không chạy.

## Đã kiểm thử (headless Edge + bot)

- Boot en/ar/ja, `data={"levelIdx":3}` vào đúng màn 4, `?currentLevel=137` chạy tốt.
- Bot phá màn 1 và màn 10: sự kiện + tiến độ đúng; hết giờ ra đúng một `lose`; 0 lỗi JS.
- Flow first-open: menu → Play → tutorial 3 bước (bot tự chơi) → `save_data`
  kèm `tutorialSeen:true` → vào màn 1 thắng bình thường; người chơi cũ
  (`tutorialSeen` đã lưu) bấm Play vào thẳng ván.
- Review đa-agent (33 agents): 20 lỗi xác nhận đã sửa hết (race thắng/thua sát giờ,
  continuation async sống sót qua loadLevel, desync grid/DOM khi restore, v.v.).

## 📋 Backlog

- [ ] Confetti khi thắng màn; thêm mặt thú mới (voi, sư tử…)
- [ ] Sao 1–3 theo thời gian còn lại → meta progression qua `save_data`
- [ ] Daily challenge seed cố định (app truyền seed qua `data`)
