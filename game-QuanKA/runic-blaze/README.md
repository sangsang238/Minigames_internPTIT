# Runic Blaze — QuanKA

> **Match-3 endless high-score** | HTML5 single-file | **Flat-matte (phẳng)** — tile khối màu đặc + icon trắng, nền teal | v3.3 — hazard **Cursed Runes** + events + onboarding

---

## Thông tin game

| Trường | Giá trị |
|---|---|
| **Tên game** | Runic Blaze |
| **Package** | `com.falcon.runicblaze` |
| **Engine** | HTML5 (single-file `index.html`) |
| **Version** | 3.3.0 |
| **Giao diện** | Flat-matte (phẳng) — tile KHỐI MÀU ĐẶC + icon trắng, nền teal-slate dịu, không neon/glow/gloss/gradient/3D |
| **Category** | PUZZLE (điểm-cao, endless) |
| **Entry** | `runic-blaze/index.html` |

> v2 là match-3 theo **level**. v3 chuyển sang **score-attack vô tận** — chơi một mạch, tích điểm.
> Khung header + vòng điểm-cao (self-popup, best live, Play Again) **mô phỏng skydom (HaND)**;
> **thẩm mỹ dark-fantasy** + hazard **Hắc Ấn** + **events** là của riêng runic-blaze.
> Base match-3 giữ cảm giác **thoả mãn kiểu skydom** (nổ liên hoàn, board sạch), độ khó đến từ **Hắc Ấn**.

---

## Gameplay & Cơ chế

### Core Loop (endless — KHÔNG level, KHÔNG giới hạn lượt)
1. **Board 7×7**, 5 màu rune (thêm màu thứ 6 khi điểm ≥ 5 000), sinh không match sẵn, luôn có nước đi
2. **Tap** chọn rồi tap ô kề để swap, hoặc **swipe** — swap kích hoạt ngay khi kéo đủ 35% ô
3. Swap hợp lệ (≥3 cùng loại hoặc combo special) → xoá → gravity → refill → **cascade** nhân điểm (juicy!)
4. Swap không tạo match → trả về + rung (không mất gì)
5. Chơi tới khi một **Hắc Ấn** phát nổ (hoặc hết nước đi) → Game Over

### Hắc Ấn — Cursed Rune (hazard chính, càng lâu càng khó)
- Định kỳ sinh **1 ô nguyền** (ấn đỏ máu có **SỐ đếm ngược**): **bất động, không ghép, không đổi chỗ**
- **Mỗi lượt đếm ngược −1** (số hiện ngay trên ô; ≤2 thì nhấp nháy đỏ)
- **Phá:** ghép hoặc cho special nổ **CẠNH nó** (4 hướng) → tan (**+250 điểm**)
- **Về 0 mà chưa phá → detonate → THUA**
- **Leo thang theo số lần xuất hiện + điểm:** ngòi **7 → 3** (ngắn dần), sinh **dày hơn** (cadence 8→4 lượt), **tối đa 2 cùng lúc** khi điểm ≥ 14 000; **ân hạn 4 lượt đầu** chưa sinh

### Events (thưởng ở mốc điểm — "mechanics hay")
- **Arcane Surge** (mỗi 5 000 điểm) → tặng **1 Prism** ngẫu nhiên trên board + fanfare
- **Blaze Storm** (mỗi 12 000 điểm) → biến vài ô thành **Blaze** rồi kích **nổ dây chuyền** (bữa tiệc nổ)

### Rune đặc biệt (forge từ match to)

| Special | Cách tạo | Hiệu ứng |
|---|---|---|
| **Blaze Rune** (mũi tên 2 đầu ↔/↕) | Match 4 thẳng hàng | Nổ nguyên hàng/cột |
| **Nova Rune** (sao nổ 8 cánh) | Match hình L / T | Nổ 3×3 — **nổ 2 lần** |
| **Prism Rune** (bánh xe 6 múi màu) | Match 5 thẳng hàng | Swap với rune bất kỳ → thiêu toàn bộ rune màu đó |

Special **KHÔNG vẽ glyph màu nữa** — chỉ 1 icon trắng đậm + **viền trắng 3px**
(rune thường viền đen mảnh) → nhìn là biết ngay ô nào đặc biệt.

#### 🔧 Sửa nhầm lẫn biểu tượng (QA 2026-07-27)

QA báo 2 lỗi đọc-hình, đã sửa:

1. *"thay ký tự runic ở màu vàng, xanh dương, hồng cho đỡ nhầm với icon bom nổ đặc biệt"* —
   3 glyph này đều **nhọn/toả tia** nên đọc thành "nổ", nhất là **hồng** (sao 4 cánh)
   gần trùng icon **Nova** (sao 8 cánh = "bom"). Thay bằng hình **KHÔNG toả tia**:

   | Màu | Cũ | Mới |
   |---|---|---|
   | Xanh dương `--c1` | tinh thể nhọn + gợn zigzag | **giọt nước** tròn + nét sáng |
   | Vàng `--c3` | tia sét | **ấn lục giác** + gạch ngang |
   | Hồng `--c5` | sao 4 cánh + chấm giữa | **xoáy ốc** (1.55 vòng) |

   Đỏ (lửa) / lục (cây) / tím (trăng khuyết) giữ nguyên — QA không chấm, và cả 3
   đều bất đối xứng nên không lẫn với sao nổ đối xứng tâm.
2. *"ô đặc biệt nổ toàn bộ màu chưa rõ biểu tượng"* — **Prism** cũ là tinh thể
   trắng to + 6 chấm màu **bé xíu** quanh viền → chỉ còn đọc ra "viên đá trắng".
   Nay là **bánh xe 6 múi màu đầy ô** + lõi trắng: MÀU thành phần chính, nói thẳng
   "thiêu **toàn bộ** rune của **một màu**".

Combo 2 special (Prism+Prism xoá board, Prism+Blaze/Nova biến-cả-màu, Blaze+Blaze chữ thập, Nova+Nova 5×5…) giữ nguyên.

### 🔧 Đọc-được-hay-không (QA 2026-07-28)

- **Combo giờ NỔI HẲN** — *"combo cho nổi lên, hiện chữ trắng và mờ nên user ko
  để ý"* + *"thời gian giữ lại lâu hơn, giờ biến mất nhanh quá"*. Cũ là chữ trần
  nổi trên board (board cũng sáng màu → chữ chìm), animation .8s mà đã gồm cả
  fade nên chỉ ~0.6s đọc được. Nay bọc **chip nền đặc + viền theo màu bậc combo**
  (phẳng, cùng kiểu `#toast`), chữ khen **26→32px**, và animation 1.5s có **đoạn
  GIỮ NGUYÊN ~0.84s** trước khi mờ — trước đây hoàn toàn không có đoạn giữ này.
- **Bỏ toast báo Hắc Ấn** — *"thông báo có boom xuất hiện bằng text ở ngoài map
  cũng khiến user ko để ý"* → **bỏ hẳn**. Báo hiệu dồn hết **vào chính ô đó**,
  nơi mắt người chơi đang nhìn: vòng nổ to hơn (`cellPx×1.15 → ×1.6`) + ô **nhấp
  3 nhịp** (`curse-arrive`) + tiếng `sCurse()` như cũ. Khoá i18n `cursedSpawn`
  đã xoá.

### Điểm & Best
- Mỗi wave: `số tile × 40 × chain` (chain ×1→×5), mỗi special +80, mỗi Hắc Ấn phá +250
- **Best (kỷ lục) chạy LIVE** ở header, persist ngay khi vượt; ván sau vẫn nhớ
- **Game Over popup game tự vẽ** (popup-common §1.1): "Game Over" / "New Record!", SCORE · BEST (vàng), **Play Again**

### Tutorial — HƯỚNG DẪN TRỰC TIẾP TRONG MÀN (1 scene, KHÔNG chuyển cảnh)

> 🔧 Đổi theo QA 2026-07-28: *"tutorial thay text bằng hướng dẫn trực tiếp trong
> màn chơi thì hay hơn"*. Cũ là **carousel 3 thẻ text phủ lên board** — người
> chơi chỉ bấm Next, đọc xong vẫn không biết phải làm gì.

- **KHÔNG có menu chính.** Lần chơi đầu, game **dựng sẵn thế cờ trên CHÍNH bàn
  thật** rồi bắt người chơi **tự đi** nước được dạy — học bằng tay, không bằng chữ:
  1. **Match Runes** — thế X X ? , dưới ? là X → đổi 2 ô để thành **3 hàng ngang**.
  2. **Forge Specials** — thế X X ? X → đổi 2 ô để thành **4 hàng ngang → rèn Blaze**.
  3. **Cursed Runes** — thêm 1 **Hắc Ấn ngay KỀ** hàng sắp nổ → nổ là **phá được nó**.
- **2 ô cần chạm** viền vàng dày + nhịp thở; **các ô sẽ ghép cùng** (và Hắc Ấn ở
  bước 3) giữ **sáng nguyên** để thấy rõ *vì sao* nước đó ăn; **mọi ô còn lại mờ
  40%**. Chạm/đổi sai chỗ → **không ăn thua gì**, chỉ lắc 2 ô đúng để nhắc.
- **Không thể thua khi đang học**: bỏ qua đếm ngược Hắc Ấn / sinh thêm / event.
- Thanh hướng dẫn nằm **giữa header và bàn** (không che bàn — cả bài học nằm trên
  bàn); hụt chỗ thì rơi xuống sát đáy. Có **Skip**.
- **Chữ dùng nguyên bộ khoá `ob1T…ob3D` đã dịch đủ 23 locale** — chỉ đổi CHỖ hiện,
  **không phát sinh dịch mới**, vẫn đúng quy ước "chỉ tutorial được localize".
- Các lần sau → vào thẳng ván. Idle 5s → gợi ý; hết nước đi → tự xáo rune thường
  (special/Hắc Ấn giữ chỗ)

**Đã verify bằng bot headless** (chạy trọn 3 bước): mỗi bước tô đúng **2 ô**, có
làm mờ nền; **đi sai → bàn KHÔNG đổi** (chặn OK) ở cả 3 bước; đi đúng → sang bước
kế. Đối chiếu đúng bài học: **B1 (match-3) rèn 0 special**, **B2 (match-4) rèn 2
special**, **B3 phá Hắc Ấn 1 → 0**. Kết thúc: `ended=false` (không thể thua),
`tutorialSeen=true`, thanh ẩn, hết ô tô sáng, ván thường vẫn còn nước đi. 0 lỗi JS.

### UI trong trận (khung header chuẩn — mô phỏng skydom)
- **Header:** `Back` (trái) · `SCORE` · `BEST` (vàng) · `Volume` (phải). Chỉ header + board, **không sub-strip** (số đếm ngược nằm ngay trên ô Hắc Ấn)
- Back = lưu best + `quit`; Volume = mute (persist)

### Âm thanh (WebAudio synthesize 100%, không file ngoài)
- **BGM dark cinematic** (Rê thứ, string bed saw + pedal Rê + ♭II Neapolitan + trống trầm) — giữ từ v2
- **SFX** qua bus `sfxOut`: select/swap/pop(theo chain)/forge/beam/nova/prism/zap/shuffle/hint/**hắc ấn sinh**/lose/button
- Mute persist qua `save_data`; audio suspend khi pause/nền, unlock ở chạm đầu

---

## Kỹ thuật

- **1 vòng rAF + tween engine dt-based**: animation & sleep gameplay đều là tween pause-aware → mượt mọi Hz
- **levelEpoch**: mọi `await` re-check epoch — Play Again / event / storm giữa chừng không hỏng board mới
- **Hắc Ấn** = kind `K_CURSED`, `tile.curse` = số đếm ngược; loại khỏi mọi logic màu (`collectRuns`/`liveRunAt`/`findHintMove`/prism-target/mostCommonColor), rơi theo gravity; `tickCursed()` giảm số & phát hiện detonate, `maybeSpawnCursed()` sinh theo nhịp, phá = thêm vào tập xoá khi kề ô bị clear (trong `executeWave`)
- **Persistence (§1.2):** chỉ `{ best, muted, tutorialSeen }` — score-attack **không lưu ván dở**, mở lại = ván mới (giống skydom)
- **Đa màn hình**: board đo theo `#board-outer`; canvas nền + FX theo `devicePixelRatio`; landscape chỉ căn board
- **Theme PHẲNG (flat)**: nền đặc `--bg-c`; tile khối đặc + viền màu 1.5px (không gradient/glow/gloss `::after`); glyph 2 nét phẳng (thân màu + phụ tím nhạt, bỏ bloom/lõi trắng); nút `btn3d` phẳng (không lip/inset/đổ bóng); popup/pill/banner phẳng (bỏ text-shadow/box-shadow); FX hạt vẽ `source-over` màu đặc (bỏ `'lighter'`/ember-gradient); giữ chuyển-động (screen-shake, opacity-pulse, vòng rune gạch nét)

---

## Tuân thủ quy ước chung

- ✅ **game-common.md** — **game điểm-cao (§1.1):** `game_result` `result:null, showModal:false`, game **tự vẽ popup**; **Play Again → `retry_level`**; `ads` mỗi 3 lần thua; `level` luôn `null`; Back = `quit`; `statusBarHeight`; `onAppPause` (lưu best + pause) / `onRetryLevel` (ván mới) / `onNextLevel` (no-op). **Không dùng `victory`/`next_level`**
- ✅ **popup-common.md** — game không-level tự vẽ popup kết quả (SCORE/BEST/Play Again), auto-fit cỡ số nhiều chữ số
- ✅ **zip-common.md** — single-file, `game.json` đủ field, 3 ảnh cover đúng tên/kích thước
- ✅ **i18n** — inline `I18N`, **chỉ tutorial dịch đủ 23 ngôn ngữ** (khoá `ob1T…ob3D`, `skip`); mọi text UI khác (SCORE/BEST/popup/toast events…) dùng tiếng Anh qua fallback `en` — theo QA 2026-07-22 "localize thừa", deviation chủ đích so với game-common §5. Tutorial mới (2026-07-28) **tái dùng nguyên bộ khoá cũ**, không phát sinh bản dịch mới

---

## 📋 Backlog

- [ ] Hắc Ấn nhiều lớp (cần phá 2–3 lần) ở mốc điểm rất cao
- [ ] Milestone điểm (5k/10k/20k) → toast + FX ăn mừng riêng
- [ ] Haptic khi Hắc Ấn còn 1 lượt (cần native)

> ⚠️ **Cover PNG** (`cover-*.png`) đang là art match-3 dark-fantasy của v2 — đúng tông nhưng chưa thể hiện
> Hắc Ấn. Là art người dùng duyệt → **chờ xác nhận trước khi tạo lại**.
