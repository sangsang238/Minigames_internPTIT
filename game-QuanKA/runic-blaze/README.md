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

### 🔧 Text nổi: bỏ nền, thêm outline + "đá chữ cũ lên" (2026-07-28, mentor)

Mentor bác cách làm chip nền của đợt trước: *"tại sao em lại làm viền với nền
vậy… txt combo thì mình ko làm nền nhá, xấu lắm — em chỉ cần cho chữ TO HƠN với
THỜI GIAN TỒN TẠI LÂU HƠN thôi."*

- **Bỏ hẳn nền + viền + bo góc + padding** ở mọi text nổi trong màn: combo badge,
  score-fly, nhãn special (`Blaze!/Nova!/Prism!`), toast. Chữ to hơn bù lại:
  combo **32→38px**, score-fly **15→17px**, nhãn special **17→19px**.
- **Outline mảnh thay cho nền**, màu = **đúng màu viền cũ**: nhãn special lấy màu
  rune (đỏ/lam/lục/hổ phách/tím/hồng). Làm bằng `text-shadow` 8 hướng chứ không
  phải `-webkit-text-stroke` — stroke của WebKit vẽ **đè lên mặt chữ** làm nét
  mảnh đi, `text-shadow` nằm phía sau nên giữ nguyên độ dày.
- **KHÔNG đụng** SCORE/BEST, popup Game Over, và thanh hướng dẫn tutorial (đó là
  panel chữ 2 dòng, bỏ nền là không đọc nổi trên board).

**Chữ mới ĐÁ chữ cũ lên** — mentor: *"toàn bộ chữ combo xuất hiện từ dưới lên;
nếu chữ mới xuất hiện, nó sẽ có hiệu ứng đá chữ cũ lên trên… chữ cũ tồn tại nốt
thời gian còn lại rồi mất."* Cũ là `dodgeOverlap`: **chữ mới** tự né lên trên chữ
cũ → chuỗi dài thì chữ mới bị đẩy mỗi lúc một cao, rời khỏi chỗ vừa nổ, mà vẫn đè
nhau khi 2 chữ ra gần cùng lúc. Nay ngược lại: chữ mới **luôn hiện đúng chỗ của
nó**, chữ cũ bị đẩy lên 34px nhường chỗ, tuổi thọ giữ nguyên.

### 🔧 Vòng 2 (QA 2026-07-28, sau khi bỏ nền)

- **Câu khen nhỏ lại** 38 → **30px** ("hơi to").
- **CHỒNG câu khen, chữ mới đá chữ cũ**: trước đây câu khen là **một** phần tử bị
  ghi đè nên chuỗi combo dài chỉ thấy đúng câu cuối. Nay mỗi combo sinh **một
  câu mới**; câu cũ bị đá lên **40px**, **nhỏ lại** (×0.84 mỗi bậc, sàn 0.55) và
  **mờ đi** (−0.26 mỗi bậc), sống nốt tuổi thọ của nó rồi tự mất — đúng
  *"great đá nice lên trên… cái nào xuất hiện sau thì đá cái trước"*. Dựng 2 lớp
  lồng nhau vì cả "đá lên" lẫn "bung–giữ–mờ" đều cần `transform`: lớp ngoài cho
  JS đá (transition), lớp trong cho animation.
- **Outline hết lấm lem**: cách cũ chồng **8 bản `text-shadow`** lệch 1px — chỗ
  giao thì đậm, khe giữa thì hở nên nhìn nhoè. Nay `-webkit-text-stroke` +
  **`paint-order: stroke fill`** cho nét liền mạch mà **không** làm mảnh chữ
  (mặc định WebKit vẽ stroke đè lên mặt chữ; `paint-order` đẩy nó ra sau). Có
  `@supports not (paint-order: stroke)` lùi về text-shadow 4 hướng cho WebView cũ.
- **`Blaze Storm!`**: bỏ nền, đặt ở **góc trên-trái của BOARD** (theo rect board
  thật, không phải góc màn hình), **thấp xuống 1 ô** (`+cellPx`) và **nghiêng
  30° sang trái**.
  > ⚠️ **Phép xoay ban đầu KHÔNG hề chạy trong game thật.** Mọi keyframe của
  > `resultPop` đều đặt `transform`, mà **animation ghi đè khai báo thường** →
  > `transform` tĩnh của `.corner` bị nuốt sạch. Đo được: `animationName =
  > resultPop`, computed `transform = matrix(0.5,0,0,0.5,…)` ⇒ **góc xoay 0°**.
  > Sửa bằng keyframes riêng `cornerPop` mang sẵn `rotate(-30deg)` trong TỪNG
  > khung, chọn qua `#result-banner.corner.show` (id + 2 class nên thắng
  > `#result-banner.show`). Đo lại: `animationName = cornerPop`, **góc xoay −30°**;
  > banner Game Over vẫn `resultPop`, **0°** — không bị ảnh hưởng.
  >
  > **Bài học kiểm thử:** ảnh chụp trước đó thấy có xoay là do harness chèn
  > `animation: none !important`. Ảnh đó chỉ chứng minh **rule CSS tồn tại**,
  > KHÔNG chứng minh nó sống sót khi animation chạy. Với thuộc tính mà animation
  > cũng đụng tới (`transform`/`opacity`), phải đo `getComputedStyle` trên trang
  > THẬT (animation đang chạy), hoặc chụp bằng `animation-delay` âm để nhảy tới
  > khung cuối mà vẫn giữ animation sống.
- **Màu SÁNG không được làm outline cho chữ trắng.** `✦ Arcane Surge!` dùng
  `#ffd23f` → viền vàng dày quanh chữ trắng đọc ra thành *"chữ trắng nền vàng"*,
  chói và khó nhìn. Nay xét **độ sáng cảm nhận** (Rec. 601, ngưỡng 150) để tự
  đảo vai: màu sáng → cho vào **CHỮ**, outline lấy nền tối; màu tối/đậm (Blaze
  đỏ, Cursed đỏ máu…) giữ chữ trắng + outline màu như cũ. Quy tắc chung nên
  không phải chữa từng ca.
- **Chỉ CÂU KHEN MỚI mới đá câu khen cũ.** Trước đây `spawnScoreFly` và
  `spawnTileLabel` cũng gọi `kickPopsUp()`, nên câu khen **tự nhảy lên dù không
  có câu khen nào mới**, và mỗi combo bị đá 2 lần ⇒ **khoảng cách gấp đôi** —
  đúng 2 điều QA phàn nàn. Nay `kickPopsUp(comboOnly)` tách 2 nhóm: câu khen chỉ
  bị đá bởi câu khen, chữ điểm/nhãn chỉ bị đá bởi chữ điểm/nhãn.
- **Khoảng cách gần lại**: câu khen `40 → 30px`, chữ điểm/nhãn `34 → 24px`. Câu
  đã bị đá thì **ẩn dòng `COMBO ×n`**, chỉ còn chữ khen — khối thấp lại (~33px)
  nên 30px vẫn không chồng chữ.
- **Hắc ấn nổ → popup**: cắt một nửa, `160+340+700 ≈ **1.2s**` (trước ~2.4s).
- **Tự động Play Again** — *lỗi thật, đã sửa*: `devSimulateNative` tự gọi
  `onRetryLevel()` sau 2.5s mỗi khi nhận `game_result`. Game này gửi
  `showModal:false` và **tự vẽ popup + nút Play Again**, nên app thật KHÔNG bao
  giờ gọi lại — đoạn mô phỏng đó tạo ra hành vi không có thật và làm popup tự
  đóng. Đã bỏ. **Verify:** sau khi thua 8s → `onRetryLevel` gọi **0 lần**, popup
  vẫn hiện, `levelEpoch` không đổi.

### 🔧 Death screen dài hơn + hạ tải lúc chuỗi dài

- **Death screen**: hết nước đi **0.95s → 2.3s**; hắc ấn nổ **~1.36s → ~2.4s**
  (riêng transition bóng tối đã 1s, trước đây popup ập lên lúc cảnh nổ còn đang
  chạy). Hai hằng `DEATH_HOLD_MS` / `BOOM_HOLD_MS` để chỉnh nhanh.
- **Lag lúc combo liên tục**: hạt tròn là loại đông nhất (trần 420). Cũ mỗi hạt
  tốn `fillStyle` + `beginPath`+`arc`+`fill`; nay dùng **sprite pha sẵn theo màu**
  → còn **1 `drawImage`**. Đo được `fx.drawImage` 11.8/frame ⇒ dưới code cũ chỗ
  đó là ~35.4 lệnh path, tức tổng FX **~50 → 26.6 lệnh/frame**; càng chạm trần
  particle thì càng lợi.
  > Đã **loại trừ** 2 nghi can khác bằng số đo, không phải đoán: `morphTile` chỉ
  > 9 lần và `querySelector` 0.01/frame trong cả phiên → **không phải** thủ phạm.

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

## Tối ưu hiệu năng (2026-07-28) — soi theo đúng bộ tiêu chí mentor dùng cho hamster-jump

Đo bằng **đếm lệnh**, không đoán từ đọc code. (`performance.now()` vô dụng dưới
`--virtual-time-budget` — nó đứng yên trong mỗi frame nên mọi phép đo ms ra
`0.0000`; đơn vị đo đúng ở đây là **số lệnh canvas** và **số lần ép layout**.)

| | Trước | Sau |
|---|---|---|
| Lệnh canvas của nền, mỗi frame | **135** (40 `beginPath` + 40 `arc` + 40 `fill` + 14 `drawImage` + 1 `clearRect`) | **27.5** |
| → quy ra mỗi giây | ~8 100 | **~1 650** |
| Ép layout đồng bộ, mỗi nước đi | **7.8** | **0.6** |

**1. Nền — vẽ lại thứ gần như không đổi, 60 lần/giây.** Mỗi ngôi sao tốn trọn bộ
`beginPath`+`arc`+`fill` + 1 lần đổi `fillStyle`, chỉ để vẽ một chấm 1–2px trôi
**≤0.17px mỗi frame**. Sửa 2 lớp:
- **4 sprite sao pha sẵn màu** → mỗi sao còn **đúng 1 `drawImage`**, không còn
  đổi `fillStyle` lần nào (trước là 40 lần/frame).
- **Nền chạy ~30fps thay vì 60** (`BG_STEP_MS = 33`): sao trôi ≤10px/s nên 33ms
  mới nhích 0.33px — mắt không phân biệt được. Frame bỏ qua thì **giữ nguyên
  canvas, không `clearRect`** → không nháy. `dt` được gộp lại nên tốc độ trôi
  thực **không đổi**.

**2. Bắt trình duyệt layout đúng lúc cần mượt nhất.** Rect của board là hằng số
giữa 2 lần layout, nhưng **mỗi** hiệu ứng (score-fly, nhãn special, vòng nổ,
tia…) lại gọi `getBoundingClientRect` → **ép layout đồng bộ cả trang, đúng lúc
cascade đang chạy**. Nay đo 1 lần rồi dùng lại (`boardRect()`), huỷ cache khi
layout đổi. Lúc màn đang **rung** thì vẫn đo thật nhưng **không lưu**, tránh
cache dính vĩnh viễn giá trị lệch theo `transform`.

**3. Lớp phủ `.hidden` vẫn được composite.** `opacity: 0` **không** gỡ lớp phủ
khỏi cây composite — `.overlay` và `#pause-veil` phủ **kín màn hình** vẫn được
ghép lại mỗi frame suốt lúc chơi. Thêm `visibility: hidden` trễ đúng bằng thời
gian fade → tắt hẳn sau khi mờ xong, mà vẫn giữ hiệu ứng fade.

**4. CSS trùng lặp & rule chết.** Gộp `.tile.selected .tile-inner` (khai báo 2
khối liền nhau); xoá 6 rule chỉ còn trong CSS, không nơi nào dùng — sót từ v2 khi
còn menu/how-to/bộ đếm lượt: `.btn3d.mini`, `.restart-ico`, `.home-ico`,
`.help-ico`, `.ico-diamond`, `.moves-danger` + `@keyframes dangerPulse`.

**Kiểm chứng không làm hỏng gì:** cache rect lệch **0.00px** so với số đo thật và
đo lại đúng sau `invalidate`; `overlay.hidden` cho `visibility: hidden`; nền trích
ra PNG vẫn đủ sao tím + tàn lửa cam như cũ; tutorial 3 bước vẫn chạy trọn (B2 rèn
1 special, B3 phá Hắc Ấn 1→0); combo badge + Hắc Ấn nguyên vẹn. **0 lỗi JS.**

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
