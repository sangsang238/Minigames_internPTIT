# Hamster Jump — QuanKA

> Game canh thời điểm **nhảy** xếp tháp phô mai | HTML5 single-file | Endless (không level)

| | |
|---|---|
| **Package** | `com.falcon.hamsterjump` |
| **Engine / Version** | HTML5 · 1.0.0 |
| **Category** | ARCADE |
| **Entry** | `hamster-jump/index.html` |

## Cách chơi

- **Không có menu** — mở game là **vào thẳng gameplay**. Lần đầu (cờ `tut` chưa
  đặt) chơi qua tutorial 3 lần đáp có gợi ý (không thua, không tính điểm) rồi
  **chảy thẳng vào ván thật** (giữ nguyên tháp, bắt đầu tính điểm — KHÔNG reset).
- **Header chuẩn**: Back (trái) · pill **SCORE** (★) + **BEST** (👑) ở giữa ·
  nút **Volume** bật/tắt âm thanh (phải). Ngay dưới: **độ cao tháp** (icon 2 phô
  mai chồng) + số **xu**.
- Miếng phô mai **trượt vào trên ván gỗ** ở đúng tầm mặt tháp (hướng, tốc độ và
  nhịp chờ ngẫu nhiên; nhanh dần theo độ cao, trần 10.5px/frame). **Khởi đầu dịu**:
  3 tầng đầu chạy ×0.75, tăng dần lên tốc chuẩn (×1.0) ở tầng 20.
- **Tap / Space** → hamster nhảy thẳng lên (~0.61s); miếng trượt qua bên dưới và
  **dừng ngay chỗ hamster đáp lên** → thành tầng mới (miếng **nhún & nghiêng** về
  phía đáp cho đã tay). Đáp lệch thì tháp xiêu vẹo (không cắt miếng, không đổ);
  đáp sát mép (<30px) → loạng choạng 1s.
- Kỷ lục **tháp cao nhất** (`bh`) vẫn được lưu trong save_data (dùng cho thống kê).
- Lệch ≤ ~7px → **Perfect** (vòng sáng, chuỗi hiện "Perfect ×n" dưới điểm).
- **Nhảy vọt qua** (không ai đáp) → KHÔNG thua: miếng văng đi kéo **sập 5 tầng**
  (−5 điểm, đứt chuỗi), hamster rơi xuống đáp tầng thứ 6.
- **Thua** khi bị phô mai **xô ngã** (nhảy quá trễ / rơi lại quá sớm — kể cả đang
  bay) hoặc **đâm đầu vào phô mai mốc** → bung dù rơi xuống, camera pan-out trọn
  tháp, popup kết quả.

## Miếng đặc biệt

| Loại | Mở từ tầng | Hiệu ứng |
|---|---|---|
| ✨ Vàng | 5 | Nhanh ×1.25 — đáp được **+8 xu** |
| 🪀 Lò xo | 6 | Cú nhảy **kế tiếp** cao & lâu hơn 35% (báo bằng ▲) |
| 🐭 Mini | 8 | Rộng ~60% — **điểm ×2** |
| 🧊 Băng | 12 | Đáp xong **trượt mạnh theo đà** (ma sát nhỏ) — dễ tuột tới mép, loạng choạng |
| 🦠 Mốc | 15 | Bay ở **làn cao** — đứng yên cho nó qua **+2 điểm**, nhảy trúng = thua |
| 🎁 Hộp quà | ~12 | Mỗi 12–20 tầng một hộp — **+3 xu** |
| 👯 Double | 25 | ~9% round: 2 miếng từ 2 phía so le — đáp 1, miếng kia tự rơi |

## Điểm & Xu

| Sự kiện | Thưởng |
|---|---|
| Đáp thường / mini | +1 / +2 |
| Perfect thứ n liên tiếp | **+5×n** (5→10→15…; mini ×2) |
| Perfect chuỗi ≥ 2 | +1 xu |
| Né phô mai mốc | +2 |
| Xu rơi từ trời (nhảy chạm) | +1 xu (18% ra +3) |

Xu tích lũy vĩnh viễn (backlog: shop skin). `?reset=1` xoá kỷ lục + xu khi chơi thử.

## Âm thanh

WebAudio synth 100% inline (không file ngoài), thiết kế **ấm & mềm chống chói**:
mọi giọng đi qua lowpass riêng → bus → **lowpass master 2.8kHz + highpass 100Hz +
compressor** trước khi ra loa (không bao giờ có sóng saw/square thô hay noise rít).

- **Nhạc nền**: polka "toy-chip" **vui nhộn** (C trưởng, ~128 BPM) — bass oom-pah +
  giai điệu music-box hỏi/đáp, 3 biến tấu xoay vòng (~45s) để đỡ nhàm; peak nhỏ,
  luôn nằm dưới SFX; scheduler lookahead resume-safe.
- **SFX**: nhảy & đáp **dùng chung một tiếng "pop" tự nhiên mềm**; xu (ding dịu),
  lò xo, Perfect (ngũ cung dâng theo chuỗi, trần ≤988Hz), né mốc, loạng choạng,
  **mất 5 tầng** (chuỗi 5 nốt rơi dần + "whump"), xô ngã, bung dù, kết quả
  (best/thua), nút bấm, nhắc tutorial.

Mở khoá ở cử chỉ đầu tiên (autoplay policy); cờ `mute` lưu trong save_data; tự
tắt tiếng khi app xuống nền. Đã đo offline: **không clip** (kể cả khi 4 tiếng to
chồng nhau) và **năng lượng > 4kHz ≈ 0** (không gắt tai).

## Tuân thủ quy ước

- **game-common.md** — `sendMessage` 5 trường, `level: null`; `game_result`
  (`showModal:false`) · `retry_level` · `quit` · `ads` (mỗi 3 ván) · `save_data`
  (`{best, coins}`, persist ngay khi đổi + mirror localStorage, boot lấy max);
  đọc `statusBarHeight`/`data`/`language` đúng thứ tự ưu tiên; callback native
  định nghĩa sớm; nút Back SVG chuẩn; font Google Sans Flex (+fallback ngoài
  Latinh, cả canvas); reset CSS + tắt tap-highlight.
- **popup-common.md** — game điểm cao tự vẽ popup: 1 tiêu đề `New Best!`/`Game
  Over`, màu accent cố định, `fitScores()`, delay 700ms có guard `clearTimeout`.
- **zip-common.md** — single-file, `game.json` đủ field, 3 cover đúng tên/kích thước.
- **i18n** — inline đủ 23 ngôn ngữ, fallback `en`; đã xử lý RLM tiếng Ả Rập +
  NBSP tiếng Pháp.

## Backlog

- [ ] Shop skin hamster/phô mai dùng xu
- [ ] Quest daily + login streak (localStorage)
- [ ] Revive/gift bằng quảng cáo — chờ app hỗ trợ callback ad (bridge `ads` hiện một chiều)
