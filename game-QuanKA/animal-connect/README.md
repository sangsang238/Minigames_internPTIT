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
  right/in/out). Sạch bàn trước khi hết giờ = thắng; hết giờ = thua.
- **Điểm**: +10/cặp, combo trong 4s +5×(n−1), thắng +2/giây còn lại.
- **Trợ giúp mỗi màn**: 3 Hint, 2 Shuffle; hết nước đi tự xáo miễn phí.
- **14 mặt thú** vẽ inline SVG; theme đồng cỏ pastel, lá + dấu chân trôi trên canvas.
- **Không có menu** — mở app là vào thẳng scene gameplay. Lần đầu chơi tự vào
  **tutorial** trước rồi mới sang màn 1.
- **Header chuẩn** (theo mẫu app): **Back** (trái) · **SCORE · BEST** (2 pill giữa,
  BEST = điểm cao nhất từng đạt, accent vàng) · **Volume** (phải, bật/tắt SFX).
  Back = lưu ván dở rồi bắn `quit`.
- **HUD phụ**: Level · thanh Time · Pairs còn lại. `BEST` persist qua `save_data`
  + localStorage; ván dở tự lưu, mở lại chơi tiếp.
- **HUD responsive**: font/pill/chip theo `clamp()`, media query gọn hoá khi màn
  thấp (≤640px/≤480px), board tự cuộn khi thiếu chỗ.
- **Độc lập refresh rate**: mọi chuyển động theo `dt` (nền lá chuẩn hoá 60fps,
  đồng hồ trừ theo giây thật, clamp dt 0.25s chống stall); DOM đồng hồ chỉ ghi
  ~5 lần/s — test giả lập 30fps vs 144fps cho kết quả giống hệt nhau.
- **SFX tổng hợp Web Audio** (không file ngoài, không nhạc nền): chọn/nối (cao
  dần theo combo, bù loudness khi lên cao)/sai cặp/hint/xáo/tick 10s cuối/thắng/
  thua + click nút. Nút loa ở header (cờ `muted` persist qua save nhẹ); mở khoá
  sau cử chỉ đầu (autoplay policy), app ra nền / pause là suspend toàn bộ.
- **Tutorial tương tác 3 bước** (nối cặp kề → đường rẽ 2 lần → đường vòng ra
  ngoài bàn) — **bắt buộc lần đầu mở game** (cờ `tutorialSeen` persist qua
  `save_data` + localStorage), có nút Skip.

### Level sinh theo công thức — `levelConfig(idx)`

Deterministic theo số màn (hash, retry ra đúng màn đó); xếp tile random mỗi lần chơi.

- **Bàn**: ramp 6×6 → 8×9 trong 8 màn đầu, sau đó xoay vòng bàn lớn (tối đa 8×10, tổng ô luôn chẵn).
- **Thời gian**: `số cặp × (8.5 − 0.35×idx)` giây, sàn **5.0 s/cặp**.
- **Dồn tile**: 7 màn đầu đi đủ tour các mode, sau đó bốc theo hash (bỏ `none`).
- **Loại thú**: 12 → 13 → 14 từ màn 3 (nhiều loại để ít cặp trùng).
- Vô hạn → **không có màn cuối, không bắn `victory`**; thắng luôn gửi
  `game_result win` + `save_data {levelIdx: n+1}`.

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
