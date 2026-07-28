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
- **Điểm tích luỹ**: mỗi phô mai +1 (tính cả **miếng gốc** — vào ván đã là 1),
  **Perfect +5 và cộng dồn theo chuỗi**, vàng +3, né mốc +3, sập tháp trừ đúng số
  tầng mất. Không còn cơ chế xu, không có đồng hồ đếm độ cao riêng — chỉ **một
  con số** cho tới lúc **Game Over**. Chi tiết ở mục **Điểm**.
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

#### "Giật giật" — nguyên nhân THẬT: camera đứt vận tốc (2026-07-28)

Sau khi đã sửa render (DPR/alpha/gradient) và nhịp (WAIT/jitter) mà QA **vẫn**
báo giật, đo lại bằng cách bọc `drawHam` ghi đúng toạ độ **màn hình** mỗi frame
(bot chơi 30–40 tầng). Kết quả: **không có frame nào nhảy > 20px** — nên đây
KHÔNG phải "nhảy hình". Nhưng bước lớn nhất cả ván là **15.1px/frame, xuất hiện
ở ĐÚNG mọi lần `jump→idle`** (lần đáp nào cũng có, không sót).

Thủ phạm: mục tiêu camera bám **đại lượng RỜI RẠC**.

- `camTargetOff` = mặt tháp, mà `tower.length` +1 **ngay khoảnh khắc ghim miếng**
  → mục tiêu nhảy trọn 1 tầng (92px) trong đúng 1 frame. Camera ease 12%/frame
  nên hamster bị đẩy ~11px chỉ trong frame đó rồi mới trôi về.
- Vị trí vẫn liền (C0) nhưng **vận tốc đứt (C1)**: đang rơi ~4px/frame bỗng
  thành 15px/frame. Mắt đọc ra đúng thành một cú "khựng" — và vì nó bám vào
  hành động lặp lại nhiều nhất (đáp), cảm giác là "giật giật" suốt ván.
- `camTargetX` = `topCx()` đứt y hệt: lúc ghim, "miếng trên cùng" đổi sang miếng
  vừa đáp, lệch tới ±nửa miếng → càng đáp lệch càng nặng (đúng kiểu người chơi
  thật, không phải bot).

Sửa: cho camera bám **đại lượng LIÊN TỤC**.

- `camFocusY()` — khi hamster đang bay, tiêu điểm đi trước theo độ cao của nó
  nhưng **kẹp tối đa 1 tầng**. Frame cuối trước khi ghim `h` vẫn > `BLOCK_H` nên
  kẹp = `BLOCK_H`; ghim xong mặt tháp cao thêm đúng `BLOCK_H` → **hai bên bằng
  nhau**, hết bước nhảy. Bonus: camera dâng mượt theo cú nhảy.
- `camTargetX()` bám **hamster** thay vì tâm miếng trên cùng — lúc ghim `ham.x`
  không hề đổi nên liên tục tuyệt đối. Khung hình gần như y cũ vì hamster luôn
  đứng trên miếng trên cùng (lệch tối đa nửa miếng).

| Bước lớn nhất mỗi frame | Trước | Sau |
|---|---|---|
| Lúc đáp (`jump→idle`) | **15.1 px** | **4.6 px** |
| Trục ngang lúc đáp | 7.5 px (camera) | **0 px do camera** |
| Lúc bật nhảy (`idle→jump`) | 12.0 px | 10.6 px |

Sau khi sửa, **mọi** bước ngang còn lại đều quy được về cơ chế **trượt băng**
(`slideVx` khớp đúng `dx`, `topKind=ice`) — không còn bước nào do camera. Bước
lúc bật nhảy là **lực bật của cú nhảy, cố ý giữ** (đứng yên → bật lên phải dứt
khoát); đó cũng là lý do nó không đọc thành lỗi.

#### Siết 2 cơ chế "dễ dãi" + luật sinh miếng (QA 2026-07-28 đợt 2)

**1. Vùng đáp — hết cảnh "teleport lên mép".** Cũ chỉ cần `h ≤ BLOCK_H` là ghim,
mà `h` chạy từ 92 xuống 0 → chân có thể đã tụt sâu **cạnh bên** miếng (h=30 ⇒
thấp hơn mặt miếng 62px) rồi mới ghim, ghim xong hamster bị đặt lên **mặt**
miếng ⇒ **nhảy vọt 62px trong 1 frame**. Thêm 2 chốt:

- `LAND_SNAP_MAX = 22` — chân tụt quá 22px dưới mặt miếng thì không còn tính là
  "đáp lên". (1 frame @30Hz tụt tối đa ~18px nên vẫn đủ chỗ ở mọi tần số quét.)
- `LAND_EDGE_INSET = 12` — tâm hamster phải vào **trong** mép miếng ≥12px, không
  được đứng đúng cạnh (cũ `< w/2` cho phép tâm nằm ngay mép).

Đáp sát mép (còn trong vùng hợp lệ) → nhá chữ nhỏ **"Close one!"** cạnh hamster.

**2. Chồng tháp — hết cảnh mép-chồng-mép.** Hai tầng phải **ăn nhau ≥ 34% bề
rộng miếng hẹp hơn**; hụt thì **sập 3 tầng** (điểm giảm đúng 3) kèm chữ
`Slipped! -3`. `Math.min(3, tower.length - 1)` luôn chừa tầng gốc nên **tháp thấp
hơn 3 tự về đúng tầng 1**, không cần xử lý riêng. `missPenalty` (sập 5) nay dùng
chung hàm `collapseTower()` — trước là 2 bản logic sao chép, dễ lệch nhau.

**3. Sau LÒ XO cấm VÀNG và MINI.** Cú nhảy kế đã cao & lâu hơn 35%; chồng thêm
miếng chạy nhanh hoặc miếng hẹp là bắt canh 2 biến số lạ cùng lúc.

**4. Miếng VÀNG hạ tốc ở tầng cao.** Tốc nền đã tự tăng theo tầng, nhân thêm
`×1.25` cố định thì tầng cao thành gắt: trần cũ `MAX_SPEED×1.25 = 13.1px/frame`,
vượt xa trần miếng thường 10.5. Nay hệ số **giảm dần 1.25 → 1.08 từ tầng 15 đến
tầng 30** (`goldMul()`), trần cao nhất còn **11.34**.

**5. Cue `Double cheese!`** đặt **chính giữa** (`spawnCue(0, …)`) vì 2 miếng vào
từ **cả hai phía**, không có "một bên" để chỉ vào. Cả 3 cue nay hiện **1000ms**.

**Verify** (đo lại trên **bản mentor đã tối ưu hiệu năng**, sau khi áp lại 5 mục
trên): bot chơi 9000 frame ở ±0 và ±3 frame lệch nhịp — **0 lỗi JS**, xếp
119–125 tầng, **0 ván thua**; bước dọc lúc đáp **≤7.2px/frame** (hết teleport);
`Close one!` bắn 1–4×, `Double cheese!` bắn 6–11×, `Quick!`/`Watch out!` đều bắn;
**0 miếng vàng/mini nào ra ngay sau lò xo**; vàng ở tầng ≥15 đo được **max đúng
11.34** = `MAX_SPEED × 1.08`, tức taper đang có hiệu lực (trần cũ 13.1).

Nhánh chồng hụt bot **không tự chạm tới** (`ham.x` gần như đứng yên nên bot luôn
đáp quanh tâm) → **test tay 3 ca**: lệch 40px → không phạt (tháp 10→11); lệch
100px → sập đúng 3 (10→8, `Slipped! -3`, 3 miếng rơi, **state vẫn `playing`** —
không phải thua); tháp 2 tầng chồng hụt → **về đúng tầng 1**.

> ℹ️ Thực tế chồng hụt hầu như chỉ xảy ra **sau miếng băng** (chỉ khi đó `ham.x`
> mới dịch đủ xa). Nếu QA thấy nó gần như không bao giờ kích hoạt thì nới
> `MIN_STACK_FRAC` lên cao hơn 0.34.

#### Độc lập TẦN SỐ QUÉT — đo lại sau khi đổi camera (2026-07-28)

Chạy bot với đồng hồ tổng hợp bước `1000/hz` ms mỗi frame, cùng 25 tầng:

| Hz | frames | nhịp/tầng | bước lúc đáp |
|---|---|---|---|
| 30 | 1172 | 1551 ms | 521 px/s |
| 60 | 2195 | 1455 ms | 431 px/s |
| 90 | 3396 | 1497 ms | 401 px/s |
| 120 | 3960 | 1519 ms | 525 px/s |
| 144 | 3630 | 1474 ms | 194 px/s |

**Nhịp chơi lệch ±3.3%** quanh ~1500ms ở mọi Hz → tốc độ ván chơi không đổi theo
màn hình. `camFocusY()` mới cũng độc lập Hz **theo thiết kế**: nó là hàm thuần
của `timeMs`, và mốc kẹp `BLOCK_H` trùng đúng điều kiện ghim (`h > BLOCK_H` thì
bỏ qua) nên hai vế bằng nhau ở mọi bước thời gian. 0 lỗi JS ở cả 5 mức.

⚠️ **Còn 1 điểm phụ thuộc Hz — không phải bug, nhưng nên biết**: `onTap()` gán
`ham.t0 = timeMs`, mà `timeMs` chỉ nhích 1 lần/frame → **thời điểm bấm bị làm
tròn về frame**. Cửa sổ Perfect ~44ms nên ở 60Hz sai số làm tròn 16.7ms (chấp
nhận được), nhưng ở 30Hz lên 33ms → Perfect khó hơn hẳn. Vị trí ĐÁP thì đã nội
suy dưới khung hình (`overMs`) nên không lệch. Muốn công bằng tuyệt đối thì lấy
`event.timeStamp` để lùi mốc bấm về đúng lúc chạm — chưa làm vì máy đích tối
thiểu 60Hz.

#### Cue cảnh báo miếng khó (2026-07-27) — `spawnCue()`

Miếng vàng chạy ×1.25 và miếng mốc chạm là thua, cả hai đều hay úp sọt. Nay ngay
lúc miếng sinh, game nhá một chữ cảnh báo tại **mép nó sắp trôi vào**, **nghiêng
trái 15°**, sống **820ms**. Miếng lọt vào khung sau ~410–520ms nên chữ hiện
**TRƯỚC khi thấy miếng**, rồi còn nán lại ~300ms cùng miếng mới mờ hẳn.

| Miếng | Chữ | Màu | Chỗ đặt |
|---|---|---|---|
| ✨ Vàng | `Quick!` | `BLOCK_SKINS.gold.body` | **ngang** làn trượt |
| 🦠 Mốc | `Watch out!` | `BLOCK_SKINS.mold.body` | **DƯỚI** làn bay (mốc bay cao hơn mặt tháp `MOLD_LIFT`) |

Hai chi tiết dễ sai, đã xử lý:

- **Ghim vào trong mép**: miếng sinh ở tận ±459px trong khi nửa khung chỉ 360px
  → đặt chữ đúng toạ độ sinh là nằm ngoài màn hình, không ai thấy.
- **Inset theo bề rộng THẬT** (`ctx.measureText`), tối thiểu 80px: `"Watch out!"`
  dài gần gấp đôi `"Quick!"` — để inset cứng 80px thì mép trái chữ rơi ra ngoài
  khung (`cueLeft=-2`). Nay tự nới lên 96px → `cueLeft=14`.

Chữ nổi (`floatTexts`) được bổ sung 3 thuộc tính tuỳ chọn: `life` (đời riêng,
mặc định `FLOAT_MS` 800ms), `rot` (radian) và `rise` (độ trôi lên).

Verify headless (ép sinh từng loại rồi lấy `canvas.toDataURL`): cả 2 cue đúng
`rotDeg=-15.0`, `life=820`; vàng ở `screenX=80`, mốc ở `96` (vào từ trái) /
`624` (vào từ phải), **nằm trọn trong khung** cả 2 phía; cue mốc có
`cueBaseline=221 > blockBottom=184` = đúng **bên dưới** miếng; và lúc chụp miếng
vẫn **`onScreen=false`** — cue hiện trước thật. 0 lỗi JS.
- **Tap / Space** → hamster nhảy thẳng lên (~0.61s); miếng trượt qua bên dưới và
  **dừng ngay chỗ hamster đáp lên** → thành tầng mới (**+1 điểm**; miếng **nhún &
  nghiêng** về phía đáp cho đã tay). Đáp lệch thì tháp xiêu vẹo (không cắt miếng,
  không đổ); đáp sát mép (<30px) → loạng choạng 1s.
- Lệch ≤ ~7px → **Perfect** (vòng sáng + chữ nổi kèm số điểm) → **+5, chuỗi liên
  tiếp cộng dồn +5/+10/+15/+20…** (xem mục Điểm).
- **Nhảy vọt qua** (không ai đáp) → KHÔNG thua: miếng văng đi kéo **sập 5 tầng**
  (điểm tụt theo số tầng mất, đứt chuỗi), hamster rơi xuống đáp tầng thứ 6.
- **Thua** khi bị phô mai **xô ngã** (nhảy quá trễ / rơi lại quá sớm — kể cả đang
  bay) hoặc **đâm đầu vào phô mai mốc** → bung dù rơi xuống, camera pan-out trọn
  tháp, popup kết quả.

## Miếng đặc biệt

| Loại | Mở từ tầng | Hiệu ứng |
|---|---|---|
| ✨ Vàng | 5 | Chạy nhanh hơn — hệ số **×1.25 ở tầng thấp, giảm dần về ×1.08 từ tầng 15→30** (tầng cao tốc nền đã nhanh sẵn). **Báo trước:** nhá chữ **"Quick!"** (vàng, nghiêng trái 15°, 1000ms) ngay tại mép nó sắp trôi vào. **Không ra ngay sau lò xo** |
| 🪀 Lò xo | 6 | Cú nhảy **kế tiếp** cao & lâu hơn 35% (báo bằng ▲) |
| 🐭 Mini | 8 | Miếng **hẹp ~60%** — khó canh đáp hơn. **Không ra ngay sau lò xo** |
| 🧊 Băng | 12 | Đáp xong **trượt mạnh theo đà** (ma sát nhỏ) — dễ tuột tới mép, loạng choạng |
| 🦠 Mốc | 15 | Bay ở **làn cao** — đứng yên cho nó qua (an toàn), nhảy trúng = thua. **Báo trước:** nhá **"Watch out!"** (xanh mốc, nghiêng trái 15°, 1000ms) **BÊN DƯỚI** làn nó sắp bay vào |
| 👯 Double | 25 | ~9% round: 2 miếng từ 2 phía so le — đáp 1, miếng kia tự rơi. **Báo trước:** **"Double cheese!"** đặt **chính giữa** (vào từ cả 2 phía), 1000ms |

## Điểm (đổi theo QA 2026-07-28)

| Việc | Điểm |
|---|---|
| Xếp được 1 phô mai | **+1** |
| **Perfect** | **+5**, chuỗi liên tiếp **CỘNG DỒN**: +5 → +10 → +15 → +20 … (Perfect thứ *n* = **+5×n**) |
| Đáp trúng miếng **vàng** | **+3** |
| **Né được** miếng mốc | **+3** |
| Sập tháp (nhảy vọt −5 tầng / chồng hụt −3 tầng) | **−đúng số tầng mất**, không xuống dưới 0 |

Chuỗi Perfect đứt (một lần đáp không Perfect) thì lần Perfect kế **về lại +5**.

⚠️ **Đây là đổi CẤU TRÚC, không phải chỉnh số.** Trước đây `score = tower.length`
— một **giá trị dẫn xuất**, nên về nguyên tắc không cộng thưởng được. Nay `score`
là **bộ tích luỹ** (`addScore()` / `setScore()`), kéo theo:

- **Snapshot ván dở phải lưu `score`** — không còn suy lại được từ số tầng.
  Snapshot cũ (chưa có trường này) tự lùi về luật cũ nên **không mất ván dở**.
- Điểm **to hơn hẳn**: bot chơi 122 tầng ra **707 điểm** (luật cũ là 122) ≈ **×5.8**
  → mọi `best` cũ sẽ bị phá ngay ván đầu. Đó là hệ quả tất nhiên của việc đổi luật.

`best` là điểm cao nhất từng đạt, persist ngay khi vượt. `?reset=1` xoá kỷ lục.

**Verify** (dựng thế cờ tay vì bot không ép được chuỗi Perfect dài theo ý):
chuỗi 4 Perfect cộng đúng **6, 11, 16, 21** (= 1+5, 1+10, 1+15, 1+20); đứt chuỗi
→ lần lệch chỉ +1, Perfect kế **về đúng +6**; vàng Perfect **+9** (1+5+3), vàng
không Perfect **+4** (1+3); sập 5 tầng từ 100 → **95**, từ 2 → **0** (không âm);
snapshot lưu đúng `score=137`. Chơi thật 122 tầng: 0 lỗi JS, chữ nổi hiện đúng
`Perfect! +5` → `×2 +10` → … → `×5 +25`, `✓ +3` (né mốc), `+3` (vàng).

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
