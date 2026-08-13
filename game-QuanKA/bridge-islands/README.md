# Bridge Islands — QuanKA

> **Puzzle logic bắc cầu nối đảo** (thể loại "Hashi / Hashiwokakero") | HTML5 single-file | 60 màn cố định

## Thông tin game

| Trường | Giá trị |
|---|---|
| **Package** | `com.falcon.bridgeislands` |
| **Engine / Version** | HTML5 single-file · 1.0.0 |
| **Category** | PUZZLE |
| **Loại** | Game **có level** — 60 màn, tính điểm theo **run**, `victory` ở màn cuối |
| **Kích thước** | `index.html` ~97 KB (không asset ngoài; 30 KB trong đó là icon bàn tay tutorial dạng base64) |
| **Tham khảo** | Hashiwokakero (Nikoli) |

## Vì sao chọn thể loại này

Đối chiếu **~30 game đang live trên app** với sheet `MiniGame-TTS-PTIT.xlsx`
(tab `Game`, 90 dòng) thì hầu hết thể loại casual phổ thông đã bị chiếm:

| Định làm | Vì sao bỏ |
|---|---|
| Ball Sort / Water Sort | cả hai đã live |
| Bubble Shooter, Block Puzzle | đã live (team Thắng) |
| Twodots | app đã có Connect Dots + Number Connect + Animal Lines + Tap Match; QuanKA đã có 2 game onet |
| Merge Plus / Roll Merge / Merge the Jelly | app đã có **7 game merge/2048**, pipeline còn 3 cái nữa |
| Snake io | sheet đã có **5 dòng snake**, 2 dòng bị Reject đúng vì *"team khác làm rồi"* |

**Hashi không xuất hiện ở cả app lẫn sheet.** Ngoài chỗ trống, nó còn hợp vì tái
dùng đúng thế mạnh đã chứng minh ở Energy Loops: **dựng lời giải trước rồi suy
ngược ra đề**, nên tính giải được là hệ quả của cách sinh chứ không phải thứ phải
đi dò. Thêm nữa: chỉ tap (không kéo), vẽ hoàn toàn bằng SVG, không physics,
không `requestAnimationFrame`, không asset ngoài.

## Gameplay

- Lưới ô; một số ô là **đảo** mang số 1–8. Nối các đảo bằng **cầu** ngang/dọc.
- **Chạm đảo A → chạm đảo B**: số cầu giữa hai đảo chạy vòng `0 → 1 → 2 → 0`.
- Luật: giữa 2 đảo tối đa **2 cầu**; cầu **không cắt nhau**, không xuyên qua đảo;
  số trên đảo = **đúng** tổng số cầu cắm vào nó; và **toàn bộ đảo phải thành MỘT
  khối liên thông duy nhất**.
- **Ngân sách lượt** cho mỗi màn. Hết lượt mà chưa xong = **THUA = hết run** →
  app hiện popup thua, bấm Retry là **chơi lại từ màn 1** (giữ BEST).
- **SCORE cộng dồn cả run**: `50 × số màn + 25 × số lượt còn thừa`.
- **Sao**: `moves ≤ par` → 3★, `≤ par×1.4` → 2★, còn lại 1★.

## Điều kiện thắng cần ĐÚNG HAI phép kiểm — đây là bẫy của thể loại

Energy Loops quyết định thắng chỉ bằng **một** lần BFS, vì các mảnh được cắt từ
rừng khung nên "mọi ô sáng" tự ép "mọi đầu nối khớp".

**Hashi không có tính chất đó.** Một thế cờ có thể thoả mãn **mọi** con số mà vẫn
tách thành hai cụm rời nhau. Ví dụ nhỏ nhất: bốn đảo số `1 1 1 1` xếp thẳng hàng —
nối cặp ngoài với nhau và cặp trong với nhau thì mọi con số đều đủ, nhưng bàn cờ
nằm thành hai nửa:

```
①────①   ①────①      ← mọi số đều đủ, nhưng KHÔNG phải lời giải
```

Nên thắng phải kiểm **cả hai**:

1. bậc của mọi đảo **bằng đúng** số ghi trên nó, **và**
2. một lần BFS từ đảo bất kỳ chạm được **toàn bộ** đảo.

> Điều này **không** mâu thuẫn với tính duy nhất của lời giải: duy nhất nghĩa là
> chỉ có một *lời giải Hashi hợp lệ*, mà liên thông là một phần của định nghĩa
> hợp lệ — thế cờ "đủ số nhưng rời" đơn giản không phải lời giải.

Bộ test đánh thẳng vào chỗ này: **vét cạn toàn bộ thế cờ hợp lệ** của các màn nhỏ
(3^slots, đã lọc bỏ những thế có cầu cắt nhau) để đi tìm đúng loại thế cờ đó, rồi
bắt `isSolved()` phải từ chối. Quét 22 màn nhỏ tìm được **đúng 1** thế như vậy —
hiếm, nhưng có thật, và đó là lý do phép kiểm thứ hai tồn tại.

## Sinh màn — dựng lời giải trước, suy ngược ra đề

1. Rải đảo: mỗi đảo mới **gắn vào cụm đã có** bằng 1–2 cầu; ô đích phải trống,
   **không kề trực tiếp** đảo nào (để mọi cầu luôn có ≥1 ô trống mà vẽ), và cả
   nhịp cầu phải không xuyên đảo, không đè cầu cũ.
2. Thêm vài cầu **tạo chu trình** ở chỗ hợp lệ → đề bớt dạng cây.
3. **Số trên đảo = bậc của nó.**

→ Liên thông và không-cắt-nhau **đúng theo cách dựng**, không cần kiểm lại.

### Solver vừa lọc đề, vừa chứng minh tính duy nhất

Đề vừa sinh được đưa cho một solver **chỉ suy luận, không bao giờ đoán**:

| Quy tắc | Nội dung |
|---|---|
| **Bão hoà** | đảo cần `r` cầu, tổng sức chứa còn lại là `C`; `r == C` → cắm hết |
| **Sàn** | với mỗi hướng `n`: buộc cắm ít nhất `r − (C − c_n)` cầu về `n` (dạng tổng quát, bao trùm luôn ca kinh điển `r == C − 1`) |
| **Cấm cô lập** | cây cầu nào khép kín một nhóm đã đủ số mà nhóm đó chưa gồm **toàn bộ** đảo thì bị cấm (hay gặp: hai đảo số 1 không được nối nhau) |
| **Cắt nhau** | cầu đã chắc chắn có thì mọi cầu cắt nó bị loại |

Lặp tới điểm bất động. **Chỉ nhận màn nếu solver đi tới lời giải đầy đủ**, và điều
đó cho luôn hai tính chất:

- **Giải được không cần đoán** — mọi bước đều bị ép.
- **Lời giải duy nhất** — mỗi cây cầu đặt xuống là lựa chọn duy nhất tại thời điểm
  đó, nên không thể tồn tại lời giải thứ hai. **Không cần DFS đếm nghiệm.**

Test còn so lời giải solver suy ra với lời giải lúc sinh: nếu khác nhau thì tức là
solver **không đúng đắn** (chứ không phải đề mơ hồ). Qua 60 màn, sai khác = **0**.

### Bảng seed nướng sẵn

Seed thắng cho từng màn được tìm **offline** rồi nướng vào `SEED_OFF`:

- **Vào màn chỉ chạy generator ĐÚNG 1 LẦN**, không dò. Màn 60 vốn cần 46 lần thử —
  trên máy yếu là một cú khựng thấy được.
- Seed còn được chọn để **`par` tăng đều 5 → 29, 0 bước lùi**. Để generator tự do
  thì màn 37 ra **dễ hơn** màn 31 (par 17 so với 23) — lỗi độ khó mà playtest sẽ
  bắt ngay.
- Vẫn còn nhánh dò dự phòng: nếu sau này sửa generator làm seed cũ hỏng, game tự
  tìm seed khác thay vì treo màn. Test bắt cả 60 seed phải **trúng ngay lần đầu**.

## Ramp 60 màn

| Màn | Lưới | Số đảo |
|---|---|---|
| 1–3 | 5×5 | 5 |
| 4–8 | 5×6 | 6 |
| 9–14 | 6×6 | 8 |
| 15–22 | 6×7 | 10 |
| 23–30 | 6×8 | 12 |
| 31–38 | 7×8 | 14 |
| 39–45 | 7×9 | 16 |
| 46–52 | 7×10 | 19 |
| 53–60 | 7×10 | 22 |

Tối đa **7 cột** — theo bài học màn dọc của fruit-connect.

## Ngân sách lượt

```js
movesLimitFor(par, lv):  lv≤9 →3.4 | ≤20 →2.8 | ≤35 →2.5 | ≤50 →2.3 | else 2.2
                         return max(par + 4, ceil(par * k))
```

Hệ số **rộng hơn Energy Loops** vì ở Hashi **sửa một cây cầu sai tốn tới 3 nhịp**
(`1 → 2 → 0` mới về rỗng), trong khi xoay sai một ô ở Energy Loops thường chỉ tốn
1 nhịp. Lấy nguyên hệ số cũ sẽ bóp nghẹt người chơi ở màn to.

Đo thực tế: **người chơi hoàn hảo chỉ dùng 45% ngân sách** ở màn nặng nhất — phần
còn lại là chỗ cho sai sót.

## Giao diện

- **Header** bê nguyên khối `#topbar` của Energy Loops (vốn lấy từ animal-connect,
  đúng cỡ mentor yêu cầu): `Back | SCORE ★ | BEST ♛ | Volume`, hàng dưới
  `LEVEL n · thanh lượt còn lại · số lượt còn lại`.
- **Bàn chơi là một `<svg>` duy nhất.** Mọi đảo và mọi *chỗ có thể bắc cầu* được
  tạo **một lần** lúc vào màn; sau đó chạm chỉ **đổi class**, không tạo/xoá node
  nào → một cú chạm không thể ép layout.
- Mỗi chỗ bắc cầu giữ sẵn 4 đường: nét đứt gợi ý, cầu đơn, và hai nửa của cầu đôi;
  class của nhóm quyết định đường nào hiện.

### Tách tín hiệu màu (sửa sau khi soi ảnh chụp)

Bản đầu tô **viền đảo xanh lơ** cho "đã nối vào cụm chính" và **viền xanh lá** cho
"đủ số" — hai màu gần như trùng nhau trên cùng một nét, nên nhìn ảnh chụp **không
phân biệt được đảo mới xong một nửa với đảo đã hoàn thành**.

Sửa bằng cách tách hai tín hiệu sang **hai loại phần tử khác nhau**:

| Tín hiệu | Thể hiện ở |
|---|---|
| Đảo **đủ số** | thân + số của **đảo** chuyển xanh lá |
| Đảo **quá số** | thân + số của **đảo** chuyển đỏ (báo sai ngay, không đợi hết lượt) |
| **Cụm liên thông** lớn nhất | **cầu** chuyển xanh lơ |

Cách này còn đúng bản chất hơn: liên thông là tính chất của **cầu**, không phải của
đảo. Và nó làm luật "phải thành một khối" **hiện ra bằng mắt** thay vì phải giải
thích bằng chữ.

### Input: chỉ tap, không kéo

Chạm đảo → các đảo nối được (gần nhất mỗi hướng, đường thông) hiện **nét đứt gợi
ý** → chạm một trong số đó để đổi số cầu.

**Bỏ chọn sau mỗi lần nối.** Bản đầu giữ nguyên đảo đang chọn để bấm thêm cho
nhanh thành cầu đôi, nhưng như vậy mọi cú chạm tiếp theo vào **hàng xóm** của đảo
đang chọn sẽ *bắc cầu* thay vì *chọn* — người chơi với tay sang đảo kế bên là mất
oan một lượt. Số lượt tiêu tốn **y hệt nhau ở cả hai cách** (một lượt tính theo
thay đổi trên bàn, không tính theo cú chạm), nên chọn cách an toàn.

### Tutorial — làm lại cho mạch lạc

**Bản đầu gây rối và đã bị bỏ.** Ba lỗi:

- Bước 1 viết *"Tap two islands"* nhưng bàn tay chỉ gõ **một** đảo — và cú chạm đó
  chỉ *chọn* chứ chưa vẽ cầu, nên người mới chạm xong thấy **không có gì xảy ra**.
- Bước 2 *"Match the number"* chỉ vào đảo có số lớn nhất, **chưa có cây cầu nào** →
  khớp với cái gì thì không có gì trên màn hình chứng minh.
- Bước 3 chỉ có chữ, không hình.

Bản mới: **4 bước, mỗi bước một hành động và một kết quả nhìn thấy được ngay**.

| Bước | Visual | Chữ | Chuyển bước khi |
|---|---|---|---|
| 1 | bàn tay trên **một** đảo (đảo có cầu trong lời giải) | "Tap an island" | người chơi **chọn** một đảo bất kỳ |
| 2 | bàn tay **chuyển sang đảo bạn**; nét đứt gợi ý tự hiện quanh đảo đang chọn | "Tap another" | **một cây cầu hiện ra** |
| 3 | vòng ripple trên đảo **vừa đủ số** (đang xanh lá), bỏ bàn tay | "Number = bridges" | hết 2,6s |
| 4 | — | "Connect them all" | hết 3s → đóng |

Hai điểm mấu chốt:

- **Bước 3 chỉ vào đảo mà người chơi vừa làm cho đủ số.** Câu "Số = số cầu" được
  đọc ra từ một đảo xanh lá có đúng số cầu ngay trước mắt, chứ không phải nói suông.
- **Không chặn thao tác, và bám theo người chơi.** Chạm đảo khác với chỗ bàn tay
  chỉ → bàn tay **chỉ lại** sang một đảo bạn của **đảo vừa chọn**; bỏ chọn → lùi về
  bước 1. Không bao giờ có cảnh bàn tay chỉ vào một đảo không còn liên quan.

Vòng ripple để **màu trắng**, không dùng xanh lơ của theme, vì nó phải nổi trên cả
đảo xanh lá (đủ số), cầu xanh lơ, vòng chọn vàng lẫn đảo đỏ (quá số).

Hỗ trợ `?tutorial=1` (chơi lại tutorial — quy ước mentor có kiểm, thấy ở tab
`Feecback` dòng r18, dù **không** nằm trong `.claude`) và `?reset=1` (xoá sạch tiến
độ đã lưu).

## Tuân thủ quy ước chung

- **game-common** — `sendMessage` đủ 5 trường; `level` 1-indexed; thắng/thua **đều**
  `showModal:true`; `victory` ở màn 60; `quit` từ nút Back; `ads` mỗi 5 màn; 3
  callback `onAppPause`/`onNextLevel`/`onRetryLevel`; `waitForNativeInjection` 200ms;
  `statusBarHeight`/`currentLevel`/`data`/`language` đúng thứ tự ưu tiên.
- **popup-common §1** — game theo level ⇒ **KHÔNG tự vẽ popup kết quả**, để app lo.
  Đây là **khác biệt có chủ ý so với Energy Loops** (game đó tự vẽ popup thua theo
  yêu cầu riêng); Bridge Islands theo đúng tiền lệ animal-connect / fruit-connect.
  Cái duy nhất game tự vẽ là **màn hình sao** — thứ app không thể biết.
- **Lưu 2 tầng (§1.2)** — nhẹ qua `save_data`: `{lv, st, m, tut, best, sc}`; snapshot
  ván dở (`blv`, chuỗi số cầu, `mv`) chỉ nằm ở `localStorage` (`bridge_islands_v1`),
  ghi gộp 500ms + flush thật ở `onAppPause` / `visibilitychange` / `pagehide`.
- **i18n** — bảng `I18N` inline, 23 ngôn ngữ cho tutorial; nhãn UI giữ tiếng Anh.
- ℹ️ `button-common.md` / `background-common.md` vẫn **không tồn tại trong repo** →
  `btn3d` lấy từ energy-loops như các game trước.

## Kiểm chứng

Chạy bằng headless Edge (máy không có node).

| Hạng mục | Kết quả |
|---|---|
| Tổng số assertion | **12.917 (bot) + 45 (tutorial) + 21 (perf/khôi phục/native) — 0 fail** |
| Tutorial: đi đúng đường hướng dẫn | 4 bước đúng thứ tự; bước 2 luôn chỉ vào đảo **có chỗ bắc cầu** tới đảo đang chọn; bước 3 luôn chỉ vào đảo **đã đủ số** |
| Tutorial: đi chệch hướng dẫn | chạm đảo khác → bàn tay chỉ lại đúng; bỏ chọn → lùi về bước 1, không mất coach mark |
| Tutorial: độ dài chữ | 12 ngôn ngữ, **mọi câu ≤ 5 từ** |
| Seed nướng sẵn trúng ngay lần đầu | **60/60** |
| Solver suy ra được lời giải (no-guess + duy nhất) | **60/60** |
| Solver ra lời giải **khác** lúc sinh (⇒ solver sai) | **0** |
| Đường cong `par` | 5 → 29, **0 bước lùi** |
| Bot giải trọn màn qua đúng đường input thật | **60/60** |
| Bẫy thể loại: "đủ số nhưng rời cụm" bị từ chối | **1/1** tìm thấy khi vét cạn 22 màn nhỏ |
| Hình học: đảo kề nhau / cầu cắt nhau / cầu xuyên đảo / bậc ngoài 1–8 | **0 vi phạm** |
| Cầu cắt nhau bị chặn khi người chơi cố vẽ | **7/7** |
| Fuzz | **4.139** cú chạm, 0 lần vượt ngân sách, 0 số cầu sai miền |
| Lỗi JS | **0** |
| Khôi phục ván dở | đúng màn, **khớp từng cây cầu**, đúng số lượt |
| Input native `?statusBarHeight=44&currentLevel=7&language=vi&tutorial=1` | đúng cả 4 |
| Layout dọc 390×844 | không tràn ngang, không đè header |
| Cover | 1920×1080 / 800×1200 / 800×800 (đọc từ byte 16–23 của file PNG) |
| `zip-common §5` | `OK — game.json hợp lệ` |

## Tối ưu để không lag ở máy yếu

Headless **không đo được wall-clock** (`--virtual-time-budget` đóng băng
`performance.now()` trong lúc chạy code đồng bộ, mọi con số ra `0.00ms`). Nên đo
thẳng **cấu trúc gây giật**, thứ đếm được chính xác:

| Chỉ số (bàn 7×10, 22 đảo) | Kết quả |
|---|---|
| Layer compositor bị ghim bởi `will-change` | **1** (đúng một cái: thanh lượt) |
| `localStorage.setItem` / lần chạm | **0.00** |
| Forced sync layout / lần chạm | **0.00** |
| Animation còn chạy lúc rảnh (sau tutorial) | **0** |
| DOM nodes / hình SVG | 314 / 200 |

Bốn điều rút từ Energy Loops được áp thẳng từ đầu:

1. **Không `will-change` trên phần tử lặp lại.** Nó ghim một layer cho **mỗi** phần
   tử, thường trú trong bộ nhớ GPU cả phiên.
2. **`localStorage.setItem` là I/O đĩa đồng bộ** → gom 500ms, flush thật ở
   `onAppPause` / `visibilitychange` / `pagehide`.
3. **Dựng node một lần, sau đó chỉ đổi class** — chạm không tạo/xoá node.
4. **`visibility: hidden` KHÔNG dừng CSS animation** → coach mark phải `animation:
   none` khi ẩn.

> Điểm 4 suýt lọt lần nữa: phép đo đầu tiên báo *"0 animation trong lúc tutorial"*
> và nhìn như đã đạt, nhưng thật ra lúc đo thì tutorial **đã kết thúc rồi** — nhãn
> sai chứ không phải kết quả tốt. Đã thêm phép đo riêng ngay khi vừa load: bắt
> coach mark **phải đang chạy** lúc hiện, và **phải tắt hẳn** lúc ẩn.

**Chưa đo ms trên máy thật.** Những con số trên là đếm chính xác (layer, I/O,
layout, animation), không phải suy diễn — nhưng FPS trên một máy Android yếu thì
phải chạy thử mới biết.

## 📋 Backlog

- Nút **Undo** (hoàn tác 1 nước) — thể loại này người chơi hay muốn thử-sai; hiện
  phải bấm vòng `1 → 2 → 0` để xoá, tốn 2 lượt.
- Nút **Hint**: solver đã có sẵn trong game, đủ để chỉ ra **một cây cầu bị ép**
  tiếp theo mà không cần lộ cả lời giải.
- Cân nhắc **đánh dấu X** lên chỗ chắc chắn không có cầu (quy ước quen của Hashi).
- Đo FPS thật trên máy Android yếu.
