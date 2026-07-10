# Skydom — HaND

> Match-3 **endless** trên trời candyland | HTML5 single-file | Không level, ăn điểm cao

| | |
|---|---|
| **Package** | `com.falcon.skydom` |
| **Engine / Version** | HTML5 · 1.0.0 |
| **Category** | PUZZLE |
| **Entry** | `skydom/index.html` |

## Cách chơi

- Bàn **8×8**, **5 màu** viên — mỗi màu là một **hình riêng** (tim đỏ · giọt nước
  xanh dương · tam giác xanh lá · tròn vàng · thoi tím) để người mù màu vẫn phân
  biệt được.
- **Kéo** (swipe) hoặc **tap 2 viên kề nhau** để đổi chỗ. Đổi hợp lệ khi tạo được
  hàng/cột **≥ 3 viên cùng màu** → chúng nổ, viên trên rơi xuống lấp chỗ, tạo
  **cascade** (dây chuyền) ăn thêm điểm.
- **Endless — không giới hạn lượt, không level.** Chơi tới khi **bom đếm ngược nổ**
  (xem dưới). Mục tiêu: phá **kỷ lục điểm** (Best).
- Bàn tự **trộn lại** (shuffle) khi hết nước đi hợp lệ.
- **Không bị khoá khi đang nổ:** lúc một vùng đang nổ/rơi, người chơi vẫn **kéo được ở
  những cột đã lắng** (grab theo từng cột) — nước đi được **chèn** vào chuỗi đang chạy,
  không cần chờ nổ xong.

## Viên đặc biệt (special)

Ghép nhiều hơn 3 → tạo viên mạnh. **Special (sọc/bom) chỉ NỔ khi được GHÉP cùng màu
lần nữa** (hoặc đổi với special khác); riêng **cầu vồng** kích hoạt khi đổi với viên
bất kỳ.

| Tạo bằng | Viên | Hiệu ứng khi kích hoạt |
|---|---|---|
| **4 viên** thẳng hàng | 🚀 **Sọc** (ngang/dọc) | Nổ **cả một hàng hoặc cột** |
| Ghép hình **L / T** (giao nhau) | 💥 **Bom gói** | Nổ vùng **3×3** quanh nó (nổ 2 nhịp) |
| **5 viên** thẳng hàng | 🌈 **Cầu vồng** | Phóng **tia xẹt** rồi xoá **mọi viên cùng một màu**. Line dài **chia theo mỗi 5** → nhiều cầu vồng |

**Combo — đổi 2 special cạnh nhau:**

| Combo | Kết quả |
|---|---|
| Sọc + Sọc | Nổ hình **chữ thập** (1 hàng + 1 cột) |
| Sọc + Bom | Nổ **3 hàng + 3 cột** (dấu cộng dày) |
| Bom + Bom | Nổ vùng **5×5** |
| 🌈 + thường | Xoá hết viên **cùng màu** viên được đổi |
| 🌈 + Sọc/Bom | Mọi viên cùng màu hoá sọc/bom rồi **nổ lần lượt** |
| 🌈 + 🌈 | **Dọn sạch bàn** |

## Bom đếm ngược (thử thách thua)

- Định kỳ sinh **1 quả bom** với số **đếm ngược ngẫu nhiên 5–15**; mỗi lượt đi số giảm 1
  (≤ 3 thì **nhấp nháy** cảnh báo).
- Tần suất sinh **tăng dần theo độ khó**: ban đầu **mỗi 6 lượt** một quả, nhanh dần tới
  **mỗi 2 lượt**; tối đa **5 quả** trên bàn cùng lúc.
- Bom **không đổi chỗ / không match** được. **Gỡ bom** = cho một vụ nổ **trúng ô kề**
  (trên/dưới/trái/phải) hoặc chính ô bom → **+50 điểm**.
- Quả bom nào **đếm về 0** → **NỔ LỚN & thua** (game over).

## Điểm

| Sự kiện | Điểm |
|---|---|
| Mỗi viên bị xoá | **10 × hệ số chuỗi** (cascade 1, 2, 3…) |
| Gỡ được 1 bom | **+50** |

- Chuỗi cascade càng dài → hệ số nhân càng cao, kèm chữ khen tăng dần
  (Nice → Great → … → Legendary).
- **Best** lưu ngay khi đổi (persist native + localStorage), không mất khi thoát
  đột ngột; HUD Best refresh khi hết ván / vào ván mới.

## Hướng dẫn — Onboarding (chỉ hiện lần đầu)

Mở game lần đầu hiện **chuỗi 5 popup mẫu** nối tiếp, mỗi popup có **demo minh hoạ dựng
bằng chính viên của game** (`shapeSvg`):

1. **Đổi & Ghép** — kéo viên xếp 3 cùng màu → nổ ăn điểm
2. **Nổ hàng** (kẹo sọc) — ghép **4 thẳng hàng** tạo sọc → **ghép 3 cùng màu có sọc** → nổ cả hàng
3. **Bom gói** — ghép **hình T** tạo bom → **ghép 3 cùng màu có bom** → nổ 3×3
4. **Cầu vồng** — ghép **5 thẳng hàng** tạo cầu vồng → đổi với viên đỏ → **tia xẹt** xoá mọi viên đỏ
5. **Bom đếm ngược** — kéo ghép cạnh bom để **gỡ (+50)** trước khi nó về 0

Mỗi bước special mô phỏng **đúng luồng game 2 bước** (ghép TẠO → ghép KÍCH HOẠT nổ),
kèm hiệu ứng match (chớp + mảnh vỡ). Hàng nút cân đối **‹ lùi · Bỏ qua · › tiến**
(nút ‹ mờ ở bước đầu, nút › amber); popup **cuối** nút › đổi thành **Chơi** mới vào
game. Lưu cờ `tutorialSeen` (qua `save_data` + localStorage) → chỉ hiện **một lần**.

## Hiệu ứng (juice)

- **Match thường nổ & rơi nhanh gọn**, còn **mảnh vỡ nấn ná** lâu hơn một nhịp để đọng
  lại cảm giác nổ.
- **Mảnh vỡ** = hình **trăng khuyết bóng 3D** (gradient + đầu bo tròn), chỉ **hiện đúng
  lúc viên nổ** (không lấp ló trước), **bung tại chỗ** rồi mờ tan.
- **Item đặc biệt nổ trễ ~0.2s** so với viên thường ở cùng đợt — "charge" một nhịp rồi
  mới bung tia/vòng, cho thấy rõ special vừa kích hoạt.
- **Mỗi viên ăn được = 1 ngôi sao**, xuất hiện **sau khi vụ nổ hoàn tất**, tại **tâm
  viên đặc biệt** (cầu vồng → tâm cầu vồng; match thường → tâm cụm nổ). Sao **bay vèo
  thẳng** về thẻ SCORE (không rề rà), cả đàn **so le nhưng gọn trong ~1s** → lóe sáng +
  thẻ nảy + đếm điểm + tiếng "ting".
- **Kẹo chỉ rớt xuống** lấp chỗ trống khi **ngôi sao cuối đã bay ~2/3 đường** về bảng điểm.
- Cầu vồng có **tia xẹt** (sét nhiều màu) nối lần lượt tới từng viên; combo có pháo
  hoa khi phá kỷ lục.
- **Tự chữa trạng thái:** khi hiệu ứng dồn dày, viên nào lỡ bị kẹt style (phình to /
  trong suốt) sẽ được **quét & khôi phục** cuối mỗi đợt — không đụng viên đang nổ/rơi thật.
- **Âm thanh Web Audio tổng hợp** (không file rời): nhạc nền vui + tiếng nổ theo
  chuỗi + "ting" khi ghi điểm. Có nút bật/tắt tiếng.

## Tuân thủ quy ước (.claude)

- **game-common.md** — `sendMessage` đủ **5 trường**, `level: null` (game không level);
  bắn `game_result` (`result:null`, `showModal:false`) · `retry_level` (Play Again) ·
  `quit` (nút Back). `save_data` mang data nhẹ `{best, muted, tutorialSeen}` — persist
  **ngay khi đổi** + mirror `localStorage`. Đọc `statusBarHeight`/`data`/`language`
  đúng thứ tự ưu tiên; callback native (`onAppPause`/`onAppResume`/`onRetryLevel`) định
  nghĩa sẵn; nút Back dùng **SVG chuẩn**; font **Google Sans Flex** (+ fallback
  `system-ui` cho ngôn ngữ ngoài Latinh); reset CSS `*` + tắt `tap-highlight`/callout.
- **popup-common.md** — game điểm cao **tự vẽ popup**: một tiêu đề `New Best!` /
  `Game Over` (đổi theo `score > best`), nhãn xanh dương · SCORE trắng · BEST vàng ·
  nút Play Again amber, `fitScores()` co điểm dài, ẩn/hiện bằng class `.hidden`
  (fade+scale), delay ~550ms sau khi thua.
- **zip-common.md** — single-file `index.html` ở root, `game.json` đủ field bắt buộc
  (text tiếng Anh), **3 cover** đúng tên & kích thước (1920×1080 / 800×1200 / 800×800),
  dựng bằng **chính gem + khay bàn của game** (`shapeSvg` + CSS `.board-wrap`) nên khớp
  100% với game. Zip **không** kèm `.md` / file dev (`test.html`).
- **i18n** — bảng dịch inline trong `index.html`; đủ `en` (mặc định) + `vi`, các ngôn
  ngữ khác fallback `en`.

## File dev (không đóng gói)

- `test.html` — **màn test các trường hợp nổ**: mỗi nút tự dựng bàn (nền caro không tự
  khớp) rồi kích một kịch bản — match thường, từng viên đặc biệt (sọc/gói/cầu vồng),
  các combo (sọc+sọc, gói+gói, 🌈+sọc/gói, 🌈+🌈…), gỡ bom & bom nổ thua. Điều khiển
  engine qua **bridge `?debug=1`** (postMessage), chạy được cả `file://`.

## Backlog

- [ ] Dịch đủ chuỗi cho các ngôn ngữ ngoài `en`/`vi`
- [ ] Skin/theme viên mở bằng điểm
- [ ] Daily quest + streak (localStorage)
