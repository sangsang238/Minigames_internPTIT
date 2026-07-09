# Cursed Knives — QuanKA

> **Knife-Hit arcade game** | HTML5 single-file | Dark fantasy theme | v1.1.0

---

## Thông tin game

| Trường | Giá trị |
|---|---|
| **Tên game** | Cursed Knives |
| **Package** | `com.falcon.cursedknives` |
| **Engine** | HTML5 (single-file `index.html`) |
| **Version** | 1.1.0 |
| **Category** | ARCADE |
| **Entry** | `cursed-knives/index.html` |

> Clone cơ chế **Knife Hit** theo thẩm mỹ dark-fantasy của runic-blaze: dao găm arcane
> phóng vào **totem đá obsidian** đang xoay, sigil neon 3 lớp stroke (bloom mờ / màu
> chính / lõi trắng nóng), nền tinh vân + hồn lửa xanh trôi.

---

## Gameplay & Cơ chế

### Core Loop
1. Totem obsidian xoay ở trên, dao găm chờ ở dưới — **tap để phóng dao** (mỗi lúc chỉ 1 dao bay)
2. Dao cắm vào vành totem và **xoay theo totem**; một số totem có **dao cắm sẵn** (màu tím)
3. **Phóng trúng dao đã cắm → thua ngay** (dao văng, flash đỏ, popup sau 0.7s)
4. Cắm đủ số dao (cột đạn bên trái) → **totem vỡ tan** (mảnh đá bung ra ngoài + dao văng thừa hưởng vận tốc xoay) → **totem TRONG phình từ tâm ra trong ~1s** (nested reveal: phá lớp ngoài lòi lớp trong), **chặn touch tới khi phình xong**
5. Chơi **vô hạn** — thua thì chơi lại từ đầu, ganh điểm Best. **Không có bộ đếm "Stage/Level"** — chỉ mạch totem nối tiếp nhau (dòng nhỏ dưới header chỉ báo **BOSS**)

### Điểm (v1.1 — ranh giới chết là thang thưởng)
| Hành động | Điểm |
|---|---|
| Cắm 1 dao | **+1** |
| Sượt lưỡi dao khác (< ~10°) | **+2** "Sát nút!" |
| Sượt **sát hơn nữa** (< ~7.7°) | **+4** "Sát lưỡi!" (Razor) |
| Cắm **lọt giữa 2 dao** (cả hai < ~17°) | **+6** "Xỏ kim!" (Needle) |
| Phóng trúng **ngọc hồn** | **+5** |
| **Ngọc-sọ** trên TRÙM (Skull Tithe) | **+8** — nhưng totem nổi giận vọt tốc 0.5s (telegraph 0.4s) |
| Phá totem thường | **+2 + 2×số cú liều** trong stage (trần +10) |
| Phá totem **TRÙM** | **+10 + 3×số cú liều** (trần +25) |
| Kết liễu TRÙM trong pha NHANH của Last Stand | base **+20** "KẾT LIỄU!" (trần +30) |
| Phá totem **sâu hơn mọi run trước** | **+5** "Mốc sâu mới!" (Deepest Descent) |
| Vượt qua **bóng vọng** của run best | **+5** "Vượt bóng xưa!" (Echo Wraith) |

Best **persist ngay khi vượt** (không chờ hết ván) qua `save_data` + localStorage (merge MAX khi load — không bao giờ mất kỷ lục).

### Kịch tính v1.1
- **Kill-cam "The Culprit"**: chết là thế giới chậm còn 8% trong 0.56s, camera nghiêng vào điểm va chạm, dao thủ phạm phát sáng đỏ + cung cấm ±lưỡi dao, hiện **"Hụt N°"** — cái chết nào cũng đọc được, tap để skip
- **Executioner's Breath**: dao CUỐI khi vành đã ≥6 dao — thế giới nín thở (35% tốc độ, BGM nghẹt về 400Hz) rồi **snap về tốc độ thật đúng khoảnh khắc chạm**. Chậm ĐỀU cả totem lẫn dao (nhân `dt` một chỗ duy nhất) → thuần kịch tính, không phải trợ giúp
- **Omen Flick**: 120ms trước khi totem đảo chiều/vọt tốc, sigil lóe + riser — cố ý *không đủ* để phản ứng: chết vì pattern giờ đọc là "tôi tham" thay vì "game gian lận"
- **Last Stand**: TRÙM còn 1 dao là giãy chết (dao động 0.25×→1.35×, telegraph 0.8s) — chờ pha chậm ăn +10 an toàn, hay đâm pha nhanh lấy +20? Rình mãi thì **gai xương mọc** (vòng đỏ 6s, gai mọc 0.6s vô hại rồi thành vật cản, tối đa 2)
- **Deepest Descent**: phá totem sâu hơn mọi run trước được +5; vào lại stage kỷ lục có dấu ✦ cạnh nhãn stage
- **Echo Wraith**: stage là deterministic nên "nơi run best gục ngã" là vị trí cố định — bóng ma sọ tím lảng vảng đúng stage đó, cắm vượt qua là nó tan (+5)

### Stage Design — VÔ HẠN, sinh deterministic

`stageConfig(idx)` hash splitmix theo idx — cùng stage luôn cùng cấu hình (tốc độ, pattern, dao cắm sẵn, ngọc):

- **Dao cần cắm**: 6 → 9 (tăng mỗi 2 stage), TRÙM +2
- **Tốc độ xoay**: 1.8 rad/s ngay từ stage 1 (nhanh từ đầu), +0.14/stage, **trần 3.4 rad/s** ±8% jitter — khó nhưng không thành Dark Souls
- **Pattern xoay** (mở khóa dần, hash chọn để đánh lạc nhịp): `steady` → `wobble` (phồng-xẹp, thi thoảng khựng) từ stage 3 → `flip` (đảo chiều theo nhịp) từ stage 5 → `burst` (bò chậm rồi vọt) từ stage 7 — chuyển tốc độ luôn **mượt hoá**, không snap
- **Dao cắm sẵn**: 0 (2 stage đầu) → tối đa 3, rải cách nhau ≥0.5 rad; **ngọc cách dao ≥0.36 rad** để ăn ngọc không chết oan
- **TRÙM mỗi 5 stage**: sigil sọ quỷ đỏ, thêm dao, pattern dữ hơn, thưởng +10
- Va chạm **nghiêm ngặt nhưng công bằng**: ngưỡng góc 0.105 rad ≈ đúng bề rộng lưỡi dao nhìn thấy; thời điểm chạm **nội suy sub-frame** (không phụ thuộc 60/90/120 Hz)

### Trợ giúp người chơi
- **Tutorial tương tác 3 bước, không thể thua** (phóng dao → né dao cắm sẵn (dao nảy ra + nhắc thử lại) → phá totem) — **tự chạy ở lần mở đầu tiên**, skip được
- Cue **"CHẠM!"** nhấp nháy ở bước tutorial đầu

### UI (không menu — mở game vào thẳng gameplay)
- **Không có màn menu**: lần đầu mở = tutorial, các lần sau vào thẳng ván (yêu cầu chuẩn chung)
- **Header chuẩn chung**: nút **Back** (quit) trái · pill **SCORE ★** + pill **BEST ♛** (vàng) giữa · nút **Volume** phải; dòng nhỏ dưới header chỉ hiện **BOSS** (không có bộ đếm stage/level)
- Popup kết quả **tự vẽ theo popup-common**: một tiêu đề đổi chữ `New Best!` / `Game Over`, SCORE trắng / BEST vàng, nút Play Again amber (bắn `retry_level`), `fitScores()` chống tràn điểm to
- Pause vẽ trên canvas ("Chạm để chơi tiếp") — `timeMs` đóng băng nên **toàn bộ thế giới tự đứng im**, tap đầu chỉ resume không phóng dao

### Âm thanh (WebAudio synthesize 100%, không file ngoài)
- **BGM dark & epic**: bed dây **saw detune** qua lowpass 1500Hz + **drone chủ âm A** giữ suốt + màu **Neapolitan ♭II** (Am→Dm→B♭→E), không arpeggio — 1 swell/hợp âm + timpani mềm khi đổi hợp âm, ~6.5s/hợp âm; lookahead scheduler chống dồn nốt khi resume
- **10 SFX** qua bus `sfxOut` riêng (cân bằng SFX↔BGM, chống clip): phóng (swish hẹp dải êm + sine glide — hành động lặp nhiều KHÔNG dùng noise burst thô) / cắm (thunk) / ngọc (chime) / sát nút / va dao (clash kim loại + jingle thua) / totem vỡ (boom + arpeggio) / TRÙM (tritone) / tutorial xong / nút bấm
- Mute persist qua `save_data`; suspend khi pause/app nền; unlock ở chạm đầu (autoplay policy)

---

## Kỹ thuật

- **1 vòng rAF, dt clamp [0,100]ms**, mọi anim/timer key theo `timeMs` (đóng băng khi pause — không lệch đồng hồ)
- **Sprite pre-render** (dao ×2 biến thể, totem mỗi stage, wisp nền) — **không shadowBlur/gradient per-frame** (đắt trên WebView), không `backdrop-filter`
- **Không gian logic** LW=720 (dọc) / LH=1180 (ngang) — hình học totem/dao tự co theo màn, DPR cap 3 (nền cap 1.5)
- Va chạm góc: `normAng`/`angDiff` + nội suy `prevRot→rot` tại frame chạm vành
- Mảnh vỡ/dao văng **thừa hưởng vận tốc tiếp tuyến + góc quay** của totem (motion continuity, không snap)
- `?reset=1` xoá save khi playtest; mở browser thường tự log bridge message ra console

---

## Tuân thủ quy ước chung

- ✅ **game-common.md** — game **không level**: `sendMessage` 5 trường với `level: null`; `game_result` (`result: null`, `showModal: false`) khi thua; `retry_level` khi bấm Play Again; `quit`/Back đúng SVG chuẩn; `ads` mỗi 3 ván thua; `statusBarHeight` (URL ưu tiên + env safe-area +8px); `data`/`language` window ưu tiên; `waitForNativeInjection` 200ms; `onAppPause` (pause thật + suspend audio) / `onNextLevel` (no-op) / `onRetryLevel`
- ✅ **Lưu 2 tầng (§1.2)** — `save_data` chỉ data nhẹ `{best, tut, muted}`, mirror localStorage, load merge MAX; best lưu **ngay khi vượt**, HUD Best chỉ refresh khi hết ván
- ✅ **popup-common.md** — game điểm cao tự vẽ popup: 1 tiêu đề đổi chữ (không badge riêng), màu cố định (nhãn `#8fb4ff`, BEST `#ffd23f`, Play Again `#ffb02e`), ẩn/hiện bằng opacity + scale, delay 0.7s, `fitScores()`
- ✅ **zip-common.md** — single-file, `game.json` đủ field, 3 cover đúng tên/kích thước (1920×1080 / 800×1200 / 800×800)
- ✅ **i18n** — inline `I18N` **đủ 23 ngôn ngữ** cho toàn bộ UI + tutorial + how-to-play + aria-label; Google Sans Flex, ngôn ngữ ngoài Latin tự chuyển `system-ui` (cả chữ vẽ canvas); tiếng Ả Rập bỏ letter-spacing

> Không có event `victory`/`next_level` — game điểm cao không level (đúng spec §1.1, không phải deviation).

---

## 📋 Backlog

- [ ] Skin dao (mở khóa bằng điểm/ngọc tích lũy) như Knife Hit gốc
- [ ] Pattern nâng cao: totem thu nhỏ/phình to, 2 vòng đồng tâm quay ngược chiều
- [ ] Daily challenge (seed theo ngày — sẵn hạ tầng stageConfig deterministic)
- [ ] Haptic feedback qua bridge khi cắm dao/va chạm (cần native hỗ trợ)
