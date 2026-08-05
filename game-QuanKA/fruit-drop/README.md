# Fruit Drop — QuanKA

> **Onet / Pikachu connect CÓ TRỌNG LỰC** | HTML5 single-file | Level vô hạn sinh theo công thức

## Thông tin game

| Trường | Giá trị |
|---|---|
| **Package** | `com.falcon.fruitdrop` |
| **Engine / Version** | HTML5 single-file · 1.0.0 |
| **Category** | PUZZLE |
| **Loại** | Game **có level** — vô hạn, thắng/thua theo màn |
| **Kích thước** | `index.html` ~120 KB (đã gồm 20 KB sprite icon) |

## Gameplay

- Nối 2 tile **cùng loại quả** bằng đường ≤ 3 đoạn thẳng (**≤ 2 lần rẽ**) qua ô
  trống; đường được vòng ra ngoài biên (vành ngoài 1 ô luôn đi được).
- **KHÁC animal-connect: có TRỌNG LỰC.** Sau mỗi lần nối, mọi tile nằm trên ô
  trống trong cùng cột **rơi xuống** lấp chỗ — bàn tự sắp xếp lại sau *mỗi* nước.
- **Sạch bàn trước khi hết giờ = thắng màn** → sang màn kế; **hết giờ = THUA =
  endgame** (run kết thúc, chơi lại từ màn 1, giữ BEST).
- **Điểm CỘNG DỒN cả run** (lvl 1 → endgame). SCORE = điểm run hiện tại,
  BEST = điểm run cao nhất.
- **Điểm chỉ tăng KHI NGÔI SAO CHẠM ô SCORE**, không phải lúc nối xong
  (yêu cầu 2026-08-04). Sao dùng đúng path/màu của icon trên ô SCORE, nên thứ
  bay lên và chỗ nó đáp xuống đọc ra cùng một ý; số nhảy đúng nhịp từng sao đáp
  (nối được 30 điểm với 3 sao → HUD đi 0 → 20 → 30).
  - Bên trong tách đôi: `score` là **sự thật**, cộng ngay lúc nối — save, BEST và
    payload win/lose đều đọc nó, nên **bị kill giữa chừng không mất điểm**.
    `scoreShown` mới là thứ HUD hiển thị. Bất biến `scoreShown + scorePending
    === score` được kiểm sau **mỗi** nước trong bot test.
  - **Cộng điểm KHÔNG đợi `transitionend`** (sửa 2026-08-04, phát hiện từ
    playtest: khung SCORE nảy trễ "một tích tắc" so với lúc sao có vẻ đã tới).
    Nguyên nhân đo được bằng chính công thức bezier: đường bay dùng
    `cubic-bezier(.45,0,.3,1)` — một đường **giảm tốc mạnh về cuối**, tới
    **90% thời lượng đã đi được 99.3% quãng đường** (10% cuối chỉ dịch <1%,
    dưới 1px, mắt không thấy). Đợi `transitionend` (100%) để cộng điểm nghĩa
    là luôn "ngồi không" qua cái đuôi chết đó — đúng cảm giác trễ mà mentor
    chỉ ra. Nay **tách rời** hai việc: `credit()` (cộng điểm + nảy pill) bắn ở
    `FLY_MS * FLY_ARRIVE_FRAC` (0.9) tính từ lúc transform THẬT SỰ bắt đầu
    (trong `kick()`, sau 2 rAF) — còn `remove()` (dọn phần tử) vẫn đợi
    `transitionend`/timer dự phòng như cũ nên ngôi sao không bao giờ biến mất
    giữa chừng. `remove()` gọi lại `credit()` như lưới an toàn (idempotent)
    phòng khi `creditTimer` bị throttle lúc tab ẩn. Đo bằng test: cộng điểm ở
    đúng 558ms/620ms (=0.9), sao vẫn còn trên DOM tới hết bay.
  - **Throb của ô SCORE bung ngay frame đầu** (sửa 2026-08-04, lần 2 —
    playtest vẫn thấy trễ *sau khi* đã sửa thời điểm cộng điểm). Thủ phạm là
    chính đường easing chứ không phải lúc gọi: WAAPI hiểu `easing` trong
    options là easing của **cả iteration**, nên `steps(2,end)` lượng tử hoá
    tiến độ thành {0, .5, 1} → ứng với keyframe `scale(1)` / `scale(1.07)` /
    `scale(1)`. Nghĩa là ô SCORE **giữ nguyên `scale(1)` suốt 130ms đầu của
    260ms** rồi mới bung — đo trực tiếp bằng cách kéo `Animation.currentTime`
    và đọc computed style. Nay đỉnh nằm ở **keyframe offset 0** và mỗi giá trị
    được **giữ phẳng** tới stop của nó (các khe 1% là ramp dưới một frame,
    không phải tween nhìn thấy được): vẫn 3 nấc chunky đúng style `steps()`,
    nhưng phản hồi ngay frame đầu. Đo lại: **130ms → 0ms** trước khi có thay
    đổi nhìn thấy, và về đúng `scale(1)` khi kết thúc.
  - Thắng/thua gọi `updateScoreHud()` để chốt: sao còn đang bay **không bao giờ**
    giữ lại điểm trên bảng kết quả.
- **Cơ chế điểm đặc trưng — DROP COMBO**: nếu lần nối kế tiếp dùng ít nhất một
  tile **vừa rơi** ở lần trước thì combo tăng một bậc (×1.5 → ×2 → ×2.5, trần
  ×4) và hiện tem `DROP ×N`. Trọng lực vì thế là **động cơ tính điểm**, không
  phải hiệu ứng trang trí — nó thưởng đúng kỹ năng mà cơ chế này thêm vào:
  đọc bàn *sau khi* nó sập xuống.
- **Đồng hồ ẩn thưởng nối nhanh**: <1.1s → "Lightning!", <2.2s → "Fast!"
  (cộng vào trước khi nhân hệ số combo).
- **~5s không thao tác** → tự nháy một cặp nối được (auto-hint, vòng xanh dương).
  Hết nước đi → **tự xáo miễn phí** kèm toast.
- **Không có menu** — mở app vào thẳng gameplay. Lần đầu chơi tự vào **tutorial**.
- **Header chuẩn**: Back (trái) · SCORE ★ · BEST ♛ (2 pill bằng nhau, BEST vàng)
  · Volume (phải); dưới là **LEVEL | thanh giờ | số giây**.
- **SFX tổng hợp Web Audio** (không file ngoài, không nhạc nền), qua một bus
  `sfxOut`, mềm/tròn, năng lượng 250–3000 Hz. Tiếng rơi dùng **modal synthesis**
  (noise qua bandpass) và **phát 1 lần mỗi lần lắng**, không phải mỗi tile —
  không thì một cú sập dây chuyền nghe như súng máy.

### Tutorial — 1 bàn 6×4, không chuyển cảnh

4 bước, **mục tiêu từng bước được tìm trên bàn THẬT lúc chạy** (không hard-code
toạ độ), nên không bao giờ có bước yêu cầu thứ mà bàn không có:

1. **LINK** — nối cặp đang sáng (chọn cặp dọc có ≥2 tile phía trên để cú rơi thấy rõ).
2. **DROP** — sau cú rơi, game tự tìm **cặp do chính cú rơi tạo ra** và bắt dọn nó.
3. **BEND** — chạm cặp cùng loại **bị chặn**; game vẽ đường "quá vòng" **đúng 3
   lần rẽ** (đỏ nét đứt + chấm ở mỗi chỗ rẽ) cho thấy vì sao không nối được.
4. **CLEAR** — dọn nốt bàn, hiện **timer demo** + mũi tên chỉ lên nó (timer chỉ
   chạy sau thao tác đầu tiên, và **không bao giờ gây thua**).

`buildTutorialBoard()` sinh lại bàn tới khi **cả 4 bài học đều dạy được** và
greedy-solve (có trọng lực) còn **0 ô** — verify bằng máy, không phải bằng mắt.
Tutorial **không tính điểm** (score/best giữ 0), có Skip, `?reset=1` xem lại.

### Level sinh theo công thức — `levelConfig(idx)`

**Bàn to lên ở MỌI màn** từ L1 tới L12 (yêu cầu 2026-08-04), rồi chốt và
nhường việc cho đồng hồ:

| Level | Bàn | Ô | Cặp | Loại quả | Giờ | Giây/cặp |
|---|---|---|---|---|---|---|
| 1 | 6×6 | 36 | 18 | 6 | 80s | 4.44 |
| 2 | 8×6 | 48 | 24 | 8 | 100s | 4.17 |
| 3 | 8×8 | 64 | 32 | 8 | 130s | 4.06 |
| 4 | 10×8 | 80 | 40 | 10 | 155s | 3.88 |
| 5 | 10×10 | 100 | 50 | 10 | 190s | 3.80 |
| 6 | 12×10 | 120 | 60 | 12 | 215s | 3.58 |
| 7 | 12×11 | 132 | 66 | 12 | 225s | 3.41 |
| 8 | 12×12 | 144 | 72 | 12 | 235s | 3.26 |
| 9 | 12×13 | 156 | 78 | 12 | 245s | 3.14 |
| 10 | 12×14 | 168 | 84 | 12 | 250s | 2.98 |
| 11 | 12×15 | 180 | 90 | 12 | 250s | 2.78 |
| 12 | 12×16 | 192 | 96 | 12 | 255s | 2.66 |
| 13+ | 12×16 | 192 | 96 | 12 | 240s → **giảm đều** → 190s (L16+) | 2.50 → 1.98 |

- **Chỉ HÀNG mọc thêm sau 12×10, không phải cột** — vì **cột mới là thứ màn hình
  giới hạn**. Trên máy 360px bàn 12 cột đã cho tile 25px; 14 cột sẽ tụt xuống
  ~22px, dưới ngưỡng mà bộ sprite được đo là còn phân biệt được (chính vì ngưỡng
  này mà `strawberry` bị loại khỏi bộ). Hàng thì gần như miễn phí.
- **Mọc thêm hàng KHÔNG làm tile nhỏ đi chút nào.** Chiều rộng vốn đã là ràng
  buộc (`availW/(COLS+1)` < `availH/(ROWS+1)` trên mọi máy đo), nên 12×16 cho
  đúng cỡ tile như 12×10: 22px @320px, 25px @360px, 29px @412px. Đã verify bàn
  lớn nhất **vừa cả iPhone SE 320×568** và vẫn giữ được vành ngoài 1 ô cho
  đường vòng biên (không chạm sàn 16px). Công thức mô hình được **hiệu chuẩn
  đối chiếu với `cellPx` thật** (model 35 = thực tế 35) chứ không phải phỏng đoán.
- Giờ tính **theo CẶP** nên bàn to tự được thêm giờ — to ra làm màn **dài hơn**
  chứ không lập tức bóp nghẹt; thứ siết là hệ số `perPair`, và nó **tiếp tục
  siết sau khi bàn đã ngừng lớn**. Vẫn là "một biến chạy một chiều tại một
  thời điểm", chỉ khác là điểm bàn giao dời từ L6 sang L12.
- Đánh đổi đã đo, không phải đoán — xem bảng ở mục dưới: mọc tới 16 hàng tốn
  khoảng **5 điểm tỉ lệ deadlock** (9.8% → 14.5% ở lối chơi đối-nghịch).
- **Chỉ 12 loại quả trên một bàn, nhưng cửa sổ 12-loại XOAY theo level** nên cả
  **14 icon** đều xuất hiện trong một run — có đa dạng mà không trả giá deadlock.
- Vô hạn → **không có màn cuối, không bắn `victory`** (deviation có chủ ý so với
  game-common §1.5, giống animal-connect / runic-blaze).

### ⚠️ Phát hiện quan trọng: trọng lực làm DEADLOCK NHIỀU HƠN, không ít hơn

Giả định ban đầu (ghi trong plan) là trọng lực sẽ *giảm* deadlock. **Đo bằng
simulator thì ngược lại** — và đây là thứ định hình toàn bộ bảng level ở trên.

**Nguyên nhân cấu trúc:** Onet tĩnh, dọn một cặp để lại một **lỗ ở giữa bàn**, và
chính những cái lỗ đó là hành lang đi đường cho các cặp sau. Có trọng lực, mọi
lỗ **dồn hết lên đỉnh**, để lại một khối đặc mà chỉ **bề mặt** của nó là nối
được. Kết quả đo (chơi ngẫu-nhiên-đối-nghịch, qua đúng generator/pathfinder của
bản ship):

| Đòn bẩy | Kết quả đo |
|---|---|
| Thiết kế ban đầu (10×12, 14 loại) | **37–52%** ván bị kẹt ít nhất 1 lần |
| Số loại quả (cùng 1 bàn) | 6 loại **5%** · 10 loại 12.5% · 14 loại **37.5%** |
| Tỉ lệ khung (cùng số ô + số loại) | **12×10 → 7.5%** · **10×12 → 20%** |
| Mọc thêm hàng (12 loại, N=400/cỡ) | 12×10 **9.8%** ±2.9 · 12×12 **11.5%** ±3.1 · 12×14 **14.0%** ±3.4 · 12×16 **14.5%** ±3.5 |
| Bớt loại quả để mua lại (12×16) | 12 loại 14.5% · 10 loại **12.3%** · 8 loại **4.8%** |
| Hệ số kề `f` | Gần như không đổi deadlock, chỉ làm bàn "lộ cặp" hơn → giữ **thấp (0.15)** |

**Đo lại 2026-08-04 khi cho bàn to dần:** mọc thêm hàng đi **ngược** hướng tốt
của quy luật khung (bàn cao dần), nên deadlock tăng **9.8% → 14.5%**. Giả định
ban đầu của em là "nhiều cặp mỗi loại hơn sẽ bù lại" — **sai**, hiệu ứng khung
lấn át. Lần đo đầu N=120 còn cho ra một số 5.8% kẹp giữa hai số ~20%, tức nhiễu
chứ không phải tín hiệu; phải nâng lên **N=400 kèm khoảng tin cậy 95%** mới đọc
được đường cong thật. Vẫn nhận 14.5% vì đó là ~4× tốt hơn khi chơi thật, tức
**1 lần auto-xáo mỗi ~27 màn** thay vì ~30 — rẻ so với việc có ramp. Nếu cần
mua lại, bớt số loại quả là đòn bẩy sẵn sàng (bảng trên), nhưng cửa sổ 12 loại
được giữ vì đa dạng mỗi bàn đáng giá hơn mấy điểm đó.

→ Hai đòn bẩy thật là **KHUNG (rộng thắng cao)** và **SỐ LOẠI** (giữ ~3+ cặp mỗi
loại trên bàn), **không** phải hệ số kề. Sau khi sửa: **trung bình 0.111 lần xáo
mỗi màn** (3–16% tuỳ màn), **0 bàn không giải được**, adjacency ~34%. Đây là số
*đối-nghịch*; animal-connect đo chơi thường tốt hơn ~4×, tức người chơi thật gặp
auto-xáo khoảng **1 lần mỗi ~30 màn** — đúng nghĩa lưới an toàn.

## Nghệ thuật — nguồn stock, KHÔNG generate

- **Icon: Fluent Emoji "Flat" (Microsoft) — license MIT.** 14 quả/rau vector
  32×32, không viền, **không gradient/filter/defs** (đã grep xác nhận) nên render
  cực rẻ. Gộp thành **một sprite `<symbol>` 20 KB**; mỗi tile là một `<use>`.
  Nghĩa vụ license duy nhất là giữ notice — đã để trong comment đầu `index.html`.
- **Bộ 14 quả chốt bằng ảnh chụp thật ở cỡ tile thật**, không chọn bằng cảm tính:
  - `strawberry` bị loại — ở 30px nó và `apple` đều là đốm đỏ tròn, không phân biệt được.
  - `blueberries` (ứng viên thay) cũng bị loại — nó **đụng `grapes`** (cùng là chùm
    quả mọng tím), sửa được lỗi này lại đẻ ra lỗi tương đương.
  - → chọn **`mushroom`**: silhouette mũ-vòm-cộng-thân là duy nhất trong bộ, và là
    món hai tông đỏ/kem duy nhất.
- **Phong cách: neo-brutalist in ấn PHẲNG HOÀN TOÀN** — nền giấy kem + chấm
  halftone, viền mực đen 3px, khối màu phẳng, mọi animation dùng `steps()`.
  **Không có bóng đổ nổi/3D ở bất kỳ đâu** (theo yêu cầu 2026-08-04: bỏ hết UI
  kiểu nút 3D) — nút/pill/khung bàn/banner chỉ còn viền mực; nút bấm phản hồi
  bằng thu nhỏ + đổi nền, không phải "lún vào bóng". Các `box-shadow` còn lại
  đều là **vòng offset 0** (chọn / gợi ý), không phải bóng đổ.
  **Không blur, không `filter`, không `backdrop-filter`, không gradient màu** —
  `radial-gradient` duy nhất trong file là hoạ tiết **chấm halftone** (một
  pattern lặp, compositor vẽ một lần rồi thôi), không phải chuyển sắc.
- **Trạng thái chọn cố tình KHÔNG dùng màu**: 14 quả 14 màu nên bất kỳ vòng accent
  nào cũng đụng một quả — vòng đỏ tomato trên tile táo từng gộp thành một khối đỏ
  đặc không đọc được. Nay chọn = **phóng to + viền mực dày**, đọc được trên mọi
  icon. Hint dùng **xanh dương** vì trong bộ **không có quả nào màu xanh dương**.

## Tối ưu hiệu năng

| Hạng mục | Cách làm |
|---|---|
| **Không có vòng lặp rAF** | Nền là CSS, tile rơi là transition của compositor, link fade bằng CSS → **không có gì cần 60Hz**. Việc định kỳ duy nhất là đồng hồ ở **5Hz** (`setInterval` 200ms). Đo bằng máy: lúc idle **0 lần gọi `requestAnimationFrame`**. |
| **`will-change` chỉ đặt lúc THẬT SỰ động** | Trước đây nằm trên `.tile`, tức **cả 120 tile** của bàn 12×10 bị đẩy lên **layer compositor riêng suốt cả màn** — một hoá đơn GPU-memory thường trực để tăng tốc thứ gần như luôn đứng yên. Nay chỉ ở `.tile.falling`. Đo: **120 → 0** tile được promote lúc nghỉ, vẫn có `transform` khi rơi. |
| **Thanh giờ chạy bằng `transform`** | Trước là transition `width` → **layout + paint mỗi frame**, và vì tick 200ms so với transition 250ms nên nó **không nghỉ một giây nào** trong cả màn. Đổi sang `scaleX()` + `transform-origin:left` → thuần compositor, mượt ở 120/144Hz. |
| **Không ghi DOM thừa** | `renderTimer` chạy 5 lần/giây và trước đây ghi lại thanh giờ, 3 class list và text giây **dù không đổi gì** — text giây thực tế chỉ đổi 1 lần/giây, tức 4/5 lần ghi là vô ích. Nay mọi lần ghi đều có guard; đo: **20 lần render thừa → 0 DOM write**. |
| **Không đo lại layout mỗi nước** | `spawnFlyers` gọi `getBoundingClientRect()` cho bàn + ô SCORE mỗi lần nối → ép **layout đồng bộ cả trang**. Nay cache (`flyGeom`), chỉ xoá khi resize / banner tutorial dịch bàn. Nhịp đập ô SCORE đổi sang WAAPI nên bỏ luôn `void offsetWidth` ở tối đa 6 lần sao đáp. |
| **Chặn layout ở bàn cờ** | `contain: layout style` trên `#board`. Cố ý **không** thêm `paint`: `.selected` / `.pop` phóng to mặt tile ra ngoài ô nên paint containment sẽ cắt cụt. |
| **Kiến bò (tutorial) bớt repaint** | `stroke-dashoffset` từng chạy `linear infinite` → repaint SVG **mỗi frame** suốt lúc demo cặp bị chặn (60/s, hoặc 144/s trên panel nhanh) chỉ để dịch 28px. Đổi `steps(16,end)`: nhìn như cũ, rẻ hơn ~4–9×. |
| **Ẩn tab thì dừng hẳn** | Trước đây đưa WebView xuống nền vẫn để ticker 5Hz + audio context chạy. Nay `visibilitychange` → `pauseGame()`: dừng cả hai, và cũng là hành xử đúng — đồng hồ không nên tiếp tục trôi khi người chơi không nhìn thấy bàn. |
| **Nền không tốn gì** | Nền **chỉ còn chấm halftone** vẽ một lần (các khối tròn/vuông trôi đã bỏ theo yêu cầu 2026-08-04). Không canvas, không animation nền → animal-connect phải tối ưu vòng vẽ nền xuống 14 lệnh canvas/frame; ở đây là **0**, và giờ không còn cả element nào động. |
| **Tile rơi** | Đổi `transform` + `transition` → compositor lo. **Không một dòng JS nào chạy mỗi frame trong lúc rơi.** |
| **Tìm cặp** | BFS **theo nguồn**: một lần quét tia đánh dấu mọi tile nối được từ một nguồn → nhóm k tile cùng loại tốn **k** lần quét thay vì **k(k−1)/2** lần tìm đường. Thoát sớm + cache cặp tìm được cho auto-hint. |
| **Lối tắt "cặp chạm nhau"** | Hai tile **kề nhau cùng loại LUÔN nối được** (đoạn thẳng 0 lần rẽ, không có ô nào ở giữa). Nên câu hỏi mà `hasMoves()` thật sự hỏi — "còn nước đi không?" — thường trả lời được bằng **một lượt quét O(ô), không BFS nào**. Trọng lực liên tục xếp lại cột nên loại cặp này gần như luôn có: đo được **97.5%** số lần gọi trúng lối tắt. Cố ý **không** dùng cho `randomize` (đường của gợi ý) để gợi ý không bị đơn điệu — gợi ý chỉ chạy 5s/lần nên không đáng tối ưu. |
| **Chạm tile = O(1)** | Mỗi element giữ sẵn back-reference `__tile` tới ô pool của nó (gán một lần trong `buildPool`). Trước đây handler click **duyệt tuyến tính cả pool** so sánh node — trên bàn 12×16 là tới **192 phép so sánh cho mỗi lần chạm**. |
| **Cấp phát** | `Int32Array` cấp **một lần mỗi level**, **đóng dấu bằng `scanId`** thay vì `fill(0)` → một lần quét không tốn gì cho việc dọn buffer. |
| **DOM tile** | Pool cấp sẵn ở cỡ bàn max (120), tái dùng vĩnh viễn; đổi loại = 1 `setAttribute`. Không tạo/huỷ node lúc chơi. |
| **`backdrop-filter`** | **Không dùng ở bất kỳ đâu** (bị ban vì perf trên WebView; style phẳng cũng không cần). |
| **Bridge** | `save_data` từ **~260 → 40 message**/phiên: điểm cộng dồn nên qua kỷ lục cũ là *mỗi* nước đều lập best mới. Nay localStorage ghi ngay (miễn phí, trong WebView) còn native bị **throttle 4s + trailing send**; hết màn/hết run vẫn gửi ngay không throttle. |

### Đo đường nóng mỗi nước (bàn 12×16 = 192 tile)

Đo bằng **số phép tính**, không phải đồng hồ: dưới `--virtual-time-budget` của
Edge headless, `performance.now()` **không nhích trong lúc chạy code đồng bộ**
(hiệu chuẩn: vòng lặp 3 triệu bước đọc ra `0.00ms`), nên mọi con số thời gian ở
môi trường này là vô nghĩa. Số đếm thì tất định và so sánh được.

| Mỗi nước đi | Trước | Sau | Giảm |
|---|---|---|---|
| `reachFrom` (BFS) | 71.5 | **1.1** | −98% |
| `ray()` | 1881 | **44** | −98% |
| Ô được ray duyệt | 2682 | **71** | −97% |

Con số 71.5 lần BFS mỗi nước là thứ khiến em đi tìm: `findAnyPair` phải vét cạn
vài "bucket" loại quả bằng BFS trước khi tình cờ gặp cặp nối được. Lối tắt cặp
chạm nhau xoá gần hết phần đó.

**Đã kiểm thứ KHÔNG phải thủ phạm:** ghi `localStorage` sau mỗi nước chỉ tốn
**544 byte, 1 lần `setItem`/nước** (0.58 KB/nước) — nhỏ, nên em để nguyên; đổi
nó sẽ đánh đổi khả năng chống mất tiến độ khi app bị kill mà chẳng được bao
nhiêu.

## Tuân thủ quy ước chung

- ✅ **game-common.md** — `sendMessage` đủ 5 trường, level 1-indexed;
  `game_result` win/lose + `showModal:true`; Back → `quit`; `ads` mỗi 3 ván;
  `save_data` chỉ data nhẹ, ván dở lưu `localStorage`; đọc đủ
  `statusBarHeight`/`currentLevel`/`data`/`language` đúng thứ tự ưu tiên;
  `waitForNativeInjection`; 3 callback native; font Google Sans Flex qua
  `--ui-font` (ngoài Latinh → `system-ui`); reset CSS + tắt tap-highlight.
- ✅ **popup-common.md** — game theo level nên **không tự vẽ popup kết quả**
  (app lo, `showModal:true`).
- ✅ **zip-common.md** — single-file, `game.json` đủ field & pass script check,
  3 cover đúng **1920×1080 / 800×1200 / 800×800** (verify từ byte 16–23 của PNG).
- ✅ **i18n** — inline; **chỉ TUTORIAL dịch đủ 23 ngôn ngữ** (theo tiền lệ
  animal-connect / QA "thừa localize"), HUD & toast dùng tiếng Anh qua fallback
  `en` — deviation chủ đích so với game-common §5.
- ℹ️ `button-common.md` / `background-common.md` được các spec khác tham chiếu
  nhưng **không tồn tại trong repo** (git history xác nhận chưa từng có) →
  `btn3d` và pattern nền lấy từ runic-blaze/animal-connect như các game trước.
- ℹ️ Mở bằng browser thường: tự giả lập popup app (next/retry sau 1.4s) để
  playtest — trong app thật không chạy.

## Đã kiểm thử (headless Edge)

- **Bot 9 màn: 404 lần nối, 1 lần auto-xáo, 0 lỗi JS, 0 vi phạm bất biến.** Sau
  **mỗi lần lắng** đều assert: `grid[][]` khớp `tiles[][]`, mọi cột liền mạch từ
  đáy, `transform` DOM khớp ô lưới, số tile = 2×`pairsLeft`, và **không loại nào
  còn số lẻ** (parity cặp). Bridge: `game_result/win` ×9, `ads` ×3 (mỗi 3 ván).
- **Tutorial**: `bendTurns = 3` (đúng bài học "thừa 1 lần rẽ"), `dropFromFall =
  true` (cặp bước 2 **thật sự** do cú rơi tạo ra), score/best **giữ 0** suốt và
  sau tutorial (bug rò điểm từng làm phồng best ở animal-connect), rồi vào màn 1.
- **28 assertion edge-case pass**: race (`loadLevel` **giữa lúc tile đang rơi**
  → `levelEpoch` huỷ continuation cũ, bàn vẫn nhất quán), pause (đồng hồ đứng,
  veil hiện, nền dừng) → resume (chạy lại), snapshot (khôi phục **đúng từng ô**,
  từ chối snapshot sai level), thua do hết giờ (đúng **một** `game_result/lose`,
  best thăng hạng, run reset về `levelIdx:0/score:0`), retry (về màn 1, giữ best).
- **Boot matrix**: `en` / `ar` (→ `dir=rtl` + `system-ui`) / `ja` (→ `system-ui`)
  / `vi` (giữ Google Sans Flex, chữ có dấu **đúng UTF-8, không mojibake**);
  `data.levelIdx` khôi phục đúng màn + score/best; `currentLevel` từ native
  **thắng** levelIdx đã lưu; `?reset=1` xoá localStorage.
- **Layout**: `#board-outer` `overflow:hidden`, dò scroll trên **đúng container
  cuộn** (không dùng `documentElement` — cái bẫy từng giấu bug của
  animal-connect cả một session) → **0 scroll cả hai trục**.
- ⚠️ **Chưa kiểm được**: Edge trên máy này **bỏ qua `--window-size` cho CSS
  viewport** (đo được: xin 412/360/320 đều ra **492×822**), nên **layout ở màn
  hẹp thật chưa reproduce headless được**. Công thức cỡ ô có sàn 16px và ở
  492px cho `cellPx = 35`; ước tính ở 360px là ~27px và ở 320px là ~23px —
  **cần playtest trên cửa sổ Edge app-mode 420×800 hoặc máy thật để xác nhận**.

## 📋 Backlog

- [x] ~~3 cover PNG vẽ tile CÓ bóng đổ cứng~~ → **đã render lại 2026-08-04**:
      tile phẳng đúng như trong game, và cover giờ kể đúng cơ chế (cột vừa dọn,
      tile đang rơi kèm vệt tốc độ, sao điểm bay lên). Sprite quả được **rút
      thẳng từ `index.html`** lúc sinh nên cover không thể lệch icon so với game.
- [ ] Kiểm layout ở màn hẹp thật (320/360px) — xem cảnh báo ở trên.
      **Chưa tự kiểm được**: Edge headless có chiều rộng cửa sổ tối thiểu, nên
      `innerWidth` không bao giờ xuống dưới **~492px CSS** dù truyền
      `--window-size` bao nhiêu (`--force-device-scale-factor` cũng không chia
      viewport). Cần máy thật hoặc DevTools protocol để phủ mốc này.
- [ ] Hướng trọng lực biến thể theo level (lên / trái / phải / vào giữa).
- [ ] Dịch nốt HUD/toast cho 23 ngôn ngữ nếu mentor muốn (hiện chủ đích để `en`).
- [ ] Confetti khi thắng màn; hiệu ứng riêng cho DROP COMBO ×3 trở lên.
