# Popup kết thúc game (Game Over / New Best) — quy ước chung

Quy cách **popup kết quả khi hết ván** cho các game HTML, để 11 game nhìn đồng nhất.
Liên quan: [game-common.md](game-common.md) §1.1 (luồng `game_result` / `showModal`),
nút dùng component `btn3d` (xem phần §6 dưới).

> **Quy tắc cốt lõi:** popup chỉ có **MỘT tiêu đề**, tự đổi chữ theo điểm —
> **"New Best!"** khi phá kỷ lục (`score > best`), **"Game Over"** khi không
> (`score ≤ best`). **TUYỆT ĐỐI không** hiện "Game Over" lúc đang phá kỷ lục, và
> **không** dùng badge "New Best!" riêng tách khỏi tiêu đề.

## 1. Phạm vi áp dụng

- **Game điểm-cao / endless (không level):** game **TỰ VẼ** popup này. Khi thua gửi
  `sendMessage('game_result')` (`result:null`, `showModal:false`) rồi hiện popup. → **Áp dụng rule này.**
- **Game theo level:** **KHÔNG tự vẽ.** Gửi `sendMessage('game_result',{result:'win'/'lose',showModal:true})`
  (hoặc `sendMessage('victory')` ở level cuối) để **app tự hiện popup**. → Bỏ qua rule này.

## 2. Hai trạng thái (cùng một popup, chỉ khác tiêu đề)

| Phá kỷ lục (`score > best`) | Thua thường (`score ≤ best`) |
|---|---|
| ![New Best](popup-common-newbest.png) | ![Game Over](popup-common-gameover.png) |

Thứ tự các thành phần (trên → dưới): **Tiêu đề → khối điểm SCORE/BEST → nút Play Again.**
Không có dòng mô tả/subtitle phụ.

## 3. HTML

```html
<div id="overlay" class="overlay hidden"><div class="card">
  <div class="card-title" id="ov-title">Game Over</div>
  <div class="sb">
    <div><div class="k">SCORE</div><div class="v"      id="ov-score">0</div></div>
    <div><div class="k">BEST</div> <div class="v best" id="ov-best">0</div></div>
  </div>
  <button class="btn3d play" id="ov-btn" type="button">
    <span class="btn3d-shadow"></span><span class="btn3d-surface">Play Again</span>
  </button>
</div></div>
```

## 4. CSS

```css
/* Backdrop — phủ kín, mờ tối, fade in/out */
.overlay{position:absolute;inset:0;z-index:10;display:flex;align-items:center;justify-content:center;
  background:rgba(3,9,24,.7);opacity:1;transition:opacity .25s;}
.overlay.hidden{opacity:0;pointer-events:none;}

/* Card — bo góc lớn, gradient tối, đổ bóng, pop scale khi hiện */
.card{width:min(82%,360px);padding:26px;text-align:center;border-radius:22px;
  background:linear-gradient(180deg,#1c2f63,#0c1838);   /* gradient đổi theo theme từng game, xem §7 */
  border:2px solid rgba(120,160,255,.22);box-shadow:0 18px 50px rgba(0,0,0,.5);
  transform:scale(1);transition:transform .25s;}
.overlay.hidden .card{transform:scale(.9);}             /* card phóng 0.9→1 khi mở */

/* Tiêu đề (đổi chữ New Best! / Game Over bằng JS) */
.card-title{color:#fff;font-size:30px;margin-bottom:14px;}

/* Khối điểm 2 cột */
.sb{display:flex;justify-content:center;gap:30px;margin-bottom:18px;}
.sb .k{font-family:system-ui,sans-serif;font-size:11px;letter-spacing:2px;color:#8fb4ff;font-weight:800;} /* nhãn xanh dương */
.sb .v{font-size:32px;color:#fff;}     /* SCORE trắng */
.sb .v.best{color:#ffd23f;}            /* BEST vàng */
```

## 5. JS — hiện popup + đổi tiêu đề

```js
function gameOver(){
  const isNew = score > best;          // phá kỷ lục?
  if (isNew) best = score;             // điểm cao chỉ lưu lúc thua
  // ⬇️ MẤU CHỐT: đổi chữ tiêu đề, KHÔNG dùng badge riêng
  document.getElementById('ov-title').textContent = isNew ? 'New Best!' : 'Game Over';
  document.getElementById('ov-score').textContent = score;
  document.getElementById('ov-best').textContent  = best;
  fitScores();                         // ⬅️ tự co cỡ điểm cho vừa card khi điểm to (xem §5b)
  sendMessage('game_result');          // điểm-cao: result=null, showModal=false (xem game-common §1.1)
  // delay 0.3–0.7s cho animation thua chạy xong rồi mới hiện popup
  setTimeout(() => document.getElementById('overlay').classList.remove('hidden'), 500);
}

// Nút Play Again: báo native + chơi lại nội bộ
document.getElementById('ov-btn').addEventListener('click', () => {
  sendMessage('retry_level');
  newRun();   // reset điểm + ẩn popup: overlay.classList.add('hidden')
});
```

## 5b. Tự co cỡ điểm cho vừa card (auto-fit)

Cỡ chữ điểm cố định `32px` sẽ **tràn card khi điểm ≥ 6 chữ số** (vd `999,999`+ —
card chỉ chứa gọn tới 5 chữ số). Sau khi set SCORE/BEST, gọi `fitScores()` để giảm
cỡ chữ điểm cho khớp bề rộng card:

```js
function fitScores(){
  var r = document.querySelector('.sb');            // khối điểm 2 cột
  if (!r) return;
  var card = r.closest('.card');
  var vals = r.querySelectorAll('.v');
  vals.forEach(function(v){ v.style.fontSize=''; }); // reset về cỡ gốc (32px) trước
  var pad = parseFloat(getComputedStyle(card).paddingLeft) || 26;
  var max = card.clientWidth - 2*pad;
  var fz  = parseFloat(getComputedStyle(vals[0]).fontSize) || 32;
  while (r.scrollWidth > max && fz > 14){            // co dần tới khi vừa (sàn 14px)
    fz -= 1; vals.forEach(function(v){ v.style.fontSize = fz+'px'; });
  }
}
```

> Gọi `fitScores()` **mỗi lần** mở popup (ngay sau khi set điểm). `.sb` luôn được
> layout (overlay ẩn bằng `opacity:0`, **không** `display:none`) nên đo `scrollWidth`
> được cả khi popup chưa hiện. Reset `fontSize=''` trước để mỗi ván tính lại từ 32px,
> tránh "kẹt" ở cỡ nhỏ của ván trước. Điểm ngắn (≤5 chữ số) giữ nguyên 32px.

Ví dụ điểm 8 chữ số tự co cho vừa card (32px → 20px):

![Điểm to auto-fit](popup-common-bigscore.png)

## 6. Nút Play Again — component `btn3d` biến `.play` (amber)

```css
.btn3d{--radius:14px;--surface-color:#2f7fe0;--surface-text:#fff;
  --surface-height:44px;--surface-width:auto;--surface-padding-x:16px;--surface-font-size:22px;
  position:relative;display:inline-block;padding:0;border:0;background:transparent;
  cursor:pointer;font:inherit;color:inherit;}
.btn3d-shadow{display:none;}
.btn3d-surface{position:relative;z-index:1;display:flex;align-items:center;justify-content:center;
  gap:8px;height:var(--surface-height);width:var(--surface-width);padding:0 var(--surface-padding-x);
  border-radius:var(--radius);background-color:var(--surface-color);color:var(--surface-text);
  font-family:var(--ui-font);font-size:var(--surface-font-size);letter-spacing:.6px;
  transition:transform .09s ease-out;}
.btn3d:active{transform:scale(.97);}
.btn3d.play{--surface-height:52px;--surface-padding-x:34px;
  --surface-color:#ffb02e;--surface-text:#3a2400;--surface-font-size:17px;}  /* CTA amber */
```

## 7. Quy tắc màu (giữ đồng nhất giữa các game)

| Thành phần | Giá trị | Ghi chú |
|---|---|---|
| Tiêu đề | `#fff` trắng | dùng cho cả "New Best!" và "Game Over" |
| Nhãn SCORE/BEST | `#8fb4ff` xanh dương | **cố định** |
| Giá trị SCORE | `#fff` trắng | **cố định** |
| Giá trị BEST | `#ffd23f` vàng | **cố định** |
| Nút Play Again | `#ffb02e` amber (`btn3d.play`) | **cố định** |
| Gradient card | đổi theo theme game | vd minesweeper `#1c2f63→#0c1838`, sudoku `#15376e→#0a1f44`, stack-tower `#1a3f86→#0a2148` — **luôn tông tối** |

→ **Chỉ gradient card được phép đổi theo theme; các màu accent giữ nguyên** để mọi game đồng bộ.

## 8. Animation

- Backdrop: `opacity 0→1` trong `.25s`.
- Card: `transform scale .9→1` trong `.25s` (đồng thời với fade).
- Cơ chế: toggle class `.hidden` trên `#overlay` (bỏ `.hidden` = hiện).
  **KHÔNG** dùng `display:none/flex` (không transition được). Nếu game đang dùng cơ chế
  `.active`, đổi CSS sang opacity+scale như trên (giữ class `.active` cũng được, miễn là
  default ẩn bằng `opacity:0;pointer-events:none`).

## 9. DO / DON'T

✅ Một tiêu đề duy nhất, đổi chữ `isNew ? 'New Best!' : 'Game Over'`.
✅ Điểm-cao chỉ lưu `best` lúc thua; gửi `sendMessage('game_result')` (không tham số).
✅ Ẩn popup bằng class trên `#overlay` để có hiệu ứng fade+scale.
✅ Gọi `fitScores()` sau khi set điểm để **điểm to (≥6 chữ số) tự co cho vừa card**, không tràn.
❌ **Không** hiện "Game Over" khi đang phá kỷ lục.
❌ **Không** dùng badge "New Best!" tách riêng khỏi tiêu đề.
❌ **Không** nhét số điểm vào trường `result` (chỉ `'win'`/`'lose'`/`null`).
❌ **Không** thêm dòng mô tả/subtitle phụ; giữ đúng: tiêu đề → SCORE/BEST → nút.
❌ Game theo level **không** tự vẽ popup này (để app lo, `showModal:true`).
