# Hamster Jump — QuanKA

> Game canh thời điểm **nhảy** xếp tháp phô mai | HTML5 single-file | Endless (không level)

| | |
|---|---|
| **Package** | `com.falcon.hamsterjump` |
| **Engine / Version** | HTML5 · 1.0.0 |
| **Category** | ARCADE |
| **Entry** | `hamster-jump/index.html` |

## Cách chơi

- **Menu**: nút Play + Tutorial; icon hình thoi-trong-tròn (góc phải) mở panel
  tóm tắt luật & các loại phô mai. **Lần đầu mở game** bấm Play sẽ vào tutorial
  3 lần đáp có gợi ý (không thua, không tính điểm; cờ `tut` lưu trong save_data).
- Miếng phô mai **trượt vào trên ván gỗ** ở đúng tầm mặt tháp (hướng, tốc độ và
  nhịp chờ ngẫu nhiên; nhanh dần theo độ cao, trần 10.5px/frame).
- **Tap / Space** → hamster nhảy thẳng lên (~0.61s); miếng trượt qua bên dưới và
  **dừng ngay chỗ hamster đáp lên** → thành tầng mới. Đáp lệch thì tháp xiêu vẹo
  (không cắt miếng, không đổ); đáp sát mép (<30px) → loạng choạng 1s.
- Lệch ≤ ~7px → **Perfect** (vòng sáng, chuỗi hiện "Perfect ×n" dưới điểm).
- **Nhảy vọt qua** (không ai đáp) → KHÔNG thua: miếng văng đi kéo **sập 5 tầng**
  (−5 điểm, đứt chuỗi), hamster rơi xuống đáp tầng thứ 6.
- **Thua** khi bị phô mai **xô ngã** (nhảy quá trễ / rơi lại quá sớm — kể cả đang
  bay) hoặc **đâm đầu vào phô mai mốc** → bung dù rơi xuống, camera pan-out trọn
  tháp, popup kết quả.

## Miếng đặc biệt

| Loại | Mở từ tầng | Hiệu ứng |
|---|---|---|
| ✨ Vàng | 5 | Nhanh ×1.45 — đáp được **+8 xu** |
| 🪀 Lò xo | 6 | Cú nhảy **kế tiếp** cao & lâu hơn 35% (báo bằng ▲) |
| 🐭 Mini | 8 | Rộng ~60% — **điểm ×2** |
| 🧊 Băng | 12 | Đáp xong **trượt theo đà**, tới mép thì loạng choạng |
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

- [ ] Sound FX (Web Audio): nhảy, đáp, Perfect, ngã
- [ ] Shop skin hamster/phô mai dùng xu
- [ ] Quest daily + login streak (localStorage)
- [ ] Revive/gift bằng quảng cáo — chờ app hỗ trợ callback ad (bridge `ads` hiện một chiều)
