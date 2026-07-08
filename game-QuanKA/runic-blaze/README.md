# Runic Blaze — QuanKA

> **Match-3 puzzle game** | HTML5 single-file | Dark fantasy theme | v2.0 — hồi sinh & viết lại toàn bộ

---

## Thông tin game

| Trường | Giá trị |
|---|---|
| **Tên game** | Runic Blaze |
| **Package** | `com.falcon.runicblaze` |
| **Engine** | HTML5 (single-file `index.html`) |
| **Version** | 2.0.0 |
| **Category** | PUZZLE |
| **Entry** | `runic-blaze/index.html` |

> v1.0 từng bị abandon vì bug đóng băng board (busy-lock trong chuỗi async CSS-transition).
> v2.0 viết lại từ đầu trên **tween engine dt-based chạy trong một vòng rAF duy nhất** — mọi
> nhịp gameplay (swap/nổ/rơi/sleep) đều là tween pause-aware, không còn `setTimeout` đua với
> CSS transition → hết hẳn class bug treo `busy`.

---

## Gameplay & Cơ chế

### Core Loop
1. **Board 7×7**, 5 màu rune (level 9+ thêm màu thứ 6), sinh ngẫu nhiên không match sẵn, luôn đảm bảo có nước đi
2. **Tap** chọn rồi tap ô kề để swap, hoặc **swipe** — swap kích hoạt ngay khi kéo đủ 35% ô (không cần thả tay)
3. Swap hợp lệ (tạo ≥3 liên tiếp hoặc là combo special) → xoá → gravity → refill → cascade
4. Swap không hợp lệ → trả về + rung, **không** trừ moves
5. Đạt `scoreGoal` trước khi hết `movesLeft` → thắng; hết moves → thua

### Rune đặc biệt (forge từ match to)

| Special | Cách tạo | Hiệu ứng |
|---|---|---|
| **Blaze Rune** (mũi tên ↔/↕) | Match 4 thẳng hàng | Nổ nguyên hàng (match ngang) / cột (match dọc) |
| **Nova Rune** (vòng tròn) | Match hình L / T | Nổ 3×3 — **nổ 2 lần**: sống sót lần đầu, rơi xuống rồi nổ tiếp wave sau |
| **Prism Rune** (bánh xe 6 màu) | Match 5 thẳng hàng | Swap với rune bất kỳ → thiêu toàn bộ rune màu đó (tia sét arc) |

### Combo 2 special (swap 2 special kề nhau)

| Combo | Hiệu ứng |
|---|---|
| Prism + Prism | **Xoá toàn bộ board** |
| Prism + Blaze | Biến cả một màu thành Blaze Rune rồi kích nổ toàn bộ |
| Prism + Nova | Biến cả một màu thành Nova đã nạp → nổ chuỗi 3×3 |
| Blaze + Blaze | Chữ thập: nguyên hàng + nguyên cột |
| Blaze + Nova | Chữ thập dày: 3 hàng + 3 cột |
| Nova + Nova | Vụ nổ 5×5 |

### Điểm & Cascade
- Mỗi wave: `số tile × 40 × chain` (chain ×1→×5 cap), mỗi special kích hoạt +80
- Chain ≥2 hiện badge khen theo tier: Nice! → Great! → Awesome! → Blazing! → Legendary!
- **Blaze Rush** khi thắng: moves thừa (tối đa 6) tự biến thành Blaze Rune và nổ dây chuyền cộng điểm trước khi bắn `game_result`

### Level Design — VÔ HẠN (chơi bao lâu tuỳ sức)

Level sinh theo công thức **deterministic** `levelConfig(idx)` (hash splitmix theo idx —
retry sau khi thua dựng lại đúng envelope độ khó, chỉ cách xếp rune là random mới):

- **Moves**: 20 giảm dần theo level, sàn **16** (+1 ngẫu nhiên theo hash)
- **Điểm cần mỗi lượt**: `min(400, 100 + idx×22)` ±5% — tăng dần rồi **plateau ~400/lượt**
  để chơi dài vẫn công bằng; `goal = moves × perMove` (làm tròn 50)
- **Màu**: 5 màu tới level 8; từ level 9 là 6 màu, ~22% level hash cho 5 màu để đổi nhịp

Ví dụ: L1 ≈ 20 moves / 2 000 · L5 ≈ 18 / 3 500 · L10 ≈ 16-17 / 5 200 · L15+ ≈ 16 / 6 400 (plateau)

> ⚠️ **Deviation có chủ đích so với game-common §1.5**: level vô hạn → không tồn tại
> "level cuối" nên event `victory` **không bao giờ bắn** (giống animal-connect). Mọi màn
> thắng đều gửi `game_result/'win'` + `showModal: true`.

### Trợ giúp người chơi
- **Tutorial tương tác 3 bước** (board dựng tay + bàn tay chỉ dẫn): swap match-3 → forge Blaze → dùng Prism. Bắt buộc lần chơi đầu, skip được, mở lại từ menu
- **How-to-play panel** (sơ đồ SVG vẽ bằng chính art engine của game) — mở được từ menu **và ngay trong trận** (nút `?`)
- **Idle hint** sau 5s: cặp swap hợp lệ nháy sáng + tile tự nhích về hướng cần kéo
- **Deadlock-proof**: hết nước đi → toast + tự xáo (special giữ nguyên chỗ), board sinh mới luôn kiểm tra có nước đi
- **Level intro** (Level N · Goal · Moves) trước mỗi màn

### UI trong trận
- Topbar trái: **Back** (quit về app) + **?** (how-to-play)
- Topbar phải: **Sound** (mute persist) + **Home** (confirm → về menu, ván được lưu) + **Restart**
- HUD: chip Score (count-up) · progress bar goal (shimmer, nhấp nháy khi ≥80%) · chip Moves (đỏ khi ≤5)

### Âm thanh (WebAudio synthesize 100%, không file ngoài)
- BGM arpeggio giai điệu thứ, lookahead scheduler chống dồn nốt khi resume
- ~16 SFX: select/swap/invalid/pop (pitch tăng theo chain)/forge/beam/nova/prism/zap/shuffle/hint/win/lose/rush/button

---

## Kỹ thuật

- **1 vòng rAF + tween engine dt-based**: mọi animation & sleep gameplay đi qua tween → tự đóng băng khi pause, tốc độ đồng nhất ở 60/90/120/144 Hz
- **levelEpoch**: mọi `await` trong pipeline re-check epoch — `onNextLevel`/`onRetryLevel`/restart giữa chừng không thể làm hỏng board mới
- **Đa màn hình**: board đo theo `#board-outer` (flex) → tự khớp mọi ratio; landscape (w > 1.25h) tự chuyển HUD sang cột dọc cạnh board; canvas nền + FX scale theo `devicePixelRatio` (cap 3); media query cho màn hẹp/lùn; relayout khi `fonts.ready`
- **FX**: particle canvas riêng (composite `lighter`) — burst theo màu rune, beam quét hàng/cột, ring nova, tia sét prism re-jag mỗi frame, flash, screen-shake giảm dần theo hàm mũ
- **RTL**: tiếng Ả Rập tự bỏ letter-spacing (chữ nối liền không vỡ)

---

## Tuân thủ quy ước chung

- ✅ **game-common.md** — `sendMessage` 5 trường; `quit`/Back + Restart đúng SVG chuẩn; `ads` mỗi 3 lần kết thúc ván; `statusBarHeight` (URL ưu tiên + env safe-area, +8px); `currentLevel`/`data`/`language` window ưu tiên; `waitForNativeInjection` 200ms; `onAppPause` (lưu ván + pause thật: veil + đóng băng tween + suspend audio) / `onNextLevel` / `onRetryLevel`. **Riêng `victory` (§1.5) không dùng** vì level vô hạn — xem ghi chú ở Level Design
- ✅ **Lưu 2 tầng (§1.2)** — `save_data` chỉ data nhẹ `{levelIdx, muted, tutorialSeen}`; ván dở (board/moves/score) vào `localStorage`, validate kỹ khi restore (đúng level, đủ 7×7, giá trị hợp lệ, chưa thắng/thua) + đảm bảo còn nước đi
- ✅ **popup-common.md** — game theo level, `showModal: true`, không tự vẽ popup kết quả
- ✅ **zip-common.md** — single-file, `game.json` đủ field, 3 ảnh cover đúng tên/kích thước
- ✅ **i18n** — inline `I18N` **đủ 23 ngôn ngữ** (en, vi, zh-Hans, zh-Hant, ja, ko, es, es-MX, es-AR, pt-BR, pt, fr, de, it, nl, pl, ru, tr, ar, hi, th, id, fil) cho toàn bộ UI + tutorial + how-to-play + aria-label; Google Sans Flex + fallback `system-ui` cho ngôn ngữ ngoài Latin

---

## 📋 Backlog

- [ ] Objective đa dạng theo level (thu thập N rune màu X, phá ô băng…) ngoài score-goal
- [ ] Daily challenge / star rating (1–3 sao theo điểm dư)
- [ ] Haptic feedback qua bridge khi nổ lớn (cần native hỗ trợ)
- [ ] Leaderboard điểm tổng qua `save_data` mở rộng
