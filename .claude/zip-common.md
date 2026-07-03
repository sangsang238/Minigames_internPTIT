# Đóng gói `.zip` để publish — quy ước chung cho các game HTML

Tài liệu mô tả **cách đóng gói game thành file `.zip`** khi làm game xong, để upload
lên trang publish (tab **Upload .zip**). Zip gồm `index.html` ở root, `game.json`
mô tả thuộc tính game, và **3 ảnh cover**. Hệ thống sẽ đọc `game.json`, validate và
pre-fill form để review trước khi publish.

Chỉ đóng gói khi game đã hoàn thiện đủ **3 quy ước chung**:

- Event log & native bridge: [game-common.md](game-common.md)
- Nền trang trí chung: [background-common.md](background-common.md)
- Nút 3D `btn3d`: [button-common.md](button-common.md)

---

## 1. Cấu trúc `.zip`

```
<game>.zip
├─ index.html                      → entry (BẮT BUỘC nằm ngay tại root)
├─ game.json                       → metadata, hệ thống đọc để pre-fill form
├─ cover-landscape-1920x1080.png   → coverImages.landscape16x9
├─ cover-portrait-800x1200.png     → coverImages.portrait2x3
├─ cover-square-800x800.png        → coverImages.square1x1
└─ assets/                         (optional — chỉ khi có tách file)
   ├─ game.js
   └─ style.css
```

**Quy tắc:**

- `index.html` phải nằm **ngay tại root** của zip, không nằm trong folder con. Lỗi
  phổ biến nhất là nén cả thư mục cha → mọi file bị lồng thành
  `<game>/index.html` → hệ thống không tìm thấy entry (xem §4).
- Game theo quy ước chung là **single-file `index.html`** (CSS/JS inline) → thường
  **không cần** `assets/`. Nếu có tách file, mọi tham chiếu trong HTML phải là
  **đường dẫn tương đối** (`assets/game.js`), không dùng đường dẫn tuyệt đối hay
  file local ngoài zip.
- **Không** đóng gói file rác / file dev: `.DS_Store`, `__MACOSX/`, `.git/`,
  `node_modules/`, file `.md`, file design (`.psd`, `.fig`)…
- Tên zip đặt theo tên game, lowercase: `snake.zip`, `ballsort.zip`…

---

## 2. `game.json`

JSON **hợp lệ tuyệt đối** (không comment, không trailing comma), encoding UTF-8,
đặt ngay root zip. Toàn bộ text hiển thị (`title`, `description`, `controls`,
`tags`) viết **tiếng Anh** — khớp rule ngôn ngữ trong
[game-common.md](game-common.md).

### 2.1. Field bắt buộc

| Field         | Kiểu       | Quy tắc |
|---------------|------------|---------|
| `title`       | `string`   | Tên hiển thị, ngắn gọn. Vd `"Snake"`. |
| `packageName` | `string`   | **Reverse-DNS**, lowercase, không dấu/space. Vd `"com.falcon.snake"`. Là định danh game — đặt một lần, không đổi khi update. |
| `engine`      | `string`   | `"HTML5"` với game HTML. |
| `version`     | `string`   | Semver, bắt đầu `"1.0.0"`; tăng mỗi lần up bản mới. |
| `category`    | `string`   | UPPERCASE, **một** giá trị, chọn đúng category platform hỗ trợ. Vd `"PUZZLE"`, `"ARCADE"`. |
| `tags`        | `string[]` | **Tối đa 5**, lowercase, là từ khoá tìm kiếm. Vd `["snake", "classic", "casual"]`. |
| `description` | `string`   | 1–2 câu mô tả gameplay. |
| `controls`    | `string`   | 1 câu mô tả cách điều khiển. |
| `entry`       | `string`   | Tên file HTML của game, thường `"index.html"`. |
| `coverImages` | `object`   | 3 key `landscape16x9` / `portrait2x3` / `square1x1` → **tên file ảnh trong zip**. |

### 2.2. Field optional

| Field         | Kiểu     | Ý nghĩa |
|---------------|----------|---------|
| `aiContext`   | `string` | Ngữ cảnh/prompt AI đã dùng build game. Để `""` nếu không có. |
| `aiBuildLink` | `string` | Link tool/chat AI đã build. |
| `demoUrl`     | `string` | Link chơi thử online. |
| `storeLinks`  | `object` | `googlePlay` / `iosAppStore` / `steam`, mỗi store là `{ "url": "", "downloads": 0 }`. |

> Optional **vẫn nên khai báo đủ** với giá trị rỗng như mẫu — form pre-fill không bị
> thiếu cột, sau này chỉ điền thêm.

### 2.3. Mẫu đầy đủ

```json
{
  "title": "Snake",
  "packageName": "com.falcon.snake",
  "engine": "HTML5",
  "version": "1.0.0",
  "category": "ARCADE",
  "tags": ["snake", "classic", "arcade", "casual"],
  "description": "Eat food to grow longer and rack up points. Don't crash into the walls or your own tail.",
  "controls": "Swipe or use arrow keys to steer the snake",
  "aiContext": "",
  "aiBuildLink": "",
  "demoUrl": "",
  "storeLinks": {
    "googlePlay": { "url": "", "downloads": 0 },
    "iosAppStore": { "url": "", "downloads": 0 },
    "steam": { "url": "", "downloads": 0 }
  },
  "entry": "index.html",
  "coverImages": {
    "landscape16x9": "cover-landscape-1920x1080.png",
    "portrait2x3": "cover-portrait-800x1200.png",
    "square1x1": "cover-square-800x800.png"
  }
}
```

**Quy tắc khớp file (hệ thống validate):**

- `entry` khớp **đúng** tên file HTML có trong zip.
- 3 giá trị trong `coverImages` khớp đúng tên 3 file ảnh trong zip —
  **case-sensitive**, sai một ký tự là validate fail.

---

## 3. Ảnh cover (3 ảnh bắt buộc)

| Key             | Tỉ lệ | Kích thước (px) | Tên file quy ước |
|-----------------|-------|-----------------|------------------|
| `landscape16x9` | 16:9  | **1920×1080**   | `cover-landscape-1920x1080.png` |
| `portrait2x3`   | 2:3   | **800×1200**    | `cover-portrait-800x1200.png` |
| `square1x1`     | 1:1   | **800×800**     | `cover-square-800x800.png` |

- Định dạng **PNG** (ưu tiên) hoặc JPG, đúng kích thước pixel như bảng.
- Nội dung: key art / khung hình đẹp nhất của game + tên game (tiếng Anh). Chữ và
  logo cách mép **≥ 5%** (safe area) để không bị crop khi hiển thị thumbnail.
- Đặt tên file đúng quy ước trên cho đồng bộ giữa các game; nếu đổi tên thì phải
  sửa `coverImages` trong `game.json` khớp theo.
- Nén ảnh trước khi đóng gói (tinypng…) để zip gọn nhẹ.

---

## 4. Lệnh nén zip

**Nguyên tắc: đứng TRONG thư mục game và nén nội dung (`.`)** — để mọi file nằm
ngay root zip, không bị lồng folder.

macOS / Linux:

```bash
cd games/<game>

zip -r -X ../<game>.zip . \
  -x '.DS_Store' -x '__MACOSX/*' -x '*.git*' \
  -x 'node_modules/*' -x '*.md'
```

Windows (PowerShell) — `\*` để nén nội dung bên trong folder:

```powershell
Compress-Archive -Path games\<game>\* -DestinationPath <game>.zip -Force
```

**Lỗi phổ biến — nén cả folder:**

```bash
# SAI: index.html bị lồng trong <game>/ → hệ thống không thấy entry
zip -r <game>.zip games/<game>
```

**Kiểm tra sau khi nén:**

```bash
unzip -l <game>.zip
# Phải thấy index.html, game.json, 3 file cover-*.png ngay ở root
# (không có prefix folder), không có .DS_Store / __MACOSX.
```

---

## 5. Script kiểm tra nhanh (chạy trước khi nén)

Chạy trong thư mục game — check `game.json` đủ field, đúng format, các file tham
chiếu tồn tại. Lỗi ở đâu báo ở đó, pass thì in `OK`.

```bash
node -e '
const fs = require("fs");
const g = JSON.parse(fs.readFileSync("game.json", "utf8"));

const req = ["title","packageName","engine","version","category",
             "tags","description","controls","entry","coverImages"];
const miss = req.filter(k => !(k in g));
if (miss.length) throw "game.json thiếu field: " + miss.join(", ");

if (!Array.isArray(g.tags) || g.tags.length > 5)
  throw "tags phải là mảng tối đa 5 phần tử";
if (!/^[a-z][a-z0-9]*(\.[a-z][a-z0-9]*)+$/.test(g.packageName))
  throw "packageName sai dạng reverse-DNS: " + g.packageName;

const need = [g.entry, g.coverImages.landscape16x9,
  g.coverImages.portrait2x3, g.coverImages.square1x1];
for (const f of need)
  if (!fs.existsSync(f)) throw "Thiếu file trong folder: " + f;

console.log("OK — game.json hợp lệ, sẵn sàng nén zip");
'
```

---

## 6. Checklist trước khi publish

**Game (soát theo 3 quy ước chung):**

- [ ] Bắn đủ event qua `sendMessage`: `game_result` (kèm `showModal` đúng loại
  game), `retry_level`, `next_level`, `victory`, `save_data` — theo
  [game-common.md](game-common.md).
- [ ] Đọc `statusBarHeight` / `currentLevel` / `data` từ native; **chừa trống góc
  trên trái** (~48dp) cho nút Home của app.
- [ ] Nền dùng `drawGameBackground` ([background-common.md](background-common.md)),
  nút dùng `btn3d` ([button-common.md](button-common.md)), font **Changa One**.
- [ ] Toàn bộ text trong game là **tiếng Anh**.

**Gói zip:**

- [ ] `game.json` hợp lệ, đủ field bắt buộc, text tiếng Anh (chạy script §5 pass).
- [ ] `entry` và `coverImages.*` khớp đúng tên file thật (case-sensitive).
- [ ] 3 ảnh cover đúng kích thước **1920×1080 / 800×1200 / 800×800**.
- [ ] `index.html` nằm ở root zip; không có file rác (soát bằng `unzip -l`).
- [ ] Giải nén thử ra folder trống → mở `index.html` chạy bình thường (asset tương
  đối không vỡ).

---

## 7. Lưu ý

- **Mỗi zip = 1 game.** Up bản mới: tăng `version` trong `game.json` rồi nén lại —
  **không đổi `packageName`** (đó là định danh của game trên hệ thống).
- Hệ thống chỉ **pre-fill** form từ `game.json` — sau khi upload vẫn phải review lại
  form (title, tags, mô tả, cover) trước khi bấm publish.
- `demoUrl` / `storeLinks` chưa có thì để rỗng như mẫu, bổ sung sau khi game lên
  store.
- Giữ zip gọn nhẹ: game single-file `index.html` + 3 cover thường chỉ **1–2 MB**.

---

## Tóm tắt luồng

```
hoàn thiện game (game-common + background-common + button-common)
        ▼
chuẩn bị game.json + 3 ảnh cover (đúng tên, đúng kích thước)
        ▼
chạy script check (§5) ──fail──► sửa rồi chạy lại
        ▼ pass
nén zip từ TRONG thư mục game (§4) → unzip -l soát root
        ▼
Upload .zip → hệ thống đọc game.json, pre-fill form → review → Publish
```
