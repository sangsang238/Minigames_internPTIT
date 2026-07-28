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
- Tile **LUÔN đứng yên** sau khi nối (không có cơ chế dồn). **Sạch bàn trước khi
  hết giờ = thắng màn** → sang màn kế; **hết giờ = THUA = endgame** (run kết thúc,
  chơi lại từ màn 1). Hết nước đi tự xáo miễn phí.
- **Đồng hồ đếm ngược** ở dòng dưới header: **LEVEL x** (trái) · **thanh time**
  (giữa) · **số giây** (phải) — cả dòng dài đúng bằng & khớp dọc header. Giây/level
  giảm dần theo độ khó (`time = pairs × max(2.4, 5.5−idx×0.22)`, làm tròn 5s).
- **Điểm CỘNG DỒN cả run** (lvl 1 → endgame): mỗi màn thắng giữ nguyên điểm, sang
  màn kế cộng tiếp; thua thì `BEST = max(BEST, điểm run)`, run mới về 0. SCORE pill
  = điểm run hiện tại, BEST = điểm run cao nhất. Persist `{levelIdx, score, best}`.
- **Điểm = số NGÔI SAO** ★ (đặc trưng game): mỗi lần nối, sao xuất hiện **dọc
  đường connect** (mỗi ~1 cạnh tile 1 sao, từ điểm đầu → điểm cuối) rồi **bay
  lên ô SCORE**. Đường càng dài → càng nhiều sao → càng nhiều điểm (10đ/sao).
  Nhịp sao **đều** cho mọi độ dài (gần/xa cùng cảm giác), rải so le
  (STAR_STAGGER) + đậu 1 nhịp (STAR_DWELL) rồi bay (transition .7s).
- **Đồng hồ ẩn — thưởng nối nhanh**: đo thời gian *tìm ra* mỗi cặp (từ lần nối
  trước / lúc bàn sẵn sàng); <1.1s → **+20** kèm "Cực nhanh!", <2.2s → **+10** kèm
  "Nhanh!" (lời khen nổi giữa màn). Không có thanh giờ hiện.
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
- **Tutorial 1 BÀN 5×5, không chuyển cảnh** — dạy 3 bước bằng **hình**:
  1. **Nối** 1 cặp được chỉ (chỉ cho nối đúng cặp đó) → hiện đường nối + **tick ✓**.
  2. Thử nối 1 cặp **CÙNG loại BỊ CHẶN** → game **VẼ đường "quá vòng" 3 lần rẽ**
     (đỏ nét đứt + chấm ở mỗi chỗ rẽ) cho thấy vì sao không nối được (>2 rẽ).
     Bấm **nhầm** cặp giống-nối-được → nháy lại cặp đúng + nhắc "chạm cặp đang sáng".
  3. **Dọn nốt** cả bàn (cặp bị chặn tự mở ra sau khi dọn blocker — bàn đã verify
     giải được hết). Bước này **hiện timer demo 15s** ở header + **mũi tên** chỉ lên
     nó, banner đổi thành "…clear the whole board **before the timer runs out**".
  Bàn + cặp 3-rẽ đã **verify bằng máy** (`findPath`=null, `findMinTurnPath`=3 rẽ,
  bỏ cặp bước 1 vẫn chặn, greedy solve = 0 ô thừa). **Bắt buộc lần đầu**
  (`tutorialSeen`), có Skip, **không tính điểm**. `?reset=1` xem lại.

### Level sinh theo công thức — `levelConfig(idx)` + `genBoardGrid()`

`levelConfig` trả `{cols, rows, types, time}`; xếp tile random mỗi lần chơi.

- **Độ khó tăng dần CẢM NHẬN ĐƯỢC — 2 pha:**
  - **Pha 1 (L1–10)**: bàn lớn dần `6×6 → 6×8 → 7×8 → 8×8 → 8×9 → 8×10 → 8×11 →
    8×12 → 9×12 → 10×12` (36→120 ô) — **mỗi màn bàn to ra thật**.
  - **Pha 2 (L11+)**: bàn giữ **MAX 10×12**, `perPair = max(2.0, 4.4−idx×0.16)` siết
    dần → **timer GIẢM ĐỀU mỗi màn** (180→120s tới ~L16 rồi plateau). Cùng bàn to
    nhưng ít giờ dần = tín hiệu khó rõ. `time = round(pairs × perPair / 5) × 5`.
  - Tổng giờ: **80s (L1) → 180s (L10) → 120s (L16+)** — leo theo bàn rồi siết lại.
  - 🔧 **Sửa theo QA 2026-07-27** ("map rộng hơn qua các level, hiện tại đang giữ
    1 kích thước nên ko có thử thách lắm"): trước đây bàn **chốt MAX 8×10 ngay từ
    L5** nên từ màn 6 trở đi chỉ còn timer đổi. Dốc `perPair` cũng thoải hơn
    (0.18→0.16/màn) vì bàn to đã gánh bớt độ khó — tìm 1 cặp trong 120 ô lâu hơn
    trong 80 ô, không để hai trục khó cộng dồn.
  - **Trần 10×12 là giới hạn LAYOUT**: `cellPx = min(availW/COLS, availH/(ROWS+1), 58)`
    có sàn 16px — quá sàn thì `overflow:hidden` cắt bàn. Đã đo: máy 412px → 37px/ô,
    máy hẹp 320px → 28px/ô, landscape 915×412 → 22px/ô — **mọi cấu hình vẫn > sàn**.
- **Loại thú**: 12 → 13 → 14 từ màn 3 (nhiều loại để ít cặp trùng).
- **Sinh bàn = SINH NGƯỢC (reverse-gen)**: đặt từng cặp vào 2 ô **nối được** trên
  bàn đang lấp → bàn **luôn giải được**. Với xác suất `f = max(0.2, 0.28−idx×0.01)`
  đặt cặp **KỀ nhau** (an toàn), còn lại rải rác. Trần `f` hạ thấp để **bớt cặp kề
  nhìn thấy** (tỉ lệ ô có hàng-xóm-cùng-loại: level 1 ~45%, nền coincidental ~29%).
  Verify mô phỏng chơi ngẫu-nhiên-đối-nghịch: **~2.3% adversarial** (chơi thường
  ~0.6%) → hiếm phải auto-xáo (độ khó chính do TIMER, không do cặp kề). Vẫn giữ
  auto-xáo làm lưới an toàn.
- Vô hạn → **không có màn cuối, không bắn `victory`**; thắng luôn gửi
  `game_result win` + `save_data {levelIdx: n+1, best}`.

## Text nổi trong màn: bỏ nền (2026-07-28)

Theo QA/mentor: text nổi trong màn **không dùng nền/viền**, thay bằng **outline
mảnh** cùng màu nền cũ. Ở game này chỉ `#toast` (thông báo tự xáo) có nền — đã bỏ
`background` + `border-radius` + `padding`, chữ **14→16px** đậm hơn, outline
`rgba(19,48,35,.95)` (đúng màu nền cũ) để vẫn đọc rõ trên nền đồng cỏ sáng.

`#praise` ("Nhanh!/Cực nhanh!") vốn đã không có nền (chỉ `text-shadow`) nên giữ
nguyên. SCORE/BEST và popup kết quả **không đụng** (là UI mặc định).

## Tối ưu hiệu năng (2026-07-28) — theo bộ tiêu chí mentor dùng cho hamster-jump

Đo bằng **đếm lệnh canvas**, không đoán từ đọc code (`performance.now()` đứng yên
dưới `--virtual-time-budget` nên mọi phép đo ms đều ra `0.0000`).

| | Trước | Sau |
|---|---|---|
| Lệnh canvas nền / frame | **139** | **14** |
| → quy ra mỗi giây | **8 340** | **840** |
| `backdrop-filter` đè lên canvas động | 2 | **0** |

**1. Lá & dấu chân — vẽ sẵn thành sprite.** Mỗi vật thể trước đây tô lại bằng
path **mỗi frame**: lá = `beginPath`+`moveTo`+2×`quadraticCurveTo`+`fill`, dấu
chân = 4×(`beginPath`+`arc`/`ellipse`+`fill`), cộng `save`+`translate`+`rotate`
+`restore`. Tất cả để vẽ 26 hình **y hệt khung trước**, trôi 0.05–0.21px/frame.
Nay: sprite vẽ ở cỡ lớn nhất rồi thu nhỏ khi blit (không vỡ nét), và
**`setTransform` gộp luôn translate+rotate+scale** → thay cả cụm 4 lệnh bằng
**đúng 1 lệnh**. Thêm `BG_STEP_MS = 33` (~30fps), `dt` gộp nên tốc độ trôi thực
không đổi.

**2. `backdrop-filter` đè lên canvas vẽ lại liên tục.** `.hud-pill` (blur 6px) và
`.btn3d` (blur 8px) **luôn hiện suốt ván** và nằm đè lên canvas nền → trình duyệt
phải **blur lại vùng nền dưới chúng mỗi khung**. Đã bỏ: nền dưới pill vốn gần
phẳng (lá chỉ opacity .07–.23) và `--glass` đã là trắng 42%, `--surface-color`
trắng 50% — bỏ blur gần như không đổi hình. **Giữ** blur ở lớp phủ pause/kết quả
vì nó chỉ hiện lúc dừng, và nay `mainLoop` **ngừng vẽ nền khi pause** (trước vẫn
vẽ 60fps cho một màn hình bị che kín).

**3. Lỗi thật, không phải hiệu năng.** `@media (max-height: 640px)` chứa
`#tut-banner { padding: 9px 12px }` nhưng **đứng TRƯỚC** rule gốc cùng độ ưu tiên
→ rule gốc ghi đè ngược lại, **override cho màn thấp chưa từng có tác dụng**. Đã
chuyển khối `@media` xuống cuối `<style>`. Gộp thêm `#time-bar-fill.danger` (khai
báo 2 khối cùng selector).

**Kiểm chứng:** boot màn 3 → **56 tile** đúng `7×8`, nền trích ra PNG vẫn đủ lá +
dấu chân xoay đúng góc, **0 lỗi JS**.

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
- ✅ **i18n** — inline; **chỉ TUTORIAL dịch đủ 23 ngôn ngữ** (theo QA 2026-07-22: "thừa localize"); Level/SCORE/BEST/toast… dùng tiếng Anh qua fallback `en` — deviation chủ đích so với game-common §5.
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
- **Bàn to (2026-07-27)**: sinh 150 bàn/màn × 16 màn — **100% lấp kín**, 0.08–0.21 ms/bàn
  (bàn 120 ô không chậm hơn đáng kể), tỉ lệ phải auto-xáo **0–4%** ở mọi cỡ bàn
  (bàn 10×12 KHÔNG deadlock nhiều hơn bàn 8×10). Boot `levelIdx` 9/15 → đúng
  `10×12`, 120 tile trong DOM, board 460×552 lọt viewport, 0 lỗi JS.

## 📋 Backlog

- [ ] Confetti khi thắng màn; thêm mặt thú mới (voi, sư tử…)
- [ ] Sao 1–3 theo thời gian còn lại → meta progression qua `save_data`
- [ ] Daily challenge seed cố định (app truyền seed qua `data`)
