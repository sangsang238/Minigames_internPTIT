# Event log & native bridge — quy ước chung cho các game HTML

Tài liệu mô tả **rule log event** trong mỗi file `index.html` của game, cùng 4 input
mà native (React Native WebView) truyền vào: `statusBarHeight`, `currentLevel`
(game có level), `data` (app truyền chuỗi dữ liệu vào), và `language` (mã ngôn ngữ
hiện tại của app để game tự dịch UI).

Mọi game đều chạy trong React Native WebView. Giao tiếp 2 chiều:

- **Game → native**: `window.ReactNativeWebView.postMessage(JSON.stringify(msg))`
- **native → game**: inject biến global (`window.statusBarHeight`,
  `window.currentLevel`, `window.data`, `window.language`) **hoặc** truyền qua URL
  query (`?statusBarHeight=&currentLevel=&data=&language=`).

## Giao diện chung (UI)

Để các game đồng bộ về giao diện, **bắt buộc** dùng các quy ước UI dùng chung:

- **Font chữ**: dùng font **Google Sans Flex** (display font, variable weight) cho
  tiêu đề, điểm số, nút và HUD — khớp với `btn3d` ở
  [button-common.md](button-common.md). Nạp từ Google Fonts trong `<head>`:

  ```html
  <link rel="preconnect" href="https://fonts.googleapis.com" />
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
  <link href="https://fonts.googleapis.com/css2?family=Google+Sans+Flex:opsz,wght@6..144,1..1000&display=swap" rel="stylesheet" />
  ```

  Google Sans Flex là **variable font** (weight 1..1000) → đặt độ đậm rõ ràng cho UI
  display (vd `700`), nếu không text sẽ mảnh. CSS dùng **biến `--ui-font`** thay vì
  hard-code tên font, để JS đổi sang `system-ui` cho ngôn ngữ font không phủ:

  ```css
  /* mặc định: Google Sans Flex (phủ toàn bộ Latin, gồm tiếng Việt) */
  :root { --ui-font: 'Google Sans Flex', system-ui, sans-serif; }
  body  { font-family: var(--ui-font); font-weight: 700; }
  ```

  Có thể dùng `system-ui` cho phần body text dài/mô tả (dễ đọc hơn), nhưng tiêu đề và
  UI chính giữ `var(--ui-font)`.

  > ⚠️ **Google Sans Flex chỉ phủ chữ Latinh** (đủ cho `en` + tiếng Việt có dấu
  > `ă ơ ư đ ạ ế`, `pl`, `tr`…). Nó **không** có glyph Cyrillic (`ru`), CJK
  > (`zh`/`ja`/`ko`), Ả Rập (`ar`), Thái (`th`), Devanagari (`hi`). Nếu để Google Sans
  > Flex đứng đầu stack cho các ngôn ngữ đó thì **1 dòng text bị trộn 2 font** (chữ
  > Latin ra Google Sans Flex, ký tự còn lại rớt về `system-ui`). → Với ngôn ngữ ngoài
  > Latinh, đổi `--ui-font` thành **font hệ thống** để cả dòng dùng một font.

  Khai một danh sách ngôn ngữ Latinh; lúc boot (sau khi có `LANG`, xem §5), nếu `LANG`
  **không** thuộc danh sách thì set `--ui-font` = `system-ui`:

  ```js
  // Ngôn ngữ Google Sans Flex render ĐỦ glyph (mọi chữ Latinh, gồm tiếng Việt).
  const LATIN_LANGS = new Set([
    'en','vi','es','es-MX','es-AR','pt-BR','pt','fr','de','it','nl','pl','tr','id','fil',
  ]);

  function applyUiFont(lang) {
    if (!LATIN_LANGS.has(lang)) {        // ru / zh / ja / ko / ar / th / hi → font máy
      document.documentElement.style.setProperty('--ui-font', 'system-ui, sans-serif');
    }
  }

  applyUiFont(LANG);   // gọi lúc boot, trước khi render UI
  ```

  > **Canvas**: nếu game vẽ chữ/số lên `<canvas>`, đặt font kèm weight trong chuỗi
  > `ctx.font` (canvas không tự đọc `font-weight` từ CSS), vd:
  > `ctx.font = "700 " + size + "px 'Google Sans Flex', system-ui, sans-serif"`. Với
  > nội dung chỉ là chữ số (Latin) thì giữ `'Google Sans Flex'` ở mọi ngôn ngữ là ổn.
- **Ngôn ngữ**: game **phải localize toàn bộ text UI** (tiêu đề, nút, điểm số, nhãn
  HUD, thông báo thắng/thua, hướng dẫn…) theo ngôn ngữ hiện tại của app — native
  truyền mã ngôn ngữ vào qua `window.language` / `?language=`. Tiếng **Anh (`en`)**
  là ngôn ngữ mặc định/fallback. Cách đọc `language` và bảng mã ngôn ngữ hỗ trợ xem
  **§5**. Không hard-code 1 ngôn ngữ cố định trong UI.
- **Reset & tắt highlight khi chạm (mobile)**: **bắt buộc** dùng block reset chung cho
  `*, *::before, *::after` (gồm `box-sizing`, reset `margin`/`padding`) **kèm** tắt ô
  xanh dương WebKit hiện ra khi chạm và menu callout giữ-để-chọn trên điện thoại —
  chúng làm UI nhấp nháy/lệch khi người chơi tap nút. Đây là đoạn các game đang thêm:

  ```css
  *,
  *::before,
  *::after {
    box-sizing: border-box;
    margin: 0;
    padding: 0;
    -webkit-tap-highlight-color: transparent; /* tắt ô xanh dương khi chạm */
    -webkit-touch-callout: none;              /* tắt menu giữ-để-chọn */
  }
  ```

  > Đặt trong CSS chung của `index.html`, ngay đầu stylesheet (trước các rule khác).
  > Áp cho `*` cùng `::before`/`::after` để mọi phần tử (kể cả pseudo-element) đều
  > hết highlight. Kết hợp thêm `user-select: none` cho phần tử tap nhiều (nút, ô
  > game) để tránh bôi đen text khi chạm nhanh:
  >
  > ```css
  > .btn3d, .hud-btn, .cell {
  >   -webkit-user-select: none;
  >   user-select: none;
  > }
  > ```

---

## 1. Game → native: log event

Tất cả event đi qua **một hàm `sendMessage(type, opts)` duy nhất**. Hàm này đóng
gói thành object JSON gồm **5 trường** (`type`, `level`, `result`, `data`,
`showModal`) rồi gửi lên native. Mỗi message **luôn có đủ 5 trường** — trường không
dùng để `null` (hoặc `false` với `showModal`) để native parse đồng nhất. Khi mở
bằng trình duyệt thường (không có `ReactNativeWebView`) thì `console.log` để dev
kiểm tra — bọc `try/catch` để không bao giờ throw.

Điểm khác duy nhất giữa game **có level** và **không level** là trường `level`:

```js
// ── Game CÓ level ──────────────────────────────────────────────
function sendMessage(type, opts) {
  opts = opts || {};
  post({
    type,                              // string — tên event (xem bảng dưới)
    level: currentLevelIdx + 1,        // number — level 1-indexed
    result: opts.result ?? null,       // string|null — kết quả ('win'/'lose'…)
    data: opts.data ?? null,           // string|null — dữ liệu thô (save_data)
    showModal: opts.showModal === true,// boolean — app có tự hiện popup không
  });
}

// ── Game KHÔNG level (điểm cao / endless) ──────────────────────
function sendMessage(type, opts) {
  opts = opts || {};
  post({
    type,
    level: null,                       // game không có level → luôn null
    result: opts.result ?? null,
    data: opts.data ?? null,
    showModal: opts.showModal === true,
  });
}

// Helper chung — gửi lên native, fallback console.log khi mở bằng browser thường.
function post(msg) {
  try {
    if (window.ReactNativeWebView && window.ReactNativeWebView.postMessage) {
      window.ReactNativeWebView.postMessage(JSON.stringify(msg));
    } else {
      console.log('[<game> → native]', msg);
    }
  } catch (e) { /* ignore */ }
}
```

> Mỗi game chỉ định nghĩa **một** trong hai bản trên (tuỳ có level hay không). Phần
> còn lại của tài liệu dùng chung tên `sendMessage`. Tham số thứ 2 là **object
> tuỳ chọn** `{ result, data, showModal }` — event nào cần trường nào thì truyền
> trường đó, còn lại tự về `null`/`false`.

### Cấu trúc message

Mỗi message **luôn có đủ 5 trường**; trường không áp dụng cho event đó để `null`
(hoặc `false`).

| Field    | Kiểu          | Ý nghĩa |
|----------|---------------|---------|
| `type`   | `string`      | Tên event. |
| `level`  | `number\|null`| Level hiện tại, **1-indexed**. Game không có level → `null`. |
| `result` | `string\|null`| Kết quả của event (vd `'win'`/`'lose'`). `null` nếu không áp dụng. |
| `data`   | `string\|null`| Dữ liệu thô đính kèm. Hiện chỉ `save_data` dùng (chuỗi data **nhẹ** cần lưu, xem §1.2). Các event khác → `null`. |
| `showModal` | `boolean`  | Chỉ dùng cho `game_result`: app có **tự hiện popup thắng/thua** hay không (xem §1.1). Mặc định `false`. |

### Bảng event types

Cột "trường dùng" ghi trường nào mang giá trị (ngoài `type`/`level` luôn có); các
trường còn lại = `null`/`false`.

| `type`          | Trường dùng                    | Khi nào gửi |
|-----------------|--------------------------------|-------------|
| `game_result`   | `result` (`'win'`/`'lose'`/`null`) + `showModal` | Khi ván kết thúc. Game **có level**: `'win'`/`'lose'`. Game **không level**: `result = null`. Kèm `showModal` — xem §1.1. |
| `retry_level`   | — (chỉ `type`/`level`)         | Người chơi bấm chơi lại. Chỉ bắn khi **game tự xử lý** việc chơi lại (xem §1.3). |
| `next_level`    | — (chỉ `type`/`level`)         | Người chơi qua level kế tiếp. Chỉ bắn khi **game tự xử lý** việc chuyển level (xem §1.3). |
| `victory`       | — (chỉ `type`/`level`)         | Hoàn thành level cuối cùng (clear toàn bộ game). Bắn **thay** `game_result` (không bắn cả hai); app tự hiện popup chúc mừng (xem §1.5). |
| `save_data`     | `data` (chuỗi data **nhẹ**)    | Game yêu cầu native lưu data nhẹ (best/streak/tiến độ…). Trạng thái ván nặng lưu `localStorage`, không qua đây — xem §1.2. |
| `quit`          | — (chỉ `type`/`level`)         | Người chơi bấm **nút Back** trong game để thoát (xem §1.4). |
| `ads`           | — (chỉ `type`/`level`)         | Game yêu cầu app hiển thị quảng cáo **fullscreen** (xem §1.7). |

**Quy tắc:**

- `type` và `result` luôn là chuỗi **snake_case** cố định — native dựa vào đó để
  phân nhánh, không tự ý đổi tên.
- Mỗi hành động chỉ bắn **một** event. Vd thua màn: bắn `game_result/'lose'`; nếu
  game tự xử lý nút chơi lại thì khi bấm mới bắn `retry_level`.
- Thắng **level cuối** bắn `victory` (KHÔNG bắn `game_result` — xem §1.5); thắng
  level thường bắn `game_result/'win'`.
- `victory` chỉ bắn khi `currentLevelIdx === LEVELS.length - 1` (level cuối).
- `currentLevelIdx` = chỉ số level **0-indexed** hiện tại; `LEVELS` = mảng định nghĩa
  các level; `loadLevel(idx)` = hàm nạp & reset một level. Game không có level thì
  `currentLevelIdx` không dùng (level gửi lên là `null`).

### 1.1. Popup thắng/thua: `game_result` + `showModal`

Sau khi game bắn `game_result`, **app tự xử lý hiển thị popup thắng/thua**. **Quy
tắc chung: game KHÔNG tự vẽ modal kết quả** — luôn để app hiện popup. Việc có hiện
popup hay không do trường `showModal` quyết định:

- **Game theo level** (snake, arrow, ballsort, number-connection…): gửi
  `showModal: true` cho **mọi** kết thúc ván. App hiện popup thắng/thua/chúc mừng;
  game **không bao giờ** tự vẽ modal. Khi người chơi bấm nút trên popup, app gọi
  callback `window.onNextLevel`/`onRetryLevel` (xem §1.3). **Riêng thắng level cuối**
  bắn `victory` thay cho `game_result` (xem §1.5).
- **Game dạng điểm cao / không level** (high-score, endless, tictactoe…): gửi
  `game_result` với `level: null` và **`result: null`** (game không level không phân
  biệt thắng/thua), `showModal: false` (mặc định). App **không** hiện popup; game tự
  xử lý màn kết quả theo cách riêng (bảng điểm, restart…). Khi chơi lại bắn
  `retry_level`.

> **Tóm lại:** game theo level luôn `showModal: true` (app lo popup, game không tự
> vẽ modal trong **mọi** trường hợp). Chỉ game **không level** mới `showModal: false`
> (game tự lo màn kết quả).

```js
// Game theo level — app sẽ hiện popup thắng/thua (game KHÔNG tự vẽ modal):
sendMessage('game_result', { result: 'win',  showModal: true });
sendMessage('game_result', { result: 'lose', showModal: true });

// Game điểm cao / không level — result = null, không hiện popup:
sendMessage('game_result');                  // result=null, showModal mặc định false
```

> `showModal` chỉ có ý nghĩa với `game_result`. Các event khác (`next_level`,
> `save_data`, `quit`…) bỏ qua trường này (luôn `false`).

### 1.2. Lưu data: `save_data` (chỉ data NHẸ) + localStorage (trạng thái ván)

Có **hai tầng lưu** tách biệt theo độ "nặng" của dữ liệu:

| Tầng | Nội dung | Cơ chế |
|------|----------|--------|
| **`save_data` (native)** | **Data nhẹ**: điểm cao (`best`), `streak`, tiến độ (`levelIdx`), cờ `tutorialSeen`, config… | Bắn event `save_data`, native ghi xuống storage app |
| **localStorage** | **Trạng thái ván hiện tại** (nặng): board/cols/stock, `history`/`redoStack`, lá đang chọn… | Game tự ghi `localStorage`, không gửi native |

> **Quy tắc:** `save_data` **chỉ** mang **data nhẹ** (vài trường gọn nhẹ). **KHÔNG**
> gửi trạng thái ván đang chơi dở qua `save_data` — snapshot ván (cols/stock/
> history…) có thể rất lớn, gửi native mỗi nước đi sẽ tốn băng thông cầu nối &
> chậm. Ván dở **lưu localStorage** ngay trong WebView, khôi phục khi mở lại.

**`save_data` — chiều ngược của input `data` (§4):** native lưu chuỗi nhẹ này, lần
mở game sau truyền lại qua `window.data` / `?data=`. Không dùng `result` cho việc này.

`data` **luôn là string**. Nếu là object/JSON thì game tự `JSON.stringify` trước
khi gửi.

Vì đi qua `sendMessage`, message `save_data` **vẫn tự kèm `level`** (1-indexed) như
mọi event khác — game có level thì native nhận đủ `{ type, level, data }`, biết
data này thuộc level nào. Game không có level thì `level` là `null`.

```js
/** Lưu data NHẸ lên native. payload là string (hoặc object → tự stringify).
 *  CHỈ truyền data nhẹ (best, streak, levelIdx, tutorialSeen…) — KHÔNG truyền
 *  snapshot ván đang chơi (xem saveLocal bên dưới). */
function saveData(payload) {
  const data = typeof payload === 'string' ? payload : JSON.stringify(payload);
  sendMessage('save_data', { data });   // tự kèm level hiện tại (null nếu game không level)
}

// Vd: lưu data nhẹ mỗi khi qua màn → native nhận { type:'save_data', level, data, … }
saveData({ best, streak, levelIdx });
```

> `save_data` chỉ gửi data, không phải event "kết thúc ván" — có thể bắn bất cứ
> lúc nào cần persist (qua màn, thắng/thua). Native chịu trách nhiệm ghi
> đè (không append).

**Trạng thái ván hiện tại → `localStorage` (không qua native).** Snapshot ván đang
chơi dở (cols/stock/history…) lưu thẳng vào `localStorage` của WebView, bọc
`try/catch` (chế độ riêng tư / hết quota không được throw):

```js
const LS_KEY = 'mygame_save';   // key riêng mỗi game

/** Ghi TOÀN BỘ trạng thái ván vào localStorage để khôi phục khi mở lại. */
function saveLocal(payload) {
  try {
    const data = typeof payload === 'string' ? payload : JSON.stringify(payload);
    localStorage.setItem(LS_KEY, data);
  } catch (e) { /* private mode / quota — bỏ qua */ }
}
function readLocal() {
  try { return localStorage.getItem(LS_KEY) || ''; } catch (e) { return ''; }
}

/** Persist sau mỗi thay đổi: data nhẹ → native, ván nặng → localStorage. */
function persistState() {
  const light = { best, streak, levelIdx, tutorialSeen };
  saveData(light);                                       // nhẹ → native
  saveLocal({ ...light, game: gameSnapshot() });         // nhẹ + ván nặng → localStorage
}
```

> `gameSnapshot()` trả `null` khi không có ván active → lúc khôi phục, `game` rỗng
> thì bỏ qua (không restore), vào ván mới luôn. Không cần xóa localStorage thủ công.

**Game điểm cao — lưu `best` ngay khi đổi, HUD refresh khi hết màn.** Với game
**không level** (điểm cao / endless):

- **Persist:** mỗi khi `best` (điểm cao nhất) thay đổi thì gọi `save_data` **ngay**
  tại thời điểm đó — không chờ game over. Nhờ vậy điểm cao **không bị mất** nếu app
  bị kill / người chơi thoát đột ngột giữa ván.
- **Hiển thị:** HUD "Best" **không** cập nhật live theo từng nước; chỉ **refresh khi
  hết màn** (game over) và khi vào ván mới. Trong ván, ô Score chạy còn ô Best giữ
  nguyên kỷ lục cũ — để giá trị `best` lưu (ngầm) và giá trị hiển thị tách biệt.

```js
// Lưu best NGAY khi đổi (persist), nhưng KHÔNG đụng HUD Best ở đây.
function addScore(points) {
  score += points;
  updateScoreHud();             // chỉ cập nhật ô Score
  if (score > best) {           // best vừa đổi → persist NGAY lên native
    best = score;
    saveData({ best });         // save_data với data = JSON {"best":…}
  }
}

// HUD Best chỉ gọi khi hết màn / vào ván mới — không gọi trong addScore.
function updateBestHud() { elBest.textContent = String(best); }
```

> Vì `best` được lưu liên tục trong ván, lúc game over **không cần** gọi `save_data`
> lại nữa (best đã ở native rồi). Native ghi đè nên gọi nhiều lần vô hại.

### 1.3. Callback native → game: pause / next / retry

Ngoài việc inject biến (§2–4), native còn **gọi hàm trực tiếp** vào game để điều
khiển ván chơi (vd khi app mất focus, hoặc người chơi bấm nút trên popup do app
hiện ở §1.1). Game **cần định nghĩa sẵn** các hàm sau trên `window` — native sẽ gọi
khi cần. Hàm nào game không dùng vẫn nên định nghĩa rỗng (no-op) để native gọi an
toàn, không throw.

| Hàm                   | Khi nào native gọi | Game cần làm |
|-----------------------|--------------------|--------------|
| `window.onAppPause`   | App mất focus / chuyển nền / mở popup. | **Pause game nếu có** (dừng loop, timer, animation). Không reset state. |
| `window.onNextLevel`  | Người chơi bấm **Next Level** trên popup do app hiện. | Chuyển sang **level tiếp theo** và chơi tiếp. |
| `window.onRetryLevel` | Người chơi bấm **Retry / Play Again** trên popup do app hiện. | **Chơi lại từ đầu level hiện tại** (reset state level đang chơi). |

```js
// Pause game nếu game có vòng lặp/timer. Game tĩnh thì để no-op.
window.onAppPause = function () {
  if (typeof pauseGame === 'function') pauseGame();
};

// App gọi khi người chơi bấm Next Level trên popup → qua level kế tiếp.
// (Ở level cuối thường không có nút Next; guard dưới chỉ là phòng vệ.)
window.onNextLevel = function () {
  if (currentLevelIdx < LEVELS.length - 1) {
    loadLevel(currentLevelIdx + 1);
  }
};

// App gọi khi người chơi bấm Retry/Play Again → chơi lại từ đầu level hiện tại.
window.onRetryLevel = function () {
  loadLevel(currentLevelIdx); // reset & restart level đang chơi
};
```

> **Ai bắn `next_level`/`retry_level`?** Phân định theo `showModal` (§1.1) để hai cơ
> chế không chồng chéo:
>
> - **`showModal: true`** (game theo level — app vẽ popup): game bắn `game_result` →
>   app hiện popup → người chơi bấm nút → app gọi `window.onNextLevel` /
>   `window.onRetryLevel`. Game **không** bắn `next_level`/`retry_level` và **không**
>   tự vẽ popup.
> - **`showModal: false`** (game không level — game tự lo màn kết quả): game tự xử lý
>   nút Play Again và **bắn** `retry_level` lên native để báo chơi lại. App **không**
>   gọi callback trong luồng này.
>
> Định nghĩa các hàm **trước** khi boot game (hoặc ngay trong scope toàn cục) để
> native gọi lúc nào cũng có sẵn.

### 1.4. Nút Back (thoát game): `quit`

Mỗi game **tự vẽ một nút Back** để người chơi thoát game. Khi bấm, game bắn event
`quit` để native đóng WebView / quay về màn trước.

**Icon Back** — dùng đúng SVG mũi tên này (không tự vẽ icon khác để đồng bộ giữa các
game). `fill="currentColor"` để icon ăn theo màu chữ của nút:

```html
<svg viewBox="0 0 447.243 447.243" xmlns="http://www.w3.org/2000/svg" aria-hidden="true">
  <path fill="currentColor" d="M420.361 192.229a31.967 31.967 0 0 0-5.535-.41H99.305l6.88-3.2a63.998 63.998 0 0 0 18.08-12.8l88.48-88.48c11.653-11.124 13.611-29.019 4.64-42.4-10.441-14.259-30.464-17.355-44.724-6.914a32.018 32.018 0 0 0-3.276 2.754l-160 160c-12.504 12.49-12.515 32.751-.025 45.255l.025.025 160 160c12.514 12.479 32.775 12.451 45.255-.063a32.084 32.084 0 0 0 2.745-3.137c8.971-13.381 7.013-31.276-4.64-42.4l-88.32-88.64a64.002 64.002 0 0 0-16-11.68l-9.6-4.32h314.24c16.347.607 30.689-10.812 33.76-26.88 2.829-17.445-9.019-33.88-26.464-36.71z"/>
</svg>
```

**Vị trí:** đặt nút Back trong HUD ở góc trên (thường **góc trên bên trái**), ngay
dưới status bar (đã cộng `statusBarHeight + 8px`, xem §2).

Nên dùng component `btn3d` dạng icon vuông (`.btn-icon`, xem
[button-common.md](button-common.md)) để đồng bộ với các nút khác:

```html
<div class="btn3d btn-icon" id="back-btn" role="button" aria-label="Back">
  <span class="btn3d-shadow"></span>
  <span class="btn3d-surface">
    <svg class="back-ico" viewBox="0 0 447.243 447.243" aria-hidden="true">
      <path fill="currentColor" d="M420.361 192.229a31.967 31.967 0 0 0-5.535-.41H99.305l6.88-3.2a63.998 63.998 0 0 0 18.08-12.8l88.48-88.48c11.653-11.124 13.611-29.019 4.64-42.4-10.441-14.259-30.464-17.355-44.724-6.914a32.018 32.018 0 0 0-3.276 2.754l-160 160c-12.504 12.49-12.515 32.751-.025 45.255l.025.025 160 160c12.514 12.479 32.775 12.451 45.255-.063a32.084 32.084 0 0 0 2.745-3.137c8.971-13.381 7.013-31.276-4.64-42.4l-88.32-88.64a64.002 64.002 0 0 0-16-11.68l-9.6-4.32h314.24c16.347.607 30.689-10.812 33.76-26.88 2.829-17.445-9.019-33.88-26.464-36.71z"/>
    </svg>
  </span>
</div>
```

```css
/* icon chiếm ~52% bề mặt nút, ăn theo màu chữ qua currentColor */
.back-ico { width: 52%; height: 52%; fill: currentColor; display: block; }
```

```js
// Bấm Back → báo native thoát game.
document.getElementById('back-btn').addEventListener('click', e => {
  e.stopPropagation();
  sendMessage('quit');   // { type:'quit', level, result:null, data:null, showModal:false }
});
```

> `quit` chỉ báo ý định thoát; **native** chịu trách nhiệm đóng/điều hướng
> WebView. Game không tự "tắt" gì thêm.

### 1.5. Hoàn thành toàn bộ game: `victory`

Khi người chơi clear **level cuối cùng** (`currentLevelIdx === LEVELS.length - 1`),
game **chỉ bắn event `victory`** (KHÔNG bắn `game_result`) — app tự hiện popup chúc
mừng / điều hướng. `victory` thay luôn vai trò của `game_result` cho ván cuối, nên
**không cần** bắn cả hai.

```js
function winLevel() {
  if (currentLevelIdx === LEVELS.length - 1) {
    sendMessage('victory');   // level cuối: CHỈ bắn victory (không bắn game_result)
  } else {
    sendMessage('game_result', { result: 'win', showModal: true }); // app hiện popup
  }
}
```

> Mỗi kết thúc ván vẫn chỉ **một** event: level thường → `game_result`; level cuối →
> `victory`. App chịu trách nhiệm hiển thị popup và điều hướng (Next/Retry/Home) —
> game **không** tự vẽ popup hay nút Go to Home.

### 1.6. Nút Restart (chơi lại level hiện tại)

Game **có level** có thể tự vẽ một **nút Restart** trong HUD để người chơi reset
nhanh **level đang chơi** ngay tại chỗ (không qua popup của app). Restart **không
bắn event lên native** — nó chỉ gọi nội bộ `loadLevel(currentLevelIdx)` để dựng lại
trạng thái level từ đầu. (Khác với luồng "Retry" trên popup do app hiện ở §1.3:
luồng đó do app gọi `window.onRetryLevel`; còn nút Restart là tiện ích reset tức
thời của riêng game.)

**Icon Restart** — dùng đúng SVG mũi tên xoay này (không tự vẽ icon khác để đồng bộ
giữa các game). `fill="currentColor"` để icon ăn theo màu chữ của nút:

```html
<svg viewBox="0 0 100 100" xmlns="http://www.w3.org/2000/svg" aria-hidden="true">
  <path fill="currentColor" d="m50.7 19.003 4.793-4.793c2.696-2.697 2.696-6.991 0-9.688s-6.991-2.696-9.687 0L29.327 21.001a6.812 6.812 0 0 0 0 9.687l16.479 16.478a6.9 6.9 0 0 0 4.793 1.998c1.798 0 3.496-.7 4.794-1.998a6.812 6.812 0 0 0 0-9.687l-4.794-4.794c13.682.4 24.768 11.685 24.768 25.467 0 14.081-11.485 25.466-25.467 25.466S24.534 72.233 24.534 58.252c0-3.096.599-6.192 1.597-8.989 1.299-3.495-.4-7.49-3.994-8.788-3.496-1.298-7.49.4-8.789 3.995-1.698 4.394-2.496 9.088-2.496 13.882C10.852 79.923 28.428 97.5 50 97.5s39.148-17.577 39.148-39.148c0-21.472-17.177-38.95-38.449-39.349z"/>
</svg>
```

**Vị trí:** đặt nút Restart trong HUD ở góc trên (thường **góc trên bên phải**), đối
xứng với nút Back ở góc trái, ngay dưới status bar (xem §2).

Nên dùng component `btn3d` dạng icon vuông (`.btn-icon`) để đồng bộ với nút Back:

```html
<div class="btn3d btn-icon" id="restart-btn" role="button" aria-label="Restart">
  <span class="btn3d-shadow"></span>
  <span class="btn3d-surface">
    <svg class="restart-ico" viewBox="0 0 100 100" aria-hidden="true">
      <path fill="currentColor" d="m50.7 19.003 4.793-4.793c2.696-2.697 2.696-6.991 0-9.688s-6.991-2.696-9.687 0L29.327 21.001a6.812 6.812 0 0 0 0 9.687l16.479 16.478a6.9 6.9 0 0 0 4.793 1.998c1.798 0 3.496-.7 4.794-1.998a6.812 6.812 0 0 0 0-9.687l-4.794-4.794c13.682.4 24.768 11.685 24.768 25.467 0 14.081-11.485 25.466-25.467 25.466S24.534 72.233 24.534 58.252c0-3.096.599-6.192 1.597-8.989 1.299-3.495-.4-7.49-3.994-8.788-3.496-1.298-7.49.4-8.789 3.995-1.698 4.394-2.496 9.088-2.496 13.882C10.852 79.923 28.428 97.5 50 97.5s39.148-17.577 39.148-39.148c0-21.472-17.177-38.95-38.449-39.349z"/>
    </svg>
  </span>
</div>
```

```css
/* icon refresh nét mảnh hơn icon Back → cho to hơn (~62%), ăn theo màu chữ */
.restart-ico { width: 62%; height: 62%; fill: currentColor; display: block; }
```

```js
// Bấm Restart → dựng lại level hiện tại (KHÔNG bắn event lên native).
document.getElementById('restart-btn').addEventListener('click', e => {
  e.stopPropagation();
  loadLevel(currentLevelIdx);   // reset & restart level đang chơi
});
```

> Restart là tuỳ chọn (chỉ game có level mới cần). Nó **không** gửi `retry_level`
> hay event nào khác — chỉ reset trong game. Game không có level dùng nút Play Again
> theo §1.1 (bắn `retry_level`), không dùng nút Restart này.

### 1.7. Yêu cầu hiển thị quảng cáo fullscreen: `ads`

Khi game muốn **app hiển thị quảng cáo toàn màn hình (fullscreen)**, chỉ cần bắn event
`ads` — không kèm trường nào khác. App tự lo việc hiện quảng cáo. Việc **có sẵn ad để
hiện hay không** do app quyết định; game chỉ **báo ý định**, không tự vẽ/đợi kết quả ad.

```js
// Yêu cầu app hiện quảng cáo fullscreen.
sendMessage('ads');   // { type:'ads', level, result:null, data:null, showModal:false }

// Vd: hiện quảng cáo khi qua màn
sendMessage('ads');
```

> `ads` chỉ là **yêu cầu một chiều** — game không nhận lại callback "ad đã xong" qua
> kênh này. Nếu cần tiếp tục luồng sau khi đóng ad (vd vào level kế), app dùng các
> callback có sẵn ở §1.3 (`onNextLevel`/`onRetryLevel`).

---

## 2. native → game: `statusBarHeight`

Chiều cao status bar (đơn vị **dp**) để game chừa padding-top, tránh nội dung bị
che bởi notch/status bar.

Đọc theo thứ tự ưu tiên:

1. URL `?statusBarHeight=` — **ưu tiên** (đơn vị dp chuẩn).
2. `window.statusBarHeight` — fallback (có thể bị nhân `PixelRatio` nên chỉ dùng
   khi thiếu URL).
3. `env(safe-area-inset-top)` — OS tự báo qua `viewport-fit=cover` (chuẩn iOS/notch
   kể cả khi native truyền `0`).

Padding cuối = `max(statusBarHeight, envSafeTop) + 8px` đệm.

```js
function getStatusBarHeight() {
  const params = new URLSearchParams(window.location.search);
  const fromUrl = params.get('statusBarHeight');
  const fromWin = typeof window.statusBarHeight !== 'undefined'
    ? window.statusBarHeight : null;
  let h = Number(fromUrl != null ? fromUrl : fromWin);
  if (Number.isNaN(h) || h < 0) h = 0;   // giá trị bẩn → 0
  return h;
}
```

> `getStatusBarHeight()` chỉ trả về **chiều cao thô** (dp). Bước cộng `8px` đệm và
> kết hợp `env(safe-area-inset-top)` (ưu tiên thứ 3) làm ở tầng dùng — vd set
> `padding-top` của header/HUD = `max(getStatusBarHeight(), envSafeTop) + 8`px.

---

## 3. native → game: `currentLevel` (game có level)

Level khởi tạo khi mở game (vd resume đúng level đã lưu). **1-indexed**, khớp với
protocol message. Đọc theo thứ tự:

1. `window.currentLevel` (number) — native inject — **ưu tiên**.
2. URL `?currentLevel=`.

Clamp vào `[1, LEVELS.length]`, convert sang 0-indexed cho `loadLevel`. Mặc định
`0` (level đầu) nếu không có / không hợp lệ.

```js
function getInitialLevelIdx() {
  const fromWin = typeof window.currentLevel === 'number'
    ? window.currentLevel : null;
  const fromUrl = new URLSearchParams(window.location.search).get('currentLevel');
  const raw = Number(fromWin != null ? fromWin : fromUrl);
  if (!Number.isFinite(raw) || raw < 1) return 0;
  return Math.min(Math.floor(raw), LEVELS.length) - 1; // → 0-indexed
}
```

### Đợi native inject (chống race)

Trên Android, `injectedJavaScriptBeforeContentLoaded` với HTML local có thể chạy
**sau** inline script. Vì vậy poll ngắn chờ `window.currentLevel`, fallback boot
sau **200ms** nếu inject không tới (vd mở bằng browser thường). Sentinel: native
truyền `0` nghĩa là "chưa có level lưu" → game tự dùng mặc định.

```js
(function waitForNativeInjection() {
  const start = Date.now();
  (function check() {
    if (typeof window.currentLevel === 'number' || Date.now() - start > 200) {
      bootGame();
    } else {
      setTimeout(check, 10);
    }
  })();
})();
```

---

## 4. native → game: `data` (app truyền chuỗi dữ liệu)

Khi app cần truyền **dữ liệu khởi tạo dạng chuỗi NHẸ** vào game (vd config, best,
streak, tiến độ, theme…), dùng input `data`. Cùng pattern đọc (window + URL)
như `currentLevel` — **ưu tiên `window`** (lưu ý: riêng `statusBarHeight` ở §2 ưu
tiên URL). Trạng thái ván đang chơi dở **không** đến từ đây mà từ `localStorage`
(xem §1.2) — `parseInitialData` gộp cả hai nguồn.

1. `window.data` (string) — native inject — **ưu tiên**.
2. URL `?data=` (đã `encodeURIComponent`).

`data` **luôn là string**. Nếu là JSON thì game tự `JSON.parse` trong `try/catch`;
parse lỗi → fallback mặc định, không throw.

```js
/** Đọc data khởi tạo từ native. Luôn trả về string (rỗng nếu không có). */
function getInitialData() {
  const fromWin = typeof window.data === 'string' ? window.data : null;
  const fromUrl = new URLSearchParams(window.location.search).get('data');
  return fromWin != null ? fromWin : (fromUrl || '');
}

/** Gộp data nhẹ (native) + trạng thái ván (localStorage).
 *  - Phần nhẹ (best/streak/levelIdx…) ưu tiên lấy từ native.
 *  - Ván đang chơi dở (game) lấy từ localStorage (native không giữ phần này). */
function parseInitialData() {
  let nativeObj = null;
  const raw = getInitialData();
  if (raw) { try { nativeObj = JSON.parse(raw); } catch (e) {} }

  let localObj = null;
  const lraw = readLocal();
  if (lraw) { try { localObj = JSON.parse(lraw); } catch (e) {} }

  if (!nativeObj && !localObj) return null;
  const merged = Object.assign({}, localObj || {}, nativeObj || {});  // native thắng phần nhẹ
  if (localObj && localObj.game) merged.game = localObj.game;          // ván nặng từ local
  return merged;
}
```

> Nếu game cần đợi `data` được inject (giống `currentLevel`), gộp điều kiện vào
> cùng vòng `waitForNativeInjection`: chỉ boot khi đã có `window.data` (hoặc hết
> 200ms), tránh phải poll 2 vòng riêng.

---

## 5. native → game: `language` (localize app)

Native truyền **mã ngôn ngữ hiện tại của app** vào game để game tự dịch (localize)
toàn bộ text UI. Mã ngôn ngữ là một trong các **LocaleCode** hỗ trợ ở app (đồng bộ
với `src/i18n`). Cùng pattern đọc (window + URL) như `data`/`currentLevel` —
**ưu tiên `window`**:

1. `window.language` (string) — native inject — **ưu tiên**.
2. URL `?language=` (vd `?language=zh-Hans`).

Mặc định / fallback là **`en`** (tiếng Anh) nếu không có hoặc mã không nằm trong
danh sách hỗ trợ. Mã ngôn ngữ phân biệt hoa/thường đúng như bảng dưới (vd
`zh-Hans`, `pt-BR`, `es-MX`).

### Danh sách ngôn ngữ hỗ trợ

Đồng bộ với `src/i18n/locales/*.json` — mỗi mã là một file JSON ngôn ngữ:

| Mã (`LocaleCode`) | Ngôn ngữ                  |
|-------------------|---------------------------|
| `en`              | English (mặc định)        |
| `vi`              | Tiếng Việt                |
| `zh-Hans`         | 简体中文 (Trung giản thể) |
| `zh-Hant`         | 繁體中文 (Trung phồn thể) |
| `ja`              | 日本語 (Nhật)             |
| `ko`              | 한국어 (Hàn)              |
| `es`              | Español (Tây Ban Nha)     |
| `es-MX`           | Español (México)          |
| `es-AR`           | Español (Argentina)       |
| `pt-BR`           | Português (Brasil)        |
| `pt`              | Português (Bồ Đào Nha)    |
| `fr`              | Français (Pháp)           |
| `de`              | Deutsch (Đức)             |
| `it`              | Italiano (Italia)         |
| `nl`              | Nederlands (Hà Lan)       |
| `pl`              | Polski (Ba Lan)           |
| `ru`              | Русский (Nga)             |
| `tr`              | Türkçe (Thổ Nhĩ Kỳ)       |
| `ar`              | العربية (Ả Rập)           |
| `hi`              | हिन्दी (Ấn Độ – Hindi)    |
| `th`              | ไทย (Thái Lan)            |
| `id`              | Bahasa Indonesia          |
| `fil`             | Filipino (Philippines)    |

### Đọc `language` trong game

```js
// Danh sách mã ngôn ngữ game có bản dịch — khớp với key trong I18N dưới.
const SUPPORTED_LANGS = [
  'en','vi','zh-Hans','zh-Hant','ja','ko','es','es-MX','es-AR','pt-BR','pt',
  'fr','de','it','nl','pl','ru','tr','ar','hi','th','id','fil',
];
const DEFAULT_LANG = 'en';

/** Đọc mã ngôn ngữ từ native. Trả về 1 mã hợp lệ, fallback 'en'. */
function getLanguage() {
  const fromWin = typeof window.language === 'string' ? window.language : null;
  const fromUrl = new URLSearchParams(window.location.search).get('language');
  const raw = fromWin != null ? fromWin : fromUrl;
  return SUPPORTED_LANGS.includes(raw) ? raw : DEFAULT_LANG;
}
```

### Dùng để dịch text

> **Bắt buộc: toàn bộ localize phải gộp INLINE trong chính `index.html`.** Bảng dịch
> (`I18N`), hàm `getLanguage()`/`t()` đều viết thẳng trong `<script>` của file HTML
> — **không** tách ra file `.json`/`.js` ngoài, **không** `fetch`/`import` bản dịch.
> Lý do: mỗi game là **một file HTML đơn**, native nạp & cache theo file đó; tách
> file phụ sẽ vỡ luồng tải/offline. Cả game phải tự chứa.

Game tự khai một bảng dịch `key → text` cho từng ngôn ngữ ngay trong file, rồi tra
theo `language`, thiếu key → fallback `en`. (Không bắt buộc dịch đủ 23 ngôn ngữ —
ngôn ngữ nào thiếu sẽ tự rơi về `en`.)

```html
<!-- ngay trong <script> của index.html — KHÔNG để file ngoài -->
<script>
  const I18N = {
    en: { score: 'Score', best: 'Best', playAgain: 'Play Again', gameOver: 'Game Over' },
    vi: { score: 'Điểm',  best: 'Cao nhất', playAgain: 'Chơi lại', gameOver: 'Thua rồi' },
    'zh-Hans': { score: '分数', best: '最高', playAgain: '再玩一次', gameOver: '游戏结束' },
    // … các ngôn ngữ khác
  };

  const LANG = getLanguage();
  /** Lấy text theo key: ngôn ngữ hiện tại → fallback 'en' → cuối cùng trả về key. */
  function t(key) {
    return (I18N[LANG] && I18N[LANG][key]) || I18N.en[key] || key;
  }

  // Vd dùng: render UI theo ngôn ngữ
  elScoreLabel.textContent = t('score');   // 'Điểm' nếu LANG === 'vi'
</script>
```

> Đọc `language` **một lần lúc boot** là đủ — app không đổi ngôn ngữ giữa lúc đang
> chơi (đổi ngôn ngữ sẽ reload lại game). Nếu game cần đợi `language` được inject
> (giống `currentLevel`/`data`), gộp điều kiện vào cùng vòng `waitForNativeInjection`
> ở §3.

---

## Tóm tắt luồng

```
                 ┌────────────────────────── native (RN WebView) ──────────────────────────┐
   inject/URL →  │  statusBarHeight (dp)  currentLevel (1-indexed)  data (string)  language  │
   call      →  │  window.onAppPause()   window.onNextLevel()   window.onRetryLevel()       │
                 └──────────────────────────────────┬──────────────────────────────────────┘
                                                     ▼
                              ┌──────────────── game HTML ────────────────┐
   bridge ←  postMessage  ←   │  sendMessage(type, { result, data, showModal })
   { type, level, result, data, showModal }                                │
   types: game_result · retry_level · next_level · victory · save_data · quit · ads
                              │  + localStorage  ←  trạng thái ván dở (nặng, không qua native)
                              └────────────────────────────────────────────┘
```

> **Lưu trữ 2 tầng (§1.2):** `save_data` → native = **data nhẹ** (best/streak/tiến
> độ). `localStorage` → **trạng thái ván hiện tại** (board/history…, nặng). Lúc boot
> `parseInitialData` gộp: phần nhẹ ưu tiên native, ván dở lấy từ localStorage.