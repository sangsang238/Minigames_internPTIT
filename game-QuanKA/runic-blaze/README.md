# Runic Blaze — QuanKA

> **Match-3 endless high-score** | HTML5 single-file | Dark fantasy | v3.1 — hazard **Cursed Runes** + events + onboarding

---

## Thông tin game

| Trường | Giá trị |
|---|---|
| **Tên game** | Runic Blaze |
| **Package** | `com.falcon.runicblaze` |
| **Engine** | HTML5 (single-file `index.html`) |
| **Version** | 3.1.0 |
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
| **Blaze Rune** (mũi tên ↔/↕) | Match 4 thẳng hàng | Nổ nguyên hàng/cột |
| **Nova Rune** (vòng tròn) | Match hình L / T | Nổ 3×3 — **nổ 2 lần** |
| **Prism Rune** (bánh xe 6 màu) | Match 5 thẳng hàng | Swap với rune bất kỳ → thiêu toàn bộ rune màu đó |

Combo 2 special (Prism+Prism xoá board, Prism+Blaze/Nova biến-cả-màu, Blaze+Blaze chữ thập, Nova+Nova 5×5…) giữ nguyên.

### Điểm & Best
- Mỗi wave: `số tile × 40 × chain` (chain ×1→×5), mỗi special +80, mỗi Hắc Ấn phá +250
- **Best (kỷ lục) chạy LIVE** ở header, persist ngay khi vượt; ván sau vẫn nhớ
- **Game Over popup game tự vẽ** (popup-common §1.1): "Game Over" / "New Record!", SCORE · BEST (vàng), **Play Again**

### Onboarding (Tutorial chuẩn — 1 scene, KHÔNG chuyển cảnh)
- **KHÔNG có menu chính.** Lần chơi đầu: **carousel 3 thẻ demo** (Ghép Rune → Rèn Đặc Biệt → Hắc Ấn) **phủ lên chính board thật** (board đã dựng sẵn phía sau, không đổi cảnh); Skip / Next / dots
- Các lần sau → vào thẳng ván. Idle 5s → gợi ý; hết nước đi → tự xáo rune thường (special/Hắc Ấn giữ chỗ)

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

---

## Tuân thủ quy ước chung

- ✅ **game-common.md** — **game điểm-cao (§1.1):** `game_result` `result:null, showModal:false`, game **tự vẽ popup**; **Play Again → `retry_level`**; `ads` mỗi 3 lần thua; `level` luôn `null`; Back = `quit`; `statusBarHeight`; `onAppPause` (lưu best + pause) / `onRetryLevel` (ván mới) / `onNextLevel` (no-op). **Không dùng `victory`/`next_level`**
- ✅ **popup-common.md** — game không-level tự vẽ popup kết quả (SCORE/BEST/Play Again), auto-fit cỡ số nhiều chữ số
- ✅ **zip-common.md** — single-file, `game.json` đủ field, 3 ảnh cover đúng tên/kích thước
- ✅ **i18n** — inline `I18N` **đủ 23 ngôn ngữ** cho UI/popup/aria + khoá `next`; mô tả onboarding + tên events hiện en+vi (còn lại fallback en — xem Backlog)

---

## 📋 Backlog

- [ ] Dịch nốt mô tả onboarding + tên events (surge/storm/cursedSpawn) cho 21 locale còn lại
- [ ] Hắc Ấn nhiều lớp (cần phá 2–3 lần) ở mốc điểm rất cao
- [ ] Milestone điểm (5k/10k/20k) → toast + FX ăn mừng riêng
- [ ] Haptic khi Hắc Ấn còn 1 lượt (cần native)

> ⚠️ **Cover PNG** (`cover-*.png`) đang là art match-3 dark-fantasy của v2 — đúng tông nhưng chưa thể hiện
> Hắc Ấn. Là art người dùng duyệt → **chờ xác nhận trước khi tạo lại**.
