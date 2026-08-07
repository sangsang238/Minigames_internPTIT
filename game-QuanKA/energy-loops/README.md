# Energy Loops — QuanKA

> **Puzzle xoay ô nối mạch điện** (thể loại "Net / Loops") | HTML5 single-file | 60 màn cố định

## Thông tin game

| Trường | Giá trị |
|---|---|
| **Package** | `com.falcon.energyloops` |
| **Engine / Version** | HTML5 single-file · 1.0.0 |
| **Category** | PUZZLE |
| **Loại** | Game **có level** — 60 màn, tính điểm theo **run**, `victory` ở màn cuối |
| **Kích thước** | `index.html` ~50 KB (không asset ngoài, không sprite) |
| **Tham khảo** | `Energy: Anti-Stress Loops` |

## Gameplay

- Lưới ô, mỗi ô là một đoạn dây có đầu nối ở 0–4 cạnh. **Chạm ô → xoay 90° theo
  chiều kim đồng hồ.**
- Mục tiêu: xoay sao cho **mọi ô đều nhận điện** từ ô **lõi** (hình thoi). Điện lan
  ra từ lõi; ô nào nối tới được thì sáng lên.
- **Ngân sách lượt xoay** cho mỗi màn. Hết lượt mà chưa xong = **THUA = hết run**
  → app hiện popup, bấm Retry là **chơi lại từ màn 1** (giữ BEST).
- Từ **màn 31** có **2 lõi** (2 mạng điện, 2 màu: bạc hà + hổ phách), cứ mỗi màn
  thứ 3 lại quay về 1 lõi để nhịp độ không đơn điệu.
- **SCORE cộng dồn cả run**: `50 × số màn + 25 × số lượt còn thừa`. Tiết kiệm lượt
  là cách duy nhất để điểm cao → thưởng đúng kỹ năng mà game này đo.
- **Sao**: `moves ≤ par` → 3★, `≤ par×1.4` → 2★, còn lại 1★.
- Ô **chữ thập** (4 đầu nối) xoay 90° vẫn y hệt chính nó → game **không cho nó
  xoay** (chỉ nảy nhẹ báo đã nhận chạm) và **không tính lượt**. Nếu cho xoay thì
  người chơi mất 1 lượt mà màn hình không đổi gì — trông như game bị lỗi.

### Ngân sách lượt (`movesLimitFor`)

Theo feedback mentor 2026-08-06: *"rào bằng số lượt moves, nma cho rộng rãi ra,
tầm lv10 trở lên rồi bắt đầu khó"*.

| Màn | Hệ số so với `par` |
|---|---|
| 1–9 | **3.0×** (gần như không thể thua) |
| 10–20 | 2.4× |
| 21–35 | 2.1× |
| 36–50 | 1.9× |
| 51–60 | 1.8× |

Có **sàn cứng `par + 3`**: dù `par` nhỏ tới đâu thì ngân sách vẫn luôn dư ít nhất
3 lượt, nên **không màn nào có thể phát ra ở trạng thái không thắng nổi**. Bất biến
này được test khẳng định cho cả 60 màn.

### Vì sao "mọi ô sáng" là điều kiện thắng ĐỦ (không cần kiểm tra gì thêm)

Đây là điểm quan trọng nhất của thiết kế, vì nó biến việc kiểm tra thắng thành
**một lần BFS duy nhất**:

- Bàn được cắt ra từ một **rừng khung (spanning forest)** gồm `k` cây → có đúng
  `N − k` cạnh, tức `2(N − k)` **đầu nối**.
- Muốn cả `N` ô đều sáng thì đồ thị đang-có-điện phải phủ hết `N` đỉnh với tối đa
  `k` thành phần → cần **ít nhất** `N − k` cạnh khớp.
- Mà số cạnh có thể khớp **nhiều nhất** cũng chỉ `N − k`.

→ "Tất cả ô sáng" **ép** mọi đầu nối phải khớp, **không đầu nào được chĩa vào
tường**. Không cần quét riêng "còn đầu thừa không".

> Mệnh đề này **được test kiểm chứng bằng một bản BFS viết độc lập**: qua 4800
> nước đi ngẫu nhiên, chưa bao giờ có trạng thái "sáng hết mà vẫn còn đầu thừa".

### Sinh màn — luôn có lời giải, chứng minh được

1. **Randomized-Prim** sinh rừng khung `k` cây trên lưới `w×h`.
   Dùng Prim (bốc ngẫu nhiên trong frontier) chứ **không** dùng DFS: DFS ra hành
   lang dài ngoằn ngoèo, Prim ra cây ngắn nhiều nhánh — tức **nhiều bóng đèn**,
   nhìn mới ra "mạch điện".
2. Mask mỗi ô = tập cạnh cây kề nó (bậc 1 = bóng đèn, 2 = thẳng/khuỷu, 3 = chữ T,
   4 = chữ thập).
3. Lõi = các ô hạt giống của Prim. Màn 1 lõi: đặt **ngẫu nhiên trong lòng bàn**
   (không sát biên, **không phải chính giữa** — 40 màn mà lõi luôn nằm giữa thì
   nhìn ra ngay là khuôn mẫu).
4. Xoay ngẫu nhiên mỗi ô. Lời giải luôn là `rot = 0` với mọi ô → **không thể sinh
   ra màn không giải được**. Nếu lỡ trúng trạng thái đã-giải-sẵn thì ép lệch 1 ô.

**Toàn bộ dùng RNG có seed (mulberry32 theo chỉ số màn)** → màn 7 là *cùng một câu
đố* trên mọi máy. Không có seed thì `par`, sao và ngân sách lượt đều vô nghĩa.

### Ramp 60 màn (tối đa **7 cột** — bài học màn dọc từ fruit-connect)

| Màn | Lưới | Ghi chú |
|---|---|---|
| 1–3 | 3×3 | tutorial ở màn đầu tiên của phiên chơi |
| 4–8 | 4×4 | |
| 9–14 | 4×5 | |
| 15–20 | 5×5 | |
| 21–25 | 5×6 | |
| 26–30 | 5×7 | |
| 31–37 | 6×7 | **2 lõi** từ đây (trừ màn chia hết cho 3) |
| 38–45 | 6×8 | |
| 46–52 | 7×8 | |
| 53–60 | 7×9 | 63 ô — bàn lớn nhất |

## Giao diện

### Header — cỡ lấy theo `index(exampleforheader).html`

Bản đầu copy số đo header của **fruit-connect**, và bản đó **nhỏ hơn file mẫu
mentor gửi ~20%**. Đo tại bề ngang 390px:

| | fruit-connect (bản đầu) | file mẫu (bản hiện tại) |
|---|---|---|
| Nhãn pill | 10.0px | **12.5px** |
| Số trong pill | 17.9px | **19.5px** |
| Padding dọc pill | 9px | **16px** |
| Cao pill | ~43px | **~55px** |
| Nút Back/Volume | 42px | **50px** (≈0.92× cao pill, đúng tỉ lệ file mẫu) |

→ Cảm giác "hơi bé" của mentor là **so với file mẫu**, không phải so với
fruit-connect. Cả 3 con số trên đều có **assertion trong test** để không ai vô tình
thu nhỏ lại.

**Bố cục** (theo fruit-connect / animal-connect):

- Hàng trên: `Back` · **SCORE** (⚡ bạc hà) · **BEST** (♛ vàng) · `Volume`
- Hàng dưới: `LEVEL n` · **thanh lượt còn lại** · **số lượt còn lại**
- Thanh lượt vàng khi còn ≤1/3, **đỏ + nháy** khi còn ≤1/6.
- **Không có nút Restart** (mentor yêu cầu bỏ): có ngân sách lượt rồi thì
  trạng thái thua chính là nút reset, và Retry thuộc về popup của app.

### Bàn chơi

- **Theme phẳng** (theo tiền lệ runic-blaze): nền tối `#0D1420` + lưới mạch in sẵn
  bằng `linear-gradient` (compositor vẽ **một lần**, không bao giờ vẽ lại). **0
  gradient, 0 blur, 0 `filter`/`backdrop-filter`** ở bất kỳ đâu.
- **Hình học dây** (sửa theo feedback "lồi ra khỏi khung ô / lồi khối u"):
  - Mỗi nhánh dừng **sớm nửa nét vẽ** (`15/2 = 7.5` đơn vị) so với mép ô, để đầu
    bo tròn **đáp đúng lên mép** thay vì thò ra ngoài. `overflow` của `<svg>` để
    **cắt** chứ không `visible` như bản đầu.
  - **Bỏ hẳn chấm tròn `r=9` ở tâm ô.** Nó chính là "khối u" ở ngã 3 / ngã 2: nét
    dây bán kính 7.5 mà chấm 9 → luôn phình ra 1.5 đơn vị. Các đầu bo tròn tự chồng
    lên nhau ở tâm thành một đĩa **đúng bằng** bán kính nét, nên mối nối đã liền
    sẵn, không cần đắp thêm gì.
  - Có **assertion đo hình học thật**: với mọi ô, mọi nhánh mask khai báo phải
    **đáp đúng lên mép** (sai số < 0.9px) và mọi cạnh không có nhánh phải **cách
    mép > 2px**; không nét nào được vẽ ra ngoài ô của nó.
  - Khe giữa 2 ô giữ **3px**: đã dựng thử bản khe 0px cho dây liền mạch tuyệt đối,
    nhưng lúc đó nền các ô dính thành một mảng, mất ranh giới ô — mà đây là game
    **chạm từng ô**, ranh giới ô phải đọc được.
- **Không có vòng `requestAnimationFrame` nào** trong cả game:
  - Xoay ô = `transform: rotate()` + transition → compositor lo, không repaint.
  - **Điện lan ra** = gán `transition-delay = 105ms + depth × 22ms` theo vòng BFS.
    Ra đúng hiệu ứng "điện chạy loang" mà **không tốn một frame nào** cả.
    105ms ≈ 55% thời lượng xoay → đèn sáng đúng lúc dây khớp vào vị trí.
  - Thanh lượt dùng `scaleX` (compositor), không dùng `width` (relayout mỗi chạm).
- **Chỉ đụng vào ô có đổi trạng thái** khi render (`B.shown`), không quét class
  toàn bàn mỗi nước — đúng bài học "224 thao tác class mỗi lần vào màn" của
  fruit-connect.
- **Tap nhanh không bị chặn**: mỗi lần chạm chỉ cộng dồn góc đích (`B.vis`), ô
  luôn quay **xuôi** chiều kim đồng hồ, không bao giờ giật ngược.
- **SFX tổng hợp Web Audio** (không file ngoài, không nhạc nền): click xoay, tiếng
  "điện dâng", hợp âm rải khi xong màn và hợp âm **đi xuống** khi hết lượt. Tiếng
  điện dâng là **một** giọng lướt tần số cho cả đợt sáng, **không phải một
  oscillator mỗi ô** — một phản ứng dây chuyền có thể thắp 30 ô cùng lúc, 30 giọng
  chồng lên nghe như nhiễu.
- **Tutorial 3 bước**, **không chặn thao tác**, dùng ô nhấp nháy thay mũi tên
  (bài học playtest fruit-connect). Chạy ở **màn đầu tiên của phiên chơi**, không
  phải riêng màn 1 — app có thể thả người chơi mới thẳng vào màn 7 qua
  `currentLevel`.

## Tuân thủ quy ước chung

- ✅ **game-common.md** — `sendMessage` đủ 5 trường, `level` 1-indexed;
  `game_result` **win** (`showModal:true`) và **lose** (hết lượt, `showModal:true`);
  **`victory`** ở màn 60 (§1.5, không kèm `game_result`); Back → `quit`;
  `ads` mỗi 5 màn (§1.7); 3 callback `onAppPause`/`onNextLevel`/`onRetryLevel`;
  đọc đủ `statusBarHeight` (URL ưu tiên) / `currentLevel` / `data` / `language`
  (window ưu tiên) + `waitForNativeInjection` 200ms; font Google Sans Flex qua
  `--ui-font` (ngoài Latinh → `system-ui`); reset CSS + tắt tap-highlight.
- ✅ **§1.2 — ô BEST không chạy live.** `best` được ghi **ngay** lúc đổi (kill giữa
  chừng không mất kỷ lục), nhưng **ô BEST trên HUD giữ kỷ lục cũ suốt run** và chỉ
  refresh khi **hết run** (thua / xong màn 60) và khi **vào run mới** — để người
  chơi nhìn thấy mình đang vượt qua nó. Có assertion riêng.
- ✅ **popup-common.md** — game theo level nên **không tự vẽ popup kết quả**.
  Thắng màn chỉ hiện **3 ngôi sao** trong 1.1s: không tiêu đề, không lớp phủ tối,
  không nút. Sao là thứ **duy nhất** app không thể tự biết.
- ✅ **Lưu 2 tầng (§1.2)** — `save_data` **nhẹ**: `{lv, st, m, tut, sc, best}`.
  Ván dở (`blv` + chuỗi góc xoay + số lượt) **chỉ nằm ở `localStorage`**
  (`energy_loops_v1`), ghi sau mỗi nước vì nó miễn phí trong WebView.
  - `lv` là **"đang chơi dở ở đâu"**, không phải "đã đi xa nhất tới đâu" — nếu
    lấy mốc xa nhất thì chơi lại màn cũ rồi thoát giữa chừng sẽ **mất luôn bàn
    đang dở**. Mốc hoàn thành không mất: chữ số sao của mỗi màn chính là nó.
  - Snapshot gắn nhãn `blv` (màn của nó) nên **không bị dán nhầm** sang câu đố
    khác, và snapshot đã **hết lượt** thì bị bỏ (nếu không sẽ khôi phục vào một
    bàn chết, không còn nước đi hợp lệ nào).
- ✅ **zip-common.md** — single-file, `game.json` đủ field & pass script check
  (chạy bằng PowerShell vì máy **không có node**), 3 cover đúng
  **1920×1080 / 800×1200 / 800×800** (verify từ byte 16–23 của file PNG), chữ trên
  cover cách mép ≥ 5%. Cover vẽ bằng **đúng renderer của game** nên luôn khớp
  hình thật.
- ✅ **i18n** — bảng `I18N` inline; **tutorial dịch đủ 23 ngôn ngữ**, HUD
  (SCORE/BEST/LEVEL) giữ tiếng Anh qua fallback `en` — deviation chủ đích so với
  game-common §5, theo tiền lệ animal-connect / fruit-connect sau góp ý QA
  "localize thừa".
- ℹ️ `button-common.md` / `background-common.md` được các spec khác tham chiếu
  nhưng **không tồn tại trong repo** → `btn3d` lấy từ runic-blaze / fruit-connect.
- ℹ️ Mở bằng browser thường: tự giả lập popup của app (next/retry sau 1.4s) để
  playtest — trong WebView thật không chạy.

## Kiểm chứng

Bot chạy trong **headless Edge**, có **bản BFS + bản tính `par` viết độc lập** để
đối chiếu chéo với code game. **15.441 assertion, 0 fail, 0 lỗi JS.**

| Hạng mục | Kết quả |
|---|---|
| Sinh màn (cả 60) | Không ô rỗng; số đầu nối đúng `2(N−k)`; lời giải thắp đủ 100% ô và **0 đầu thừa**; không màn nào bắt đầu ở trạng thái đã giải |
| `par` | Khớp 60/60 với bản tính độc lập (có xử lý đối xứng: thẳng chu kỳ 2, chữ thập chu kỳ 1) |
| Ngân sách lượt | 60/60 màn có `limit ≥ par + 3` (không màn nào bất khả thi); khớp `movesLimitFor`; tỉ lệ đo được **3.00× (L1–9) → 2.43× (L10) → 2.11× (L35) → 1.80× (L60)** |
| Bot chơi 60 màn | Giải được **toàn bộ**, dùng **đúng `par` lượt**, 3★, không màn nào chạm trần lượt |
| Điểm | Mỗi màn cộng **đúng** `50×lvl + 25×lượt thừa`; `best` không bao giờ tụt sau `score` |
| Fuzz | 4.800 nước ngẫu nhiên: **không lần nào** "sáng hết mà còn đầu thừa"; BFS của game khớp BFS độc lập ở **mọi** nước; **không nước nào vượt quá trần lượt** |
| Bridge | Màn 5: đúng **1** `game_result` (win + `showModal:true`, `level:5`) + `ads`; màn 6 không `ads`; màn 60 chỉ `victory`; hết lượt ở màn 21 → đúng **1** `game_result` (**lose** + `showModal:true`, `level:21`), không kèm `victory`; mọi message đủ 5 trường |
| Thua → Retry | Retry sau khi thua → về **màn 1**, `score` về 0, `best` giữ nguyên, run chơi lại được. Retry giữa màn (chưa thua) → dựng lại **đúng màn đó**, `moves` về 0 |
| Ô BEST | Vào run mới hiện kỷ lục cũ; **giữa run không nhúc nhích** dù `score` tăng; hết run mới bắt kịp. Ô SCORE thì chạy live |
| Hình học dây | Mọi nhánh **đáp đúng lên mép ô** (sai < 0.9px), mọi cạnh không nhánh **cách mép > 2px**, **không nét nào vẽ ra ngoài ô** |
| Cỡ header | Nhãn 12.48px, số 19.5px, pill ~55px, nút **50×50px**, tỉ lệ nút/pill 0.92 — đúng `index(exampleforheader).html` |
| HUD | Không còn nút Restart; pill trên là SCORE/BEST; hàng dưới có LEVEL + lượt còn lại; thanh lượt bắt đầu đầy |
| Determinism | Màn 24 sinh 2 lần ra y hệt; màn 24 ≠ màn 25 |
| Resume | Reload giữa chừng: đúng màn, **đúng từng góc xoay**, đúng `moves`, đúng số ô sáng, `transform` trong DOM khớp state; snapshot cũ **không** dán sang màn khác |
| Input native | `?statusBarHeight=44` → HUD chừa đủ, board nằm dưới HUD, không tràn ngang; `currentLevel=7` → đúng màn 7; `currentLevel=999` → kẹp về màn 60; `language=vi` → tutorial tiếng Việt; `language=ru` → `--ui-font` về `system-ui`; `language=klingon` → fallback `en` |

### Bug bắt được trong lúc làm & test

1. **Kết quả bắn cho màn đã rời đi.** `game_result` hẹn giờ sau khi kết thúc màn
   (để chạy hiệu ứng). Nếu trong lúc đó người chơi rời màn thì timer vẫn nổ và
   `sendMessage` đóng dấu `level` **hiện tại** → báo sai màn. → Thêm `levelEpoch`:
   `loadLevel` tăng epoch, timer kiểm lại epoch trước khi bắn.
2. **Người chơi mới bị thả vào màn 7 thì không có tutorial** (trước đó tutorial chỉ
   chạy khi `currentLevelIdx === 0`). → "Màn đầu tiên của phiên chơi".
3. **Chữ "COMPLETE" trên hiệu ứng thắng vô hình** — bạc hà đè lên đúng đám dây bạc
   hà vừa thắp. → Bỏ hẳn chữ, chỉ để 3 ngôi sao vàng viền tối.
4. **Lõi chìm nghỉm trên bàn lớn** → thoi to hơn, ruột nhỏ lại.
5. **Ô BEST chạy live cùng SCORE** — phát hiện khi soi ảnh chụp: hai ô luôn bằng
   nhau, vô nghĩa. Vi phạm game-common §1.2. → Tách `best` (lưu ngay) khỏi
   `bestShown` (chỉ refresh lúc hết run / vào run mới).
6. **Khôi phục vào bàn chết.** Snapshot lưu sau *mỗi* nước, kể cả nước cuối làm hết
   lượt. Mở lại sẽ dựng lại đúng bàn đó với 0 lượt còn lại và không nước đi nào hợp
   lệ. → `restoreRotations` bỏ snapshot có `moves ≥ movesLimit`.

> Ghi chú về phép đo: assertion hình học dây lúc đầu **báo sai 30 lỗi**
> "dây ngắn 6.67px". Thủ phạm là phép đo chứ không phải code —
> `getBoundingClientRect()` trên `<path>` SVG trả **hộp hình học, không gồm nét
> vẽ**, mà 6.67px đúng bằng nửa nét (7.5% cạnh ô). Test đã được sửa để cộng bù
> nửa nét; code render vốn đã đúng.

## 📋 Backlog

- [ ] Chế độ endless sau màn 60 (sinh vô hạn, vẫn dùng seed)
- [ ] Gợi ý: nháy một ô đang xoay sai sau ~15s không thao tác
- [ ] Haptic nhẹ mỗi lần một cụm ô sáng lên (cần native)
