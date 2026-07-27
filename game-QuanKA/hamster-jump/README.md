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
  đặt) chơi **tutorial hướng dẫn**: một miếng trôi tới gần rồi **FREEZE**, hiện
  "Chạm để nhảy" → chạm → hamster nhảy, miếng canh đúng nhịp để **luôn Perfect** →
  hiện lời khen. Lặp **3 lần** rồi "Chúc chơi vui vẻ!"; **chạm lần nữa** vào ván
  thật (giữ nguyên tháp, bắt đầu chơi — KHÔNG reset). Tutorial không thể thua.
- **Header chuẩn**: Back (trái) · pill **SCORE** (★) + **BEST** (👑) ở giữa ·
  nút **Volume** bật/tắt âm thanh (phải).
- **Điểm = SỐ PHÔ MAI trong tháp**, tính cả **miếng gốc** dưới đáy (vào ván đã là
  1). Không còn cơ chế xu, không còn đồng hồ đếm độ cao riêng — chỉ **một con số**
  đếm số phô mai xếp được cho tới lúc **Game Over** (ground cheese cũng được tính).
- Miếng phô mai **trượt vào trên ván gỗ** ở đúng tầm mặt tháp (hướng ngẫu nhiên;
  nhanh dần theo độ cao, nền 6.9 → trần 10.5px/frame). **Khởi đầu dịu**: 3 tầng
  đầu chạy ×0.75, tăng dần lên tốc chuẩn (×1.0) ở tầng 20.

#### 🔧 Nhịp chơi — sửa theo QA 2026-07-27

*"tốc độ ra thanh gỗ và phô mai ra hơi chậm nên cảm giác giật giật ko giữ đc nhịp chơi"*,
làm rõ thêm: ***"thanh gỗ RA nhanh, còn phô mai giữ nguyên TỐC ĐỘ trước đó. Và
randomize tốc độ phô mai hẹp lại để giữ được nhịp chơi."***

Nên đây là vấn đề **TIẾT TẤU**, không phải vận tốc, cũng không phải render
(khựng do render đã sửa đợt trước). **VẬN TỐC TRƯỢT GIỮ NGUYÊN** — chỉ sửa lúc
*ra* và độ *ngẫu nhiên*:

| | Cũ | Mới | Vì sao |
|---|---|---|---|
| `WAIT_MS` (miếng đứng im chờ) | 430 ×(0.8–1.4) = **344–602ms** | **260 ×(0.85–1.15) = 221–299ms** | cắt hơn nửa khoảng "không có gì chuyển động" |
| `BOARD_MS` (ván gỗ thò ra) | 260 | **175** | ván bung dứt khoát |
| jitter tốc độ | ×0.85–1.20 (**biên 41%**) | ×0.92–1.08 (**biên 17%**) | cũ **cố ý** đánh lạc nhịp → không bắt được beat; nay nhịp **đọc được**. Tâm jitter vẫn 1.0 → **tốc độ trung bình không đổi** |
| `BASE_SPEED`, `speedRamp`, cửa sổ Perfect | — | **giữ nguyên** | QA chốt: phô mai giữ tốc độ cũ |

**Đo bằng bot headless** — RAF shim 60fps ảo, bot canh tap tối ưu, **15 tầng đầu
từ tháp mới**, chạy lại y hệt với bộ hằng cũ để so:

| Chỉ số | Cũ | Mới |
|---|---|---|
| Nhịp mỗi tầng | 1744 ms (**1167–3267**) | **1502 ms** (1167–1767) |
| Khoảng chết giữa 2 tầng | 468 ms | **266 ms** |
| Cửa sổ canh đáp | 327 ms | 353 ms *(không hẹp đi — tốc độ giữ nguyên)* |
| Perfect | 8/15 | **11/15** |

Điểm mấu chốt đúng thứ QA phàn nàn: **độ dao động nhịp** từ **1167–3267 ms
(chênh 2.8×)** xuống **1167–1767 ms (chênh 1.51×)** — hết "giật giật", giữ được
nhịp, mà **không hề làm game nhanh/khó hơn**. 0 lỗi JS.
- **Tap / Space** → hamster nhảy thẳng lên (~0.61s); miếng trượt qua bên dưới và
  **dừng ngay chỗ hamster đáp lên** → thành tầng mới (**+1 điểm**; miếng **nhún &
  nghiêng** về phía đáp cho đã tay). Đáp lệch thì tháp xiêu vẹo (không cắt miếng,
  không đổ); đáp sát mép (<30px) → loạng choạng 1s.
- Lệch ≤ ~7px → **Perfect** (vòng sáng + chuỗi "Perfect ×n" dưới điểm) — chỉ là
  hiệu ứng/độ khéo, **không cộng thêm điểm** (điểm chỉ đếm số phô mai).
- **Nhảy vọt qua** (không ai đáp) → KHÔNG thua: miếng văng đi kéo **sập 5 tầng**
  (điểm tụt theo số tầng mất, đứt chuỗi), hamster rơi xuống đáp tầng thứ 6.
- **Thua** khi bị phô mai **xô ngã** (nhảy quá trễ / rơi lại quá sớm — kể cả đang
  bay) hoặc **đâm đầu vào phô mai mốc** → bung dù rơi xuống, camera pan-out trọn
  tháp, popup kết quả.

## Miếng đặc biệt

| Loại | Mở từ tầng | Hiệu ứng |
|---|---|---|
| ✨ Vàng | 5 | Chạy **nhanh ×1.25** — thử phản xạ (không thưởng riêng) |
| 🪀 Lò xo | 6 | Cú nhảy **kế tiếp** cao & lâu hơn 35% (báo bằng ▲) |
| 🐭 Mini | 8 | Miếng **hẹp ~60%** — khó canh đáp hơn |
| 🧊 Băng | 12 | Đáp xong **trượt mạnh theo đà** (ma sát nhỏ) — dễ tuột tới mép, loạng choạng |
| 🦠 Mốc | 15 | Bay ở **làn cao** — đứng yên cho nó qua (an toàn), nhảy trúng = thua |
| 👯 Double | 25 | ~9% round: 2 miếng từ 2 phía so le — đáp 1, miếng kia tự rơi |

## Điểm

Điểm **= số phô mai đang có trong tháp** (kể cả miếng gốc). Mỗi lần đáp thêm một
miếng → **+1**; nhảy hụt làm sập tầng → điểm **giảm đúng số tầng mất**. `best`
(kỷ lục) là điểm cao nhất từng đạt, persist ngay khi vượt. `?reset=1` xoá kỷ lục
khi chơi thử.

## Âm thanh

WebAudio synth 100% inline (không file ngoài), thiết kế **ấm & mềm chống chói**:
mọi giọng đi qua lowpass riêng → bus → **lowpass master 2.8kHz + highpass 100Hz +
compressor** trước khi ra loa (không bao giờ có sóng saw/square thô hay noise rít).

- **Nhạc nền**: polka "toy-chip" **vui nhộn** (C trưởng, ~128 BPM) — bass oom-pah +
  giai điệu music-box hỏi/đáp, 3 biến tấu xoay vòng (~45s) để đỡ nhàm; peak nhỏ,
  luôn nằm dưới SFX; scheduler lookahead resume-safe.
- **SFX**: nhảy & đáp **dùng chung một tiếng "pop" tự nhiên mềm**; lò xo,
  Perfect (ngũ cung dâng theo chuỗi, trần ≤988Hz), né mốc, loạng choạng,
  **mất 5 tầng** (chuỗi 5 nốt rơi dần + "whump"), xô ngã, bung dù, kết quả
  (best/thua), nút bấm, nhắc tutorial.

Mở khoá ở cử chỉ đầu tiên (autoplay policy); cờ `mute` lưu trong save_data; tự
tắt tiếng khi app xuống nền. Đã đo offline: **không clip** (kể cả khi 4 tiếng to
chồng nhau) và **năng lượng > 4kHz ≈ 0** (không gắt tai).

## Tuân thủ quy ước

- **game-common.md** — `sendMessage` 5 trường, `level: null`; `game_result`
  (`showModal:false`) · `retry_level` · `quit` · `ads` (mỗi 3 ván) · `save_data`
  (`{best, tut, mute}`, persist ngay khi đổi + mirror localStorage, boot lấy max);
  đọc `statusBarHeight`/`data`/`language` đúng thứ tự ưu tiên; callback native
  định nghĩa sớm; nút Back SVG chuẩn; font Google Sans Flex (+fallback ngoài
  Latinh, cả canvas); reset CSS + tắt tap-highlight.
- **popup-common.md** — game điểm cao tự vẽ popup: 1 tiêu đề `New Best!`/`Game
  Over`, màu accent cố định, `fitScores()`, delay 700ms có guard `clearTimeout`.
- **zip-common.md** — single-file, `game.json` đủ field, 3 cover đúng tên/kích thước.
- **i18n** — inline; **chỉ TUTORIAL dịch đủ 23 ngôn ngữ** (theo QA 2026-07-22:
  "thừa localize"); SCORE/BEST/popup… dùng tiếng Anh qua fallback `en` —
  deviation chủ đích so với game-common §5.

## Backlog

- [ ] Quest daily + login streak (localStorage)
- [ ] Revive/gift bằng quảng cáo — chờ app hỗ trợ callback ad (bridge `ads` hiện một chiều)
