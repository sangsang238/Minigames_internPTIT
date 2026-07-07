# Runic Blaze — QuanKA

> **Match-3 puzzle game** | HTML5 single-file | Dark fantasy theme

---

## Thông tin game

| Trường | Giá trị |
|---|---|
| **Tên game** | Runic Blaze |
| **Package** | `com.falcon.runicblaze` |
| **Engine** | HTML5 (single-file `index.html`) |
| **Version** | 1.0.0 |
| **Category** | PUZZLE |
| **Entry** | `runic-blaze/index.html` |

---

## Gameplay & Cơ chế

### Core Loop
1. **Board 7×7**, 5 loại rune, sinh ngẫu nhiên (không có match-3 lúc spawn)
2. **Tap** để chọn tile → tap tile kề để swap; hoặc **swipe** trực tiếp sang ô bên cạnh
3. Swap hợp lệ (tạo ≥3 liên tiếp) → xóa → gravity → refill → tìm cascade
4. Swap không hợp lệ → animate trả về, **không** trừ moves
5. Trừ 1 move chỉ khi swap tạo được match

### Win / Lose
- **Win** : đạt `scoreGoal` trước khi hết `movesLeft`
- **Lose** : hết moves mà chưa đủ điểm
- **Level cuối (10)** : bắn `victory` thay vì `game_result`

### Cascade (điểm cộng)
Sau mỗi lần xóa, gravity + refill, nếu board tự có match mới:
- Cascade 1 (swap gốc): `×1`
- Cascade 2: `×2`
- Cascade 3+: `×3` (tối đa)

Score mỗi match: `50 × số tiles xóa × multiplier`

### 5 Loại Rune

| # | Tên | Màu | SVG Symbol |
|---|---|---|---|
| 0 | Flame | `#ff3d5a` Crimson | Ngọn lửa (teardrop + lỗ giữa) |
| 1 | Droplet | `#3d9fff` Azure | Giọt nước |
| 2 | Leaf | `#2dff8a` Emerald | Hai cánh lá đối xứng |
| 3 | Bolt | `#ffb830` Amber | Tia sét |
| 4 | Moon | `#c060ff` Violet | Lưỡi liềm |

### Level Design (10 levels)

| Level | Moves | Score Goal |
|---|---|---|
| 1 | 25 | 800 |
| 2 | 23 | 1 200 |
| 3 | 22 | 1 500 |
| 4 | 20 | 1 800 |
| 5 | 22 | 2 200 |
| 6 | 20 | 2 600 |
| 7 | 18 | 3 000 |
| 8 | 18 | 3 500 |
| 9 | 16 | 4 000 |
| 10 | 20 | 5 000 |

---

## Tuân thủ quy ước chung

- ✅ **game-common.md** — `sendMessage` 5 trường, `statusBarHeight`, `currentLevel`, `waitForNativeInjection`, `onAppPause/onNextLevel/onRetryLevel`, nút Back + Restart SVG chuẩn
- ✅ **popup-common.md** — game theo level, `showModal: true`, không tự vẽ popup
- ✅ **zip-common.md** — single-file, `game.json` đủ field, 3 ảnh cover đúng tên/kích thước
- ✅ **i18n** — inline `I18N` table, fallback `en`, hỗ trợ vi/zh/ja/ko/fr/de/ru/ar/th

---

## 🐛 Lỗi hiện tại (Known Bugs)

### ❌ [CRITICAL] Game đứng / không thể chơi sau 3–4 moves

**Triệu chứng:** Sau vài lần swap, board "đóng băng" — không nhận input nữa, tiles không move.

**Nguyên nhân nghi ngờ:**
- `busy` flag bị kẹt `true` do lỗi trong chuỗi `async/await` của `processCascade` hoặc `refillBoard`
- `refillBoard` dùng `requestAnimationFrame` double-pump bên trong `async` function nhưng không `await` đúng cách → race condition giữa tile transition và logic tiếp theo
- Khi `findMatches()` chạy ngay sau `refillBoard` trước khi tiles thực sự ổn định vị trí, có thể trả về false positive → cascade vô hạn → `busy` không bao giờ về `false`
- CSS `transition` còn bật trên tile cũ khi gọi `layoutTile` → tile "đang chuyển động" nhưng `board[][]` đã thay đổi → state desync

**Chưa fix** — cần refactor lại vòng `processCascade` + `refillBoard` dùng Promise-based timing thay vì RAF.

---

## 📋 Backlog — Cần cập nhật

### 🔴 Fix bugs (ưu tiên cao)
- [ ] **Fix busy-lock bug** — refactor `processCascade` + `refillBoard` để `busy` luôn về `false` sau mỗi move
- [ ] **Fix cascade vô hạn** — đảm bảo `findMatches` chỉ chạy sau khi tất cả tiles đã settle

### 🎨 Asset & Visual
- [ ] **Thay SVG inline bằng PNG asset** từ [Kenney.nl Puzzle Pack](https://kenney.nl) (CC0) cho tile trực quan hơn
- [ ] **Background** — thêm parallax nebula layer hoặc animated rune circle phía sau board
- [ ] **Tile clear effect** — thêm particle burst khi match xóa (hiện chỉ có scale animation)
- [ ] **Win/level-up animation** — flash board + radial burst trước khi gửi `game_result`
- [ ] **Sound FX placeholder** — Web Audio API: swap click, match pop, cascade whoosh

### ⚙️ Cơ chế (Mechanic) bổ sung
- [ ] **Shuffle button** — khi board không còn move hợp lệ, tự động hoặc cho phép shuffle 1 lần/level
- [ ] **Deadlock detection** — scan toàn board sau mỗi refill; nếu không có move hợp lệ nào → auto-shuffle
- [ ] **Hint system** — sau N giây không tương tác, highlight gợi ý 1 cặp swap hợp lệ
- [ ] **Score animation** — count-up từ giá trị cũ lên giá trị mới (thay vì nhảy ngay)
- [ ] **Level intro** — hiển thị "Level X" + goal 1 giây trước khi cho phép input
- [ ] **Combo streak visual** — thanh trạng thái combo hiển thị combo đang chạy (×1, ×2, ×3)
