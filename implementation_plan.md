# Fix: Lưu & Khôi phục ván dở khi Refresh — 4 Game

## Vấn đề

Mentor: "cứ refresh là chơi lại từ đầu, đúng ra là phải cho chơi tiếp."

Theo `game-common.md §1.2`, mỗi game cần **2 tầng lưu**:
- **`save_data` → native**: data nhẹ (best, tut, …)
- **`localStorage`**: trạng thái ván hiện tại (board, tower, score, …)

---

## Chẩn đoán

| Game | Type | localStorage có gì? | Restore khi boot? |
|---|---|---|---|
| **animal-connect** | Level-based | ✅ Grid + score + time + levelIdx | ✅ `tryRestoreState()` — ĐÃ ĐÚNG |
| **runic-blaze** | Endless/score | ❌ Chỉ `{best, muted, tutorialSeen}` | ❌ Luôn `newRun()` |
| **hamster-jump** | Endless/score | ❌ Chỉ `{best, tut, mute}` | ❌ Luôn `resetWorld()` |
| **cursed-knives** | Endless/score | ❌ Chỉ `{best, tut, muted, deep, eS, eK}` | ❌ Luôn `newRun()` |

Chỉ cần fix 3 game: **runic-blaze**, **hamster-jump**, **cursed-knives**.

---

## Proposed Changes

### Game 1 — `runic-blaze/index.html`

**State cần lưu:** board snapshot (màu + kind mỗi tile), score, turnCount, cursedIdx, sinceCursed, nColors, nextSurge, nextStorm

**1a. `saveCurrentState()` — sửa stub thành lưu board thật:**
```js
function saveCurrentState() {
  if (ended || !board.length) return;
  const snap = board.map(row => row.map(t => t ? { c: t.color, k: t.kind } : null));
  saveLocal({ best, muted, tutorialSeen,
    game: { snap, score, turnCount, cursedIdx, sinceCursed,
            nColors, nextSurge, nextStorm } });
}
```
Gọi `saveCurrentState()` cuối `endCascade()` (board đã ổn định sau mỗi cascade).

**1b. `bootGame()` — kiểm tra saved.game trước `newRun()`:**
```js
const g = saved && saved.game;
if (g) tryRestoreRunState(g);
else   newRun();
```

**1c. Thêm `tryRestoreRunState(g)` — hàm mới:**
Tạo lại tiles từ `g.snap`, set `score/turnCount/...`, `busy = false`.

**1d. Khi ván kết thúc (game over):**
Gọi `saveLocal({ best, muted, tutorialSeen })` (không có `game`) → lần mở tiếp start fresh.

---

### Game 2 — `hamster-jump/index.html`

**State cần lưu:** `tower[]` array `{cx, kind, w, seed}`, score, combo

**2a. `persistSaved()` — thêm game snapshot:**
```js
function persistSaved() {
  const light = { best, tut: tutSeen ? 1 : 0, mute: muted ? 1 : 0 };
  saveData(light);
  const snap = (state === 'playing' && tower.length > 0)
    ? { tower: tower.map(({ cx, kind, w, seed }) => ({ cx, kind, w, seed })),
        score, combo }
    : null;
  saveLocal({ ...light, game: snap });
}
```
Gọi `persistSaved()` cuối `pinSlider()` (mỗi lần miếng đáp xuống tháp thành công).

**2b. `boot()` — kiểm tra saved.game trước `resetWorld()`:**
```js
const savedGame = saved.game;
if (savedGame && savedGame.tower && savedGame.tower.length > 0) {
  tryRestoreTower(savedGame);
} else {
  resetWorld();
  if (!tutSeen) startTutorial();
  else { state = 'playing'; scheduleCycle(); }
}
```

**2c. Thêm `tryRestoreTower(g)` — hàm mới:**
Set `tower = g.tower`, `score = g.score`, `combo = g.combo`, gọi `updateScoreHud()`, `state = 'playing'`, `scheduleCycle()`.

**2d. Khi game over:**
Gọi `persistSaved()` (state = 'over' → snap = null → tự xoá game) → lần tiếp start fresh.

---

### Game 3 — `cursed-knives/index.html`

**State cần lưu:** `score`, `stageIdx` (stage đang đánh), `knifeAmmo` còn lại

> [!NOTE]
> Không restore trạng thái vật lý (dao đang bay, góc target). Resume từ đầu round stageIdx đó với đúng score — player không mất tiến độ lớn.

**3a. `persistSaved()` — thêm game snapshot:**
```js
function persistSaved() {
  const light = { best, tut: tutSeen ? 1 : 0, muted: muted ? 1 : 0,
                  deep, eS: echoS, eK: echoK };
  saveData(light);
  const snap = (state === 'playing' && score > 0)
    ? { score, stageIdx, knifeAmmo }
    : null;
  saveLocal({ ...light, game: snap });
}
```
Gọi `persistSaved()` sau mỗi totem bị phá (stage transition).

**3b. `boot()` — kiểm tra saved game trước `newRun()`:**
```js
if (sv.game && sv.game.score > 0) {
  resumeRun(sv.game);
} else {
  if (!tutSeen) startTutorial();
  else newRun();
}
```

**3c. Thêm `resumeRun(g)` — hàm mới:**
Set `score`, `stageIdx`, `knifeAmmo`, update HUD, gọi `spawnTotem(stageIdx)`.

**3d. Khi game over:**
`persistSaved()` (state = 'over' → snap = null) → lần tiếp start fresh.

---

## Verification Plan

1. Chơi vài moves → refresh → verify game tiếp tục đúng chỗ
2. Chơi đến game over → refresh → verify start fresh (không restore ván đã thua)
3. `?reset=1` → verify xoá state
4. Kiểm tra `localStorage` trong DevTools

---

## Open Questions

> [!IMPORTANT]
> Tên chính xác các biến trong cursed-knives (`stageIdx`, `knifeAmmo`) cần verify khi đọc file trước khi code.
