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
- **Ô SCORE không còn hiệu ứng throb** (bỏ 2026-08-05). Con số nhảy theo từng
  ngôi sao đáp đã là phản hồi rồi; phóng to cả cái ô chồng lên trên là thừa, mà
  với tối đa 6 sao mỗi lần nối thì nó nổ liên tiếp 6 lần.
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
4. **CLEAR** — dọn nốt bàn, hiện **timer demo** và **chính thanh giờ nhấp nháy**
   để tự giới thiệu (timer chỉ chạy sau thao tác đầu tiên, và **không bao giờ
   gây thua**).

   Trước đây là một **mũi tên SVG `position:fixed` chỉ lên thanh giờ** — nó phải
   được **đo và đặt lại vị trí theo thanh giờ mỗi lần layout đổi**. Thanh giờ tự
   nháy được thì cần gì ai chỉ vào: bỏ hẳn mũi tên (markup + CSS + JS), và
   `positionTutorialChrome()` không còn phải `getBoundingClientRect()` thanh giờ
   nữa. Nháy bằng `opacity` nên chạy trên compositor, không tốn main thread; và
   `steps()` ở đây là **đúng chỗ** — đây là một cú *nháy*, thứ mà stepped timing
   sinh ra để làm (một cú *scale* thì không).

**Tutorial TÍNH ĐIỂM y như màn thật** (sửa 2026-08-05). Trước đây cả khối tính
điểm nằm trong `if (!tutorialMode)`, nên bàn đầu tiên **im lặng hoàn toàn**:
không điểm, không sao bay, không lời khen, không drop-combo. Học cách chơi mà
không thấy phần thưởng thì dạy sai một nửa. Nay chạy đủ.

Hai ràng buộc đi kèm:

- **`BEST` là thứ duy nhất buổi tập KHÔNG được đụng.** Đó là kỷ lục bền, có
  đường ghi thẳng ra native, nên một buổi tập không được phép lập kỷ lục. Chỉ
  dòng ghi `best` bị chặn (`if (!tutorialMode && score > best)`); còn **BEST
  hiển thị vẫn leo theo SCORE** (qua `bestShown` trong `deliverScore`) nên nhìn
  vẫn y hệt màn thật.
- **Xong thì RESET.** `finishTutorial()` xoá điểm tập trước khi bàn giao cho màn
  1: gỡ sao còn bay (`clearFlyers`), `score = 0`, `updateScoreHud()` (đưa
  `scoreShown`/`scorePending` về 0), `updateBestHud()` (đưa BEST hiển thị về
  đúng kỷ lục thật). Làm **trước** `saveDataNow` nên điểm tập không bao giờ
  được lưu.

Đo với kỷ lục có sẵn 500: trong tutorial điểm leo tới 360 với **12 ngôi sao**
cùng lúc trên màn; sau khi xong `score = 0`, `BEST` thật vẫn **500**, HUD hiện
`SCORE=0 BEST=500`, không còn ngôi sao nào sót; localStorage giữ `score: 0,
best: 500`; và **đúng một** bản tin ra app, mang `score=0 best=500`.

`buildTutorialBoard()` sinh lại bàn tới khi **cả 4 bài học đều dạy được** và
greedy-solve (có trọng lực) còn **0 ô** — verify bằng máy, không phải bằng mắt.
Tutorial **không tính điểm** (score/best giữ 0), có Skip, `?reset=1` xem lại.

### Level sinh theo công thức — `levelConfig(idx)`

**Bàn to lên ở MỌI màn** từ L1 tới L12 (yêu cầu 2026-08-04), rồi chốt và
nhường việc cho đồng hồ:

| Level | Bàn (cột×hàng) | Ô | Cặp | Loại quả | Giờ | Giây/cặp |
|---|---|---|---|---|---|---|
| 1 | 5×6 | 30 | 15 | 5 | 60s | 4.00 |
| 2 | 6×6 | 36 | 18 | 6 | 65s | 3.61 |
| 3 | 6×7 | 42 | 21 | 7 | 70s | 3.33 |
| 4 | 6×8 | 48 | 24 | 8 | 75s | 3.13 |
| 5 | 7×8 | 56 | 28 | 9 | 80s | 2.86 |
| 6 | 7×10 | 70 | 35 | 10 | 90s | 2.57 |
| 7 | 7×12 | 84 | 42 | 12 | 90s | 2.14 |
| 8 | 7×14 | 98 | 49 | 12 | 95s | 1.94 |
| 9 | 7×16 | 112 | 56 | 12 | 90s | 1.61 |
| 10 | 7×16 | 112 | 56 | 12 | 85s | 1.52 |
| 11 | 7×16 | 112 | 56 | 12 | 75s | 1.34 |
| 12 | 7×16 | 112 | 56 | 12 | 70s | 1.25 |
| 13 | 7×16 | 112 | 56 | 12 | 65s | 1.16 |
| 14 | 7×16 | 112 | 56 | 12 | 55s | 0.98 |
| 15 | 7×16 | 112 | 56 | 12 | **50s** | 0.89 |
| 16+ | 7×16 | 112 | 56 | 12 | **50s — trần, giữ mãi mãi** | 0.89 |

**Bàn DỌC** (sửa 2026-08-05 theo brief: "thiết kế cho màn hình điện thoại dọc").
Level 1 là **6 hàng × 5 cột** — nhiều hàng hơn cột — và **không bao giờ quá 7
cột**.

- **Cột leo 5 → 6 → 7 rồi dừng; sau đó chỉ hàng mọc.** Bảy là trần brief đặt, và
  cũng xấp xỉ thứ màn hình muốn: ở 7 cột máy 360px cho 40px bề ngang mỗi ô, tức
  **chiều rộng thôi ràng buộc và chiều cao tiếp quản**.
- **Bước hàng KHÔNG phải lúc nào cũng bằng 1** — đó là *tính chẵn lẻ*, không
  phải cẩu thả. `COLS × ROWS` **phải chẵn** hoặc bàn không chia hết thành cặp.
  Với 6 cột (chẵn) hàng bước được 1; với 5 hoặc 7 (lẻ) hàng buộc phải chẵn nên
  bước 2. Vì thế đuôi bảng là `7×8, 7×10, 7×12` chứ không phải `7×8, 7×9, 7×10`.
- Dãy số ô `30 36 42 48 56 70 84 98 112` — bước `+6,+6,+6,+8,+14,+14,+14,+14`.
  Bàn ngừng lớn ở **7×16** và từ đó đồng hồ làm nốt việc.

#### Bàn dọc LẬT NGƯỢC quy luật "rộng thắng cao" — và điều đó nay chấp nhận được

Bảng cũ được xây quanh phát hiện *bàn nằm bế tắc ít hơn bàn đứng 4–10 điểm*.
Điều đó **từng quan trọng** vì bàn chết đồng nghĩa **xáo cả bàn ngay trước mặt
người chơi**. Nay **không còn**: một bàn chết chỉ tốn **một cú hoán vị hai ô**
(xem `repairOneSwap`), nên cái giá của việc quay dọc được trả bằng thứ người
chơi gần như không thấy, thay vì bằng một cú xáo tung.

Đo lại trên **chính bảng này**, 200 ván đối kháng mỗi nấc:

| Bàn | 5×6 | 6×8 | 7×10 | 7×12 | 7×16 |
|---|---|---|---|---|---|
| Lần sửa / ván | 0.17 | 0.26 | 0.43 | 0.95 | 1.09 |

**1800/1800 ván đều dọn sạch bàn**, 0 lần phải xáo. Chơi thật tốt hơn ~4×, tức
tệ nhất khoảng **1 lần thấy 2 ô đổi quả mỗi ~4 màn**.

- **Tile TO HƠN trước**, vì ít cột hơn: nhỏ nhất **24px @320×568** (trước là
  20px), 27px @360×640, 42px @412×915.
- **Đánh đổi phải nói rõ:** bàn cao + màn ngắn thì **chiều cao mới là thứ ràng
  buộc**, nên bàn chỉ lấp **~64% bề ngang** trên máy 320–375px (lề hai bên
  rộng). Trên máy dài 412×915 thì lấp 85%. Đây là hệ quả trực tiếp của việc
  chọn bàn dọc, không phải lỗi layout.
- **Pool giảm 192 → 112 tile** (`MAX_CELLS = 7×16`), tức **ít hơn 42% node DOM**
  phải dựng và bố trí — thêm một khoản lãi cho máy yếu.

#### Ba pha độ khó — mỗi lúc chỉ MỘT đòn bẩy chạy

Chốt 2026-08-05 theo brief: *"từ màn 10 trở đi không tăng số tile nữa, mà giảm
thời gian dần đến màn 15, từ màn 16 trở đi đến vô tận giữ nguyên độ khó."*

| Pha | Màn | Cái gì đang chạy |
|---|---|---|
| A | 1 – 9 | **BÀN to dần** (5×6 → 7×16), đồng hồ siết theo |
| B | 10 – 15 | Bàn đứng yên; **ĐỒNG HỒ siết một mình** 85s → 50s |
| C | 16 → ∞ | **Không gì đổi nữa.** Trần độ khó: 7×16, 56 cặp, 50s |

Điểm bàn giao được đặt trùng khít có chủ ý: `perPair` bằng `1.6` ở **M9** —
vừa là cuối pha A vừa là đầu pha B — và chạm sàn đúng ở **M15**, nên **M16 là
màn đầu tiên giống hệt màn trước nó**. Ba hằng số trong `levelConfig` nói thẳng
điều đó: `GROWTH_END = 8`, `SQUEEZE_END = 14`, `PER_PAIR_FLOOR = 0.9`.

> Một chỗ không đều cần biết: trong **pha A** thời gian **không đơn điệu tăng** —
> M7 bằng M6 (90s) và M9 tụt nhẹ so với M8 (95s → 90s). Đó là vì `perPair` giảm
> nhanh hơn số cặp tăng ở hai nấc đó. Màn vẫn khó hơn (bàn to hơn hẳn), nên em
> để nguyên; nếu muốn giờ tăng đều tuyệt đối thì phải làm pha A thoải hơn, mà
> như thế sẽ kéo dài run của người chơi trung bình.

Một run kết thúc khi đồng hồ siết chặt hơn tốc độ người chơi, nên mọi thứ suy ra
từ **một giả định**: người chơi cần bao nhiêu giây cho một cặp, tính cả thời
gian dò tìm. Mô hình chạy trên chính bảng giờ ở trên:

| Người chơi | giây/cặp | Thua ở màn | Dài một run |
|---|---|---|---|
| Chậm | 2.6 | 6 | **6.1 phút** |
| Trung bình | 2.2 | 7 | **6.7 phút** |
| Nhanh | 1.8 | 9 | **8.5 phút** |
| Rất nhanh | 1.5 | 11 | **9.9 phút** |
| Khó tin | 1.2 | 13 | 10.2 phút |
| Siêu nhân | 1.0 | 14 | 9.5 phút |

Hai hàng cuối **nhỉnh hơn mốc 10 phút** một chút, và đó là **cái giá trực tiếp
của việc kéo đoạn siết ra tới M15**: đuôi dài hơn thì người nhanh nhất được
thêm vài màn trước khi bị bóp. Nó chỉ ảnh hưởng tới những tốc độ chưa playtest
nào chứng minh là có thật, còn dốc thêm pha A để đòi lại thì sẽ làm hỏng trường
hợp phổ biến — nên em để nguyên.

- **Sàn `perPair` BẮT BUỘC phải thấp hơn tốc độ nhanh nhất có thể**, không thì
  người giỏi **không bao giờ thua**. Lần chỉnh trước em để sàn `1.5` — đúng bằng
  tốc độ người chơi rất nhanh — và mô hình cho ra **274 phút** không trượt màn
  nào. Sàn nay là `0.9`, dưới mọi tốc độ người thật giữ được qua 56 cặp.
- **Con số giây/cặp là GIẢ ĐỊNH, không phải phép đo** — nó là con số duy nhất
  cần chỉnh lại nếu playtest thật nói khác, và mọi thứ còn lại suy ra từ nó.

- **Tối đa 12 loại quả trên một bàn, và cửa sổ XOAY theo level** nên cả **14
  icon** đều xuất hiện trong một run — có đa dạng mà không trả giá deadlock.
  Số loại bám theo cỡ bàn để luôn còn ~3+ cặp mỗi loại.
- Vô hạn → **không có màn cuối, không bắn `victory`** (deviation có chủ ý so với
  game-common §1.5, giống animal-connect / runic-blaze).

### Gạch nối — vẽ theo kiểu `animal-connect`

Sửa 2026-08-05, feedback: "cái gạch nối hơi thô → dùng cái dạng mà game
animal-connect làm". Đối chiếu hai file thì khác biệt nằm ở đúng ba chỗ:

| | fruit-drop (cũ) | animal-connect | fruit-drop (nay) |
|---|---|---|---|
| `stroke-linecap` | `square` | `round` | **`round`** |
| `stroke-linejoin` | `miter` | `round` | **`round`** |
| Mờ dần | `steps(4,end)` | `ease-in` | **`ease-in`** |
| Viền mực | `max(5, cell·0.17)` = 6px | — | **`lõi + 2`** = 5px |
| Thời gian sống | .34s / xoá ở 360ms | — | **.2s / xoá ở 210ms** |

Vòng hai (2026-08-05, "vẫn hơi thô → viền mảnh nhất có thể, biến mất sớm hơn"):
viền mực chỉ tồn tại để đường nối đọc được **khi nó cắt ngang một tile**, nên
nay nó mảnh đúng mức tối thiểu làm được việc đó — **lõi cộng 1px mực mỗi bên**.
Trước là gần gấp đôi lõi, đó là chỗ còn nặng. Tỉ lệ viền/lõi 2.0 → **1.67**.
Thời gian sống bị siết hai lần theo feedback: `.34s → .26s → .2s`, giữ trước
khi mờ `70% → 55% → 40%`, xoá khỏi DOM `360ms → 280ms → 210ms`.

`miter` **bắn một cái gai ra khỏi mỗi góc rẽ**, còn `square` thì **thò quá ô
cuối** — đó chính là chỗ "thô". Fade 4 nấc trên một nét mảnh thì đọc ra là
nhấp nháy chứ không phải mờ dần. Đường vẽ minh hoạ trong tutorial (`.bend-*`)
sửa cùng lý do.

### Không chặn nhấn giữa animation

Feedback: "đừng chặn nhấn khi đang diễn anim mấy ngôi sao bay lên, nó bị mất
nhịp chơi á". Đo trước đã, vì cảm nhận và nguyên nhân không trùng nhau: sao
bay tới **826ms**, nhưng `busy` chỉ khoá tới **200ms** (dài hơn khi có tile
rơi) — tức game **không** hề chặn suốt lúc sao bay. Cái người chơi vấp là
**cửa sổ chết ~200-440ms** ngay sau khi nối, và họ quy nó cho hiệu ứng sao vì
đó là thứ đang chuyển động trên màn hình.

Cách sửa: **không nuốt cú chạm nữa**. Trong lúc bàn còn pop/rơi, cú chạm vẫn
được nhận và hiện vòng chọn ngay; nếu nó hoàn thành một cặp thì cặp đó được
**xếp hàng** và nổ ra đúng lúc bàn lắng xuống (`runPendingMatch`). Nhịp chơi
liền mạch mà **không** có hai lượt trọng lực chồng nhau.

Hai chi tiết bắt buộc để nó đúng:
- `selected` nay giữ **đối tượng tile**, không phải cặp toạ độ `{r,c}`. Toạ độ
  sẽ **ôi thiu**: tile được chọn trong lúc pop có thể bị lượt trọng lực ngay
  sau đó dời đi, và `{r,c}` khi ấy trỏ vào **tile khác** vừa rơi vào chỗ đó.
- Cặp đang xếp hàng được **kiểm lại trên bàn ĐÃ LẮNG** (còn tồn tại? còn cùng
  loại? còn nối được?) trước khi chơi — đường đi tồn tại lúc đang rơi có thể
  không còn tồn tại sau khi rơi xong.
- **Tutorial nay cũng KHÔNG chặn nữa** (sửa 2026-08-05 — trước đó nó giữ khoá
  cứng `if (busy && tutorialMode) return;`). An toàn vì **chính tutorial đã chặn
  ở tầng khác**: bước 0–2 chỉ nhận cú chạm vào đúng cặp mẫu (`tutAllowsFirst`),
  nên **không thể** có nước xếp hàng làm hụt bài học; bước 3 là chơi tự do và
  `tutOnBoardChange` ở bước đó chỉ kiểm `pairsLeft <= 0`, tức nếu một nước xếp
  hàng làm bỏ qua một nhịp `afterBoardChange` thì lần settle sau bắt lại ngay.
  `setTutPhase` vẫn gọi `clearPendingMatch()` cho chắc, vì đổi bước là đổi luôn
  cặp nào được coi là hợp lệ.
- `autoShuffle` huỷ cả ô đang chọn lẫn cặp đang xếp hàng vì nó gán **loại quả
  mới** lên chính những tile cũ.

### Hai lỗi playtest 2026-08-06 (mentor báo)

**1. Tutorial kẹt còn 2 cặp không nối được.** Nguyên nhân: `afterBoardChange()`
mở đầu bằng `if (tutorialMode) { tutOnBoardChange(); return; }` — **return
trước** dòng `if (!hasMoves()) fixDeadBoard(ep)`. Tức **tutorial hoàn toàn
không có lưới chặn bế tắc**; cả `repairOneSwap` lẫn `autoShuffle` đều không bao
giờ chạy ở đó. Bàn mentor gặp là **2×2 xen kẽ** (táo/chuối một đường chéo,
chuối/táo đường kia): hai cặp, **không cặp nào nối được**, vì mọi đường giữa một
đường chéo đều bị cặp kia chặn — kể cả vòng ra ngoài viền.

Sửa: `tutOnBoardChange` nhận `ep` và chạy đúng lưới đó ở **bước 3**. Bước 0–2
không cần: chúng chỉ nhận cặp mẫu, mà cặp mẫu được generator bảo đảm nối được.

Bài học rút ra và đã ghi thẳng lên đầu hàm: `tutOnBoardChange` là **bản thay thế
trọn vẹn** cho phần còn lại của `afterBoardChange`, nên **thứ gì phải đúng trên
mọi bàn thì phải lặp lại ở đây**.

**2. Nhấp nháy lệch nhịp.** `hintPair` lưu **toạ độ**. Trọng lực dời tile đang
được gợi ý đi chỗ khác, rồi `clearHint()` tra `tiles[r][c]` và gỡ class khỏi
**tile vừa rơi vào ô đó** — tile thật vẫn nháy mãi. Gợi ý kế tiếp thắp thêm ô
nữa, mỗi ô bắt đầu ở một thời điểm khác nhau → chúng nháy chọi nhau.

**Đúng loại lỗi đã sửa cho `selected` hôm trước mà bỏ sót `hintPair`.** Nay
`hintPair` giữ **đối tượng tile**, và hai nửa được **tua về cùng mốc 0** vô điều
kiện (một tile đang được gợi ý sẵn thì remove+add trong cùng một task không hề
restart animation, nên vẫn lệch nếu không tua).

Đo: gợi ý đầu sáng đúng 2 ô → sau một lượt trọng lực còn **0 ô sót** → gợi ý thứ
hai sáng đúng **2 ô**, lệch nhịp **0 ms**, `clearHint` gỡ sạch.

### Không bao giờ phải xáo bàn nữa

Hỏi 2026-08-05: "làm sao để 0 bao giờ phải xáo mà tốn ít tài nguyên?". Câu trả
lời hoá ra **rẻ hơn cả cách đang làm**, và nó là một **chứng minh**, không phải
mẹo may rủi:

> Tile chỉ rời bàn theo **cặp cùng loại**, nên mọi loại còn trên bàn luôn có
> **số lượng chẵn** — tức mọi tile còn lại đều **còn bạn ở đâu đó**. Dời bạn nó
> vào một ô sát bên là hai đứa **kề nhau**, mà hai tile cùng loại kề nhau thì
> **luôn nối được với 0 lần rẽ**. Vậy **một cú hoán vị luôn cứu được bàn**.

Code cũ **đã biết điều này**: `forceOneMove()` làm đúng cú hoán vị đó. Nó chỉ
nằm sau **40 vòng xáo cả bàn** như phương án chót — tức cách rẻ và *chắc chắn
đúng* chỉ được chạy sau khi cách đắt và *không đảm bảo* đã thất bại 40 lần.
Giờ hoán vị là đường chính (`repairOneSwap`), xáo bàn thành nhánh không-bao-
giờ-chạy và có counter `shuffleFallbacks` để test khẳng định nó bằng 0.

Đo:

| | xáo cả bàn (cũ) | một hoán vị (nay) |
|---|---|---|
| Ghi lại loại quả | 8 (tới 192 nếu bàn dày) | **2** |
| Lượt tìm đường BFS | 7 (tới 40 vòng) | **0** |
| Xáo mảng | 1 | **0** |
| Người chơi thấy | cả bàn đổi quả + toast | **đúng 2 ô đổi quả** |

Chi phí nay là **hằng số**, trước thì tỉ lệ với số tile còn lại. (Mấy con số
xáo nhỏ vì bàn chết **hầu như luôn xảy ra lúc gần cuối màn**, khi còn ít tile.)

Kiểm chứng hai tầng:
- **1000 ván đối kháng** trên 5 cỡ bàn khắp ramp: **200/200 ván mỗi cỡ dọn sạch
  bàn**, 0 lần phải xáo, 0 lần sửa thất bại, 0 lần sửa xong vẫn kẹt.
- **5 bàn chết thật, chạy qua DOM thật**: mỗi lần đổi **đúng 2 ô**, bàn sống
  lại, `grid`/`tiles` không lệch, số lượng từng loại giữ nguyên và vẫn chẵn.

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
| **Lật khung, N=300/cỡ** (2026-08-05) | 7×6 **11.0%** vs 6×7 **15.3%** · 8×7 **10.0%** vs 7×8 **16.0%** · 9×8 **8.7%** vs 8×9 **18.7%** · 10×9 **9.7%** vs 9×10 **14.0%** · 11×10 **12.0%** vs 10×11 **16.3%** · 12×11 **10.7%** vs 11×12 **15.0%** |
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

> ⚠️ **Hai mục đo dưới đây mô tả bảng ramp NẰM (≤12 cột) đã bị thay bằng bảng
> DỌC ở trên vào 2026-08-05.** Số liệu vẫn đúng và vẫn là lý do quy luật
> "rộng thắng cao" tồn tại — nhưng nó **không còn quyết định bảng ramp**, vì
> `repairOneSwap` đã làm bàn chết gần như miễn phí. Giữ lại để đừng ai vô tình
> "sửa" bàn dọc về nằm mà tưởng là phát hiện mới.

**Đo lại 2026-08-05 khi làm ramp thoải hơn:** feedback đề nghị dãy `6×7, 7×8,
8×8…` — tức bàn **cao hơn rộng**. Đo sáu cỡ, mỗi cỡ 300 ván, **cùng số ô và
cùng số loại**, khác biệt duy nhất là *xoay hướng nào*: **cả sáu lần đều cho
cùng một câu trả lời**, bàn đứng tốn **4–10 điểm** deadlock. Nên em giữ đúng
**bậc tăng** anh muốn nhưng **lật lại cho `cols ≥ rows`** — cùng dãy số ô, không
trả giá. Toàn ramp mới đo được **12.0% đối-nghịch ≈ 3.0% chơi thật**, tức
**1 lần auto-xáo mỗi ~33 màn** (trước là ~27).

Vừa màn hình, đo trên chính đường `resize()` của bản ship với khung bàn bị ép
về từng bề rộng máy: tile nhỏ nhất **20px @320px · 23px @360px · 26px @412px**,
và điểm nhỏ nhất rơi vào **L11 (12×10)** chứ không phải L17 — vì **chiều rộng
mới là thứ ràng buộc**, thêm hàng về sau không làm tile nhỏ thêm chút nào.

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
  halftone, viền mực đen 3px, khối màu phẳng.
- **`steps()` dùng cho FADE/NHÁY, KHÔNG dùng cho PHÓNG TO/THU NHỎ**
  (sửa 2026-08-05 theo playtest: "mấy hiệu ứng phóng to thu nhỏ ô hơi giật").
  Trên một cú mờ dần hay nháy, `steps()` đọc ra là *có chủ ý*; trên một cú
  **scale** thì nó chỉ đọc ra là **máy rớt frame**. Nên chọn tile, pop, land,
  pop-in và nhịp ô SCORE nay đều dùng `ease` — đúng như `animal-connect` làm.
  Còn nháy gợi ý / mờ đường nối / kiến bò vẫn giữ `steps()`.
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

### Lưu tiến trình — chỉ sau mỗi NƯỚC NỐI

Ghi `localStorage` **một lần cho mỗi nước nối đã hoàn tất**, không hơn.
`afterBoardChange()` chạy khi bàn lắng xuống là điểm ghi duy nhất trong lúc
chơi; ngoài ra chỉ còn các mốc đổi trạng thái thật sự: auto-xáo, vào màn mới,
bấm Back, đổi âm thanh.

Đã thử hai hướng rộng hơn và **bỏ cả hai** vì đây là game mobile-first:
- ~~Tự lưu theo nhịp thời gian~~ — sai ngay từ ý niệm: tốn tài nguyên vì đồng
  hồ chứ không phải vì người chơi làm gì.
- ~~Ghi theo mọi thao tác (chọn / bỏ chọn / chạm hụt)~~ — gấp đôi số lần ghi
  (2/nước) để đổi lấy rất ít: bàn cờ y nguyên, chỉ có đồng hồ nhích, mà nước
  nối kế tiếp ghi lại nó ngay.

Tải ghi khi chơi: **1 lần/nước, 571 byte**.

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
| **Kích thước tile = MỘT biến CSS** | Mọi tile trên bàn cùng một cỡ, nhưng code từng ghi `width` + `height` inline cho **từng** tile — 384 lượt ghi style chỉ để nói một con số, mỗi lần load level, và 192 lượt nữa mỗi lần resize. Nay `#board` mang `--tile-size` và tile kế thừa: **384 → 1**, resize **192 → 1**. `layoutAll` chỉ còn đặt vị trí. |
| **Không bố trí tile hai lần** | `takeTile` từng gọi `applyTileSize` + `applyTilePos` cho từng tile, rồi `resize()` → `layoutAll()` ngay sau đó làm lại toàn bộ. Trên bàn 192 tile là **192 lượt ghi width/height/transform thừa mỗi lần load level** — mà lượt đầu còn **sai**, vì kích thước bàn đổi giữa các màn nên `cellPx` lúc đó vẫn là của màn TRƯỚC, chỉ được lượt sau ghi đè. Đo: **384 → 192** mỗi loại. |
| **Không ép layout đồng bộ nữa** | Cách restart animation kinh điển là *bỏ class → đọc `offsetWidth` → thêm class lại*; cú đọc đó **flush style + layout của CẢ tài liệu**. Đây là thao tác DOM đắt nhất còn sót trên đường nóng, và nó xảy ra **đúng lúc tile đang chạy animation** — thời điểm tệ nhất để chặn main thread. Đo được **~1 lần mỗi nước** chỉ riêng từ popup praise/combo, cộng thêm một lần mỗi cú chạm hụt. Nay `restartAnim()` tua lại chính đối tượng animation (`getAnimations()`), không đụng gì tới layout; đường cũ chỉ còn dùng cho lần chạy đầu và WebView quá cũ. Đo: **1 → 0.1 mỗi nước (−90%)**. |
| **Pop-in vào màn = MỘT class trên bàn** | `popInAll` từng thêm class `popin` cho **từng** tile rồi gỡ ra — bàn 112 tile là **224 thao tác classList** đúng vào lúc màn đang cố hiện ra mượt. Nay là **2** (một class trên `#board`). Độ ưu tiên CSS được giữ **y nguyên ở (0,3,0)** — `.board-popin .tile .tile-face` đếm đúng ba class như `.tile.selected .tile-face` — nên thứ tự khai báo vẫn là thứ quyết định, đúng như trước. **Đừng** "dọn" thành `#board.popin ...`: id sẽ vượt mặt các rule trạng thái và ô được chọn trong 220ms đầu sẽ không hiện gì. Đo: **115 → 4** thao tác class mỗi lần vào màn (−97%). |
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
