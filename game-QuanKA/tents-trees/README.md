# Tents and Trees — QuanKA

> **Puzzle logic cắm trại** (thể loại "Tents / Tents & Trees") | HTML5 single-file | 60 màn cố định

## Thông tin game

| Trường | Giá trị |
|---|---|
| **Package** | `com.falcon.tentstrees` |
| **Engine / Version** | HTML5 single-file · 1.0.0 |
| **Category** | PUZZLE |
| **Loại** | Game **có level** — 60 màn, tính điểm theo **run**, `victory` ở màn cuối |
| **Kích thước** | `index.html` ~94 KB (không asset ngoài; 30 KB trong đó là icon bàn tay tutorial dạng base64) |
| **Tham khảo** | Tents (Nikoli / Simon Tatham's Puzzle Collection) |

## Vì sao chọn thể loại này

Lọc lại **~30 game live trên app** + sheet `MiniGame-TTS-PTIT.xlsx`: mảng
**arcade/action gần như kín** (team Thắng & Bình đã có Helix Jump, Stackfall,
SliceMaster, Dino, Flappy, Zig Zag, Color Switch, Doodle Jump, Tower Climb,
Sticky Orbit, Gravity Flip, Ball Run, Fruit Ninja, Level Devil, Dune, Basketball,
Ping Pong, Stack Tower). Lane còn lại của QuanKA là **puzzle**.

**Tents không có ở cả app lẫn sheet.** Ngoài chỗ trống, nó còn hợp vì tái dùng
đúng bài đã chứng minh hai lần (Energy Loops, Bridge Islands): **dựng lời giải
trước, suy ngược ra đề**. Thêm nữa **art cây/lều ấm áp** — icon sẽ nổi giữa một
rừng icon trừu tượng, khác hẳn hai game hình học phẳng trước đó của QuanKA.

## Gameplay

- Lưới ô, một số ô có **cây**. Người chơi đặt **lều** vào ô trống.
- **Chạm một ô**: `trống → cỏ(·) → lều → trống`.
- Luật: **mỗi cây ghép với đúng một lều** kề nó theo hàng/cột (ghép 1-1);
  **hai lều không được chạm nhau**, kể cả **chéo**; số ở rìa hàng/cột = số lều
  trong hàng/cột đó.
- **Ngân sách lượt** mỗi màn. Hết lượt mà chưa xong = **THUA = hết run** → app
  hiện popup thua, Retry là **chơi lại từ màn 1** (giữ BEST).
- **SCORE cộng dồn cả run**: `50 × số màn + 25 × số lượt còn thừa`.
- **Sao**: `moves ≤ par` → 3★, `≤ par×1.4` → 2★, còn lại 1★.

## Điều kiện thắng: phép thứ ba KHÔNG phải phép đếm

Bridge Islands cần 2 phép kiểm. Tents cần **3**, và phép thứ ba là chỗ thể loại
này hay bị cài sai:

1. mỗi hàng/cột đúng số lều như clue;
2. không hai lều nào chạm nhau (8 hướng);
3. **tồn tại ghép cặp hoàn hảo** giữa tập cây và tập lều.

> **Cách cài sai kinh điển** là viết phép 3 thành *"mọi cây đều có một lều kề
> bên"*. Một bàn 3×3 đủ để bác bỏ:
>
> ```
> T A T      cây (0,0) và (0,2); lều (0,1) và (2,2)
> . . .      cả HAI cây chỉ kề đúng một lều là (0,1)
> . . C      lều (2,2) không kề cây nào
> ```
>
> Mọi cây đều có lều kề ✓. Số lều = số cây ✓. Không lều nào chạm nhau ✓. Clue
> hàng/cột khớp ✓. **Nhưng** hai cây tranh nhau một lều còn một lều bị bỏ rơi →
> ghép cặp lớn nhất chỉ được 1 < 2 ⇒ **không phải lời giải**.

Cài bằng **đường tăng luồng (Kuhn)** trên đồ thị hai phía cây × lều. Bộ test dựng
đúng bàn cờ trên và bắt game phải từ chối — kèm khẳng định phép kiểm ngây thơ
**chấp nhận** nó, để chứng minh bài test có ý nghĩa.

> Quét vét cạn 5 màn nhỏ của bản ship **không** tìm thấy thế cờ nào rơi vào bẫy
> này. Nghĩa là các màn đã ship không bao giờ đưa người chơi vào tình huống đó —
> nhưng luật vẫn phải đúng, và ca 3×3 dựng tay là thứ chứng minh điều đó.

## Sinh màn — dựng lời giải trước

1. Lặp tới khi đủ `N` cây: chọn ô trống làm **lều** với điều kiện **không có lều
   nào trong 8 ô quanh nó**, rồi chọn một ô trống **kề hàng/cột** làm **cây**.
2. Đếm lều theo hàng/cột → ra clue. Xoá lều đi → còn cây + clue = đề bài.

Cặp (cây, lều) dựng ra **chính là một ghép cặp hoàn hảo**, và luật không-chạm
được kiểm ngay lúc đặt ⇒ **lời giải tồn tại theo cách dựng**.

### Solver — chỉ suy luận, không đoán

| Quy tắc | Nội dung |
|---|---|
| **Không kề cây** | ô không kề cây nào theo hàng/cột → cỏ |
| **Quanh lều** | 8 ô quanh một lều → cỏ |
| **Bão hoà hàng/cột** | đủ số → phần còn lại là cỏ; thiếu đúng bằng số ô chưa biết → tất cả là lều |
| **Cây bị dồn** | cây chỉ còn một ô ứng viên → ô đó là lều |
| **Ép bằng matching** | đặt thử ô đang xét thành cỏ; nếu **hết ghép cặp hoàn hảo** thì ô đó buộc phải là lều |

**Chỉ nhận màn nếu solver đi tới lời giải đầy đủ** ⇒ vừa **giải được không cần
đoán**, vừa **lời giải duy nhất** (mỗi bước là lựa chọn duy nhất). Test còn so
nghiệm solver với nghiệm lúc sinh — khác nhau nghĩa là **solver sai**, không phải
đề mơ hồ. Qua 60 màn: sai khác **0**.

> **Một quy tắc đã bị xoá vì đo ra là code chết.** Bản đầu có thêm chiều ngược
> lại: *"đặt thử thành lều, nếu hỏng matching thì phải là cỏ"*. Nó **không bao
> giờ fire được**: ô `UNKNOWN` vốn đã là ứng viên trong phép matching, nên thêm
> nó vào lần nữa cho ra đúng đồ thị cũ. Đo trên 221 màn: **0 lần**. Đã xoá — nó
> tốn một lượt matching cho mỗi ô mỗi vòng quét mà không làm gì.

### Bảng seed nướng sẵn

Seed từng màn tìm **offline** rồi nướng vào `SEED_OFF`: **vào màn chỉ chạy
generator đúng 1 lần**, không dò. Vẫn còn nhánh dò dự phòng nếu sau này sửa
generator; test bắt cả 60 seed phải **trúng ngay lần đầu**.

**Độ khó lấy từ đâu?** `par` ở game này **chính là số cây**, nên nó bị ramp quy
định và giống hệt nhau trong cùng một dải màn. Cần một đòn bẩy thứ hai: **số bước
phải dùng tới quy tắc matching**. Nhưng phải **đo trước rồi mới đặt mục tiêu** —
chỉ **6/221** màn cần quy tắc đó, khó nhất dùng 2 lần. Bản nháp đầu nhắm 3–4 bước
và **31/60 màn không sinh nổi**. Mục tiêu thật: **0 bước (màn 1–14) → 1 → 2**.

## Ramp 60 màn

| Màn | Lưới | Số cây (= par) | Bước matching |
|---|---|---|---|
| 1–5 | 5×5 | 4 | 0 |
| 6–14 | 6×6 | 6 | 0 |
| 15–25 | 6×8 | 8 | 1 |
| 26–38 | 7×9 | 11 | 1–2 |
| 39–50 | 7×11 | 14 | 1–2 |
| 51–60 | 7×13 | 17 | 1–2 |

Tối đa **7 cột** + 1 cột clue = 8 cột hiển thị. Bàn cao nhất (7×13) đo được
**368×644px** trên màn 390×844 — vẫn lọt.

## Đếm lượt — đánh dấu cỏ PHẢI miễn phí

```
par = số cây
movesLimitFor(par, lv):  lv≤9 →3.0 | ≤20 →2.5 | ≤35 →2.2 | ≤50 →2.0 | else 1.9
                         return max(par + 4, ceil(par * k))
```

**Chỉ lều lên/xuống mới tính lượt.** Đánh dấu cỏ là công cụ suy luận cốt lõi —
tính tiền nó là phạt đúng lối chơi cẩn thận mà game muốn thưởng.

> **Thứ tự vòng chạm là chỗ bản đầu làm sai.** Vòng ba trạng thái luôn bắt đi
> xuyên qua trạng thái giữa, nên **trạng thái nằm giữa là trạng thái phải trả
> giá**. Bản đầu để `trống → lều → cỏ`, tức **mỗi dấu cỏ tốn 2 lượt**; một ảnh
> chụp với 8 dấu cỏ đã ngốn gần hết ngân sách. Mà trong Tents, ô cỏ nhiều gấp
> bội ô lều (bàn 7×13 có 91 ô, chỉ 17 lều). Đã đảo thành **`trống → cỏ → lều`**:
> cỏ 1 chạm 0 lượt, lều 2 chạm 1 lượt. Người chơi hoàn hảo tốn đúng `par` lượt.

Đo thực tế: **người chơi hoàn hảo dùng 52% ngân sách** ở màn nặng nhất.

## Giao diện

- **Header** bê nguyên khối `#topbar` của Bridge Islands (vốn lấy từ
  animal-connect, đúng cỡ mentor yêu cầu).
- **Bàn chơi là một `<svg>` duy nhất**, có **máng clue** ở trên và bên trái
  (viewBox `(W+1) × (H+1)` ô). Mọi ô và mọi mảnh được tạo **một lần** lúc vào
  màn; chạm chỉ **đổi class** → không tạo/xoá node nào, không thể ép layout.
- Ô kẻ caro nhạt để đếm hàng/cột cho dễ.
- **Clue xanh khi đủ, đỏ khi thừa.**
- **Lều chạm nhau → cả hai chuyển đỏ.** Thêm sau khi nhìn ảnh chụp: clue **không
  thể** báo lỗi này (một hàng có thể đúng *số lượng* lều mà vẫn có hai cái dính
  nhau), nên nếu không có tín hiệu tại chỗ thì người chơi bí mà không hiểu vì sao.

### Tutorial — 4 bước, bám trạng thái thật

| Bước | Visual | Chữ | Chuyển bước khi |
|---|---|---|---|
| 1 | bàn tay trên **một ô mà lời giải có lều** | "Tap twice for a tent" | **lều dựng lên** |
| 2 | vòng ripple trên **cái cây của lều vừa dựng** | "One tent per tree" | 2,6s |
| 3 | ripple trên lều + **sáng 8 ô quanh nó** | "Tents never touch" | 2,6s |
| 4 | — | "Match the numbers" | 3s → đóng |

- Chữ bước 1 nói **"twice"** có chủ ý: chạm lần đầu ra cỏ, lần hai mới ra lều —
  một câu hứa "một chạm ra lều" sẽ đọc thành cú chạm chết.
- **Không chặn thao tác, và bám theo người chơi**: dựng lều ở chỗ khác thì bước 2
  ring **đúng cái cây của lều đó**. Chạm vào ô cây thì không có gì xảy ra và
  không mất lượt.
- Vòng ripple để **trắng**, không dùng màu theme, vì phải nổi trên cả cây xanh,
  lều cam, clue xanh lẫn clue đỏ.

Hỗ trợ `?tutorial=1` (chơi lại tutorial — quy ước mentor có kiểm, thấy ở tab
`Feecback` dòng r18) và `?reset=1` (xoá sạch tiến độ đã lưu).

## Tuân thủ quy ước chung

- **game-common** — `sendMessage` đủ 5 trường; `level` 1-indexed; thắng/thua đều
  `showModal:true`; `victory` màn 60; `quit` từ nút Back; `ads` mỗi 5 màn; 3
  callback `onAppPause`/`onNextLevel`/`onRetryLevel`; `waitForNativeInjection`
  200ms; `statusBarHeight`/`currentLevel`/`data`/`language` đúng thứ tự ưu tiên.
- **popup-common §1** — game theo level ⇒ **KHÔNG tự vẽ popup kết quả**. Game chỉ
  tự vẽ **màn hình sao** — thứ app không thể biết.
- **Lưu 2 tầng (§1.2)** — nhẹ qua `save_data`; snapshot ván dở (`blv`, chuỗi
  trạng thái ô, `mv`) chỉ ở `localStorage` (`tents_trees_v1`), ghi gộp 500ms +
  flush thật ở `onAppPause` / `visibilitychange` / `pagehide`.
- **i18n** — `I18N` inline, 23 ngôn ngữ cho tutorial; nhãn UI giữ tiếng Anh.
- ℹ️ `button-common.md` / `background-common.md` vẫn **không tồn tại trong repo**
  → `btn3d` lấy từ bridge-islands như các game trước.

## Kiểm chứng

Chạy bằng headless Edge (máy không có node).

| Hạng mục | Kết quả |
|---|---|
| Tổng số assertion | **2.885 (bot) + 138 (tutorial) + 22 (perf/khôi phục/native) — 0 fail** |
| Seed nướng sẵn trúng ngay lần đầu | **60/60** |
| Solver giải được (no-guess + duy nhất) | **60/60** |
| Solver ra nghiệm **khác** lúc sinh (⇒ solver sai) | **0** |
| Đường cong `par` | 4 → 17, **0 bước lùi** |
| Độ sâu suy luận | 0 → 2 bước matching |
| Bot giải trọn qua đúng đường input thật | **60/60**, đúng `par` lượt mỗi màn |
| **Bẫy thể loại** (3×3 dựng tay) | phép kiểm ngây thơ **chấp nhận**, game trả `"matching"` và **từ chối** |
| Test âm: thiếu 1 lều / hai lều chạm nhau | đều **không** báo thắng |
| Hình học: clue tổng = số cây, lều không đè cây, bậc hợp lệ | **0 vi phạm** |
| Đánh dấu cỏ | **20 dấu = 0 lượt**; lều lên = 1, lều xuống = 1 |
| Fuzz | **4.327** cú chạm, 0 lần vượt ngân sách, 0 trạng thái sai miền, 0 lần chạm được vào ô cây |
| Lỗi JS | **0** |
| Khôi phục ván dở | đúng màn, **khớp từng ô**, đúng số lượt (4 lều + 6 dấu cỏ) |
| Input native | `?statusBarHeight=44&currentLevel=7&language=vi&tutorial=1` đúng cả 4 |
| Tutorial | 4 bước đúng thứ tự; đi chệch hướng dẫn vẫn ring đúng cây; **14 ngôn ngữ × 4 câu, mọi câu ≤ 5 từ** |
| Layout dọc 390×844 | bàn cao nhất 368×644, không tràn, không đè header |
| Cover | 1920×1080 / 800×1200 / 800×800 (đọc từ byte 16–23 file PNG) |
| `zip-common §5` | `OK — game.json hợp lệ` |

## Tối ưu để không lag ở máy yếu

Headless **không đo được wall-clock** (`--virtual-time-budget` đóng băng
`performance.now()`). Nên đo thẳng **cấu trúc gây giật**:

| Chỉ số (bàn 7×13, 17 cây) | Kết quả |
|---|---|
| Layer compositor bị ghim bởi `will-change` | **1** (đúng một cái: thanh lượt) |
| `localStorage.setItem` / lần chạm | **0.02** |
| Forced sync layout / lần chạm | **0.00** |
| Animation còn chạy lúc rảnh (sau tutorial) | **0** |
| DOM nodes / hình SVG | 1.176 / 748 |

**DOM nodes cao hơn hai game trước** (Bridge Islands 314) vì mỗi ô giữ sẵn cả cây,
lều và dấu cỏ, chỉ ẩn/hiện bằng class. Đó là đánh đổi có chủ ý: **chạm không bao
giờ tạo hay xoá node**, nên độ trễ chạm không phụ thuộc vào việc dựng DOM. Node
tĩnh chỉ tốn bộ nhớ và một lần paint.

Bốn bài học từ hai game trước được áp thẳng từ đầu: không `will-change` trên phần
tử lặp lại; gom ghi `localStorage` 500ms + flush ở teardown; dựng node một lần rồi
chỉ đổi class; và **`visibility: hidden` KHÔNG dừng CSS animation** → coach mark
phải `animation: none` khi ẩn (có phép đo riêng ngay lúc vừa load, vì đo sau khi
tutorial kết thúc sẽ ra 0 và **nhìn như đạt** mà không kiểm được gì).

**Chưa đo ms trên máy thật.** Các con số trên là đếm chính xác, không phải suy diễn.

### Vòng tối ưu thứ hai — sửa 6 lỗi thật trên thiết bị di động

Chạy lại toàn bộ trên **11 kích thước màn hình** (dọc + ngang) và so từng phép đo
với bản đã commit. Bộ test được **kiểm tra ngược trên bản cũ**: 12/21 assertion
FAIL ở bản cũ, 0/21 ở bản mới — nghĩa là test thật sự bắt được lỗi, không phải test
luôn xanh.

| Lỗi | Bản cũ | Bản mới |
|---|---|---|
| **Font chặn first paint.** `<link rel=stylesheet>` tới Google Fonts chặn khung hình đầu tiên. Trong WebView **mất mạng** (máy bay, hết data, wifi cổng đăng nhập) đó là màn hình đen tới khi request bỏ cuộc. | chặn | tải qua `media="print"` + `onload` → **không chặn**, game vẽ ngay bằng `system-ui` |
| **Bàn cờ bị cắt.** Sàn `Math.max(26, …)` đẩy bàn vượt khung `justify-content:center`; phần tràn **phía trên** không với tới được. | **4/33** bố cục bị cắt (ngang, chia đôi màn hình) | **0/33** |
| **Banner tutorial đè lên bàn cờ** và ăn mất cú chạm ở hàng dưới cùng. | đè tới **61px**, 43 assertion FAIL | chừa chỗ, **gap 22–106px**, 0 FAIL |
| **Hai ngón cùng chạm** → 2 ô đổi trạng thái, mất lượt oan. | có | chặn pointer không phải `isPrimary` |
| **Pinch-zoom / double-tap zoom** (iOS bỏ qua `user-scalable=no` từ iOS 10) → phóng to kẹt ở góc bàn cờ. | có | `touch-action: none` + chặn `gesturestart` |
| **Nhấn giữ** mở menu hệ thống, đóng nó ăn mất cú chạm kế. | có | chặn `contextmenu` |

Thêm: `body` được ghim `position: fixed` + `overscroll-behavior: none` (iOS bỏ qua
`overflow: hidden` khi rubber-band); `resize`/`orientationchange`/`visualViewport`
gộp vào **một** rAF thay vì mỗi sự kiện một forced layout; `onAppResume` + resume
`AudioContext` khi quay lại (trước đó về từ app switcher là **mất tiếng vĩnh viễn**).

### Một giả định sai, đã đo lại

Ban đầu ghi rằng vẽ lại toàn bàn tốn *"~313 style invalidation mỗi lần chạm"*.
**Sai.** `classList.toggle(tok, force)` theo spec **bỏ qua bước ghi attribute** khi
phần tử đã ở đúng trạng thái. `MutationObserver` xác nhận: **2.2 lần ghi class mỗi
cú chạm, cũ và mới y hệt nhau**.

Cái thật sự tốn là **số lời gọi DOM**, và nó vẫn đáng sửa:

| | Lời gọi `classList.toggle` / lần chạm |
|---|---|
| Trước | **316,0** |
| Sau (diff bằng cache byte) | **5,3** |

Cache có thể lệch khỏi DOM nên phải chứng minh là không: **2.203 ảnh chụp DOM**
(mỗi cú chạm của bot 60 màn + 1.500 cú fuzz) được so với một lần tính lại từ đầu →
**0 lần lệch**.

## 📋 Backlog

- Nút **Undo** — hiện phải bấm vòng để gỡ lều, tốn 1 lượt.
- Nút **Hint**: solver đã có sẵn trong game, đủ để chỉ ra **một ô bị ép** kế tiếp.
- Tự động rải cỏ quanh lều vừa dựng (nhiều bản Tents có tuỳ chọn này).
- Đo FPS thật trên máy Android yếu.
