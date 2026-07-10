# StickerForge Studio — 處理邏輯說明

> 對應 `index.html`，HEAD `6d87286`。行號為撰寫時的位置，之後可能漂移；每項聲明皆經抽驗核對原始碼。
> 本文由三個子代理平行萃取（GIF pipeline / TGS pipeline / 平台規格），再經交叉審核。子代理報告產出時安全分類器一度離線，故所有載入判斷（尤其 WebP 無損）皆由我在瀏覽器實測覆核，見各節註記。

---

## 0. 總體架構

App 是**單一 HTML 檔、單一 IIFE**（約 1720–5048 行），內含兩條獨立轉檔管線，共用一組底層編碼器：

| | 輸入 | 狀態陣列 | 面板建構 |
|---|---|---|---|
| **GIF 管線** | pixel-art GIF | `items[]` | `panelHtml` / `addFiles` |
| **TGS 管線** | Telegram `.tgs`（gzip Lottie） | `tgsItems[]` | `_tgsPanelHtml` / `_addTgsFiles` |

**共用底層**：`encodeApng`（APNG 編碼）、`muxAnimatedWebp`（WebP 封裝）、`quantizeFrames`（減色）、`resampleFramesToFps`（降幀）、`fit3sDelays`（壓時長）、`C.boundingBox` / `C.cropRgba` / `C.diffBox`（區域運算）、`CompressionStream`（deflate / gzip）。

底層畫布常數 `CANVAS_SIZE = 512`（1182），UI 內別名 `CS`（1710）。

---

## 1. GIF 管線

### 1.1 解碼 — `decodeGif`（~1070，`PixelCore` IIFE 內）

- 簽章檢查 `/^GIF8[79]a$/`（1076），失敗擲 `"Not a GIF file"`。
- **disposal（處置方法）解析**（1121–1137），套用在**前一幀**矩形上，再畫當前幀：
  - `disposal === 2`（restore-to-background）：前幀矩形像素歸零 `0,0,0,0`。
  - `disposal === 3`（restore-to-previous）：由 `snapshot` 還原畫布；snapshot 在畫幀**前**擷取（1137）。
  - `disposal === 0`/`1`（do-not-dispose）：無特例分支，共用畫布原樣保留 —— 符合 GIF 規範，但為隱式行為。
- **合成**（1150–1167）：只有 index `!== transparentIndex` 的像素寫入畫布，透明像素讓底下已合成內容透出。
- 支援 4-pass interlace（`INTERLACE_PASSES`，1067）。
- **延遲換算與夾制**（1170）：
  ```js
  const raw = delay * 10;                                  // GIF 1/100s → ms
  frames.push({ rgba: canvas.slice(), delayMs: raw < 20 ? 100 : raw });
  ```
  任何 `raw < 20ms`（GIF delay 值 0 或 1）夾到 **100ms 預設**，對齊瀏覽器對壞 GIF 的常見行為。
- 回傳 `{ width, height, frames }`，每幀 `rgba` 皆為**完整合成、無殘影**的 buffer —— 下游永遠拿到完整幀，不處理增量幀。

### 1.2 幀選取 — `activeFrames` / `cacheKey`（1867–1892）

- `cacheKey` = `[fpsMode, fps, rangeStart, rangeEnd, skip].join("|")`（1867）—— **不含** scaleMode/apngMaxColors，因為 `activeFrames` 輸出停在原始解析度，縮放與減色在下游才做。memoize 於 `it._framesKey`。
- 範圍：`start = clamp(rangeStart, 0, n-1)`、`end = clamp(rangeEnd, start, n-1)`。
- skip → step：`step = 1 + clamp(s.skip, 0, 60)`（1881）—— 「skip N」把 N+1 個連續來源幀合成 1 個輸出幀，合成幀的延遲為各幀延遲**加總**。
- **manual fps 覆蓋一切**（1886）：`if (s.fpsMode === "manual") delay = 1000 / clamp(s.fps, 1, 60)` —— 加總延遲被丟棄，每幀改為均勻延遲。skip 仍決定用**哪些**幀，但不再決定其時序。

### 1.3 縮放 — `effScale` / `dcScale` / `scaleAndCenter`（1182–1213, 1821–1833）

- `maxFitScale(w,h,size)` = `max(1, floor(size / max(w,h)))` —— 塞得下的最大整數倍。
- `effScale`（1821）：`auto` → `maxFitScale`（塞滿 512 的最大倍率）；`manual` → `effectiveScale`（取 manualScale，但自動縮到塞得下）。
- `dcScale`（1828）：Discord 320 畫布**永遠**用 `fit = maxFitScale(w,h,320)` 封頂；非 manual 模式下否則沿用 `effScale`。
- `scaleAndCenter`（1195）：nearest-neighbor 整數放大（`(y/scale)|0`），floor 置中（`ox = floor((size-fw)/2)`，奇數餘量偏左上），透明底。

### 1.4 六種輸出（input → bytes）

| 輸出 | 函式 | 鏈路 | 預設 / 上限 |
|---|---|---|---|
| **512APNG**（Signal） | `buildApng512`（2338） | activeFrames → quantizeFrames(apngMaxColors) → *(fit3s)* fit3sDelays → scaleAndCenter(512) → encodeApng | 256 色、fit3s off、loops 0（∞）；warn **>300KB**（2366） |
| **320APNG**（Discord） | `buildApng320`（2345） | 同上**但無 fit3s** → scaleAndCenter(320) → encodeApng(320) | 共用 apngMaxColors/apngLoops；warn **>512KB**（2390） |
| **TGS**（Telegram） | `buildLottie`（1557）+ `encodeTgs`（1674） | 像素 → vector rects → Lottie JSON → gzip | 上限 **64KB**（比 **gzip 後**大小，2262）；debounce 250ms |
| **WEBP**（WhatsApp） | `buildWebp`（2401）+ `muxAnimatedWebp`（1446） | scaleAndCenter(512) → diffBox 差分子幀 → toBlob → mux | lossless 預設；warn **>100KB**（2451） |
| **PNG frame** | frame cell（2552） | currentFrame → scaleAndCenter(512) → encodePng | 完整 512 padded PNG |
| **PNG pad** | pad cell（2564） | 同上 → boundingBox(512 空間) → cropRgba | 裁切到內容；空幀 → skip |
| Lottie JSON | `expJson`（2579） | 同 TGS 的**未壓縮** JSON | — |

**TGS 細節（GIF → 向量）**：

- 像素 → rects：`frameRects`（1506）掃每列的同色不透明連續段；`rectMerge=true` 再垂直合併成最大矩形。
- opt 旗標（新 GIF item 全部預設 `true`，2777）：`removeDup`（相鄰同簽章幀合併，延遲累加）、`sameColorMerge`（同色 rect 併一個 shape group）、`removeEmpty`（空幀不發 layer）、`floatPrecision`（座標四捨五入到 1 位小數）、`removeDefaults`（剝除 Lottie 樣板欄位）、`precompReuse`（`true` 用單一 precomp 被兩層共享；`false` 建兩份重複 precomp，較大但相容性較好）。
- **0.4px offset 雙層技巧**（`offsetEnabled`/`offsetPx`，預設 `true`/`0.4`）：位置 `[256+dx, 256+dx, 0]`（1638）—— **同一個 dx 同時套在 x 和 y**，是對角線位移而非分軸獨立。用途：填補相鄰向量矩形被 Lottie renderer anti-alias 後的髮絲縫。
- 時間軸以 60fps 為基（`TGS_FR = 60`，1551）。

**WebP 細節**：

- 差分子幀：首幀全畫布，之後只編 `diffBox` 變動框（2421）。
- **重複幀去除**（2422）：無差異框時，把延遲併入前一編碼幀，不另發幀（libwebp 式 dedup）。
- **lossy 全幀覆蓋**（2423）：`!lossless` 時強制全畫布，避免差分邊界接縫。
- **偶數座標對齊**（2424）：WebP ANMF 的 x/y 以 2px 為單位，奇數座標 `x--; w++`。
- 編碼 `toBlob("image/webp", lossless ? 1 : min(99,quality)/100)`（2428）。**✅ 已實測**：quality `1` 在 Chromium 產出 **VP8L** chunk，解碼 round-trip 逐 byte 完全一致（maxChannelDiff = 0）—— 確為真無損。lossy 上限 99%，不到 100%。〔註：此為瀏覽器相依行為；在 Chromium 系成立。〕

### 1.5 `encodeApng` 內部（1413–1440）

- `buildPalette`（1353）：掃全幀顏色，`if (seen.size > 256) return null` —— **≤256 色用索引色（type 3）**，超過退回 RGBA（type 6）。
- palette 排序按 alpha 升冪（1361），把半透明色排前面 → `tRNS` 只需覆蓋到最後一個非不透明索引，縮短長度。
- 差分幀（1428）：`box = diffBox(prev, cur) || {x:0,y:0,w:1,h:1}` —— 像素相同時退回 1×1 dummy 區域，**仍發完整 fcTL/fdAT**（無跨幀 dedup，與 WebP 不同，見問題 §4-3）。
- `filterScanlines`（1288）：oxipng 式自適應濾波，每列試 5 種 filter 取最低分。
- chunk 順序：`SIG, IHDR, PLTE, [tRNS], acTL, fcTL/IDAT, [fcTL/fdAT]…, IEND`；`acTL.num_plays = loops || 0`（**0 = 無限**）。

### 1.6 排程與快取

- `scheduleApng`/`scheduleDc`/`scheduleWebp`：**400ms** debounce；`scheduleTgs`：**250ms**。
- 共同模式：清 timer → **同步清空 `it.lastX` 快取**（設定一改就失效）→ 遞增 token → async build 後檢查 `token !== it.xToken` 丟棄過期結果。
- `refreshStatic`（2158）是唯一同時呼叫四個排程器的地方；多數設定變更經 `upd = () => refreshStatic(it)` 觸發。例外：offset slider 與 opt.* checkbox 只呼 `scheduleTgs`；`fit3s` 只呼 `scheduleApng`（省去無關重建）。
- 匯出走快取優先：`it.lastApng || await buildApng512(it)`（2540）。

### 1.7 設定層

- 全域 `G`（1727）：`scaleMode:"manual", manualScale:12, fpsMode:"auto", fps:30, apngLoops:0, apngMaxColors:256, fit3s:false, webpLossless:true, webpQuality:90, webpLoops:0`，cells 預設 `{apng, discord, tgs, webp: true; frame, pad, lottie: false}`。
- Per-file seeding（`addFiles`，2760）：多數欄位載入時複製當下 `G` 值；`rangeStart/End/skip/loop/frameIndex` 固定重置；`dcScaleMode:"fit"/dcManual:6` 與 opt.* 為**硬編碼字面值**（無對應 `G` 全域，見問題 §4-2）。
- `applyGlobal`（2918）：改全域控制會**立即覆寫所有已開面板**的對應值。
- 持久化：`localStorage` key `stickerforge_studio_v1`。

---

## 2. TGS 管線

### 2.1 載入（`_decompressTgs` 3568、`_loadLottieLib` 3545）

- gunzip：`DecompressionStream('gzip')` → 串接 → UTF-8 decode → `JSON.parse`。
- lottie-web **延遲載入**：注入 `<script src="https://cdnjs.cloudflare.com/ajax/libs/bodymovin/5.12.2/lottie_canvas.min.js">`，快取於 `window._lottieLib`（跨上傳、跨 Tool Box 共用）。首次需網路，失敗擲明確錯誤。

### 2.2 幀渲染（`_renderLottieFrames` 3584、`_normalizeRenderedTgsFrames` 3646）

- canvas renderer，`dpr: 1`、`preserveAspectRatio: 'xMidYMid meet'`。
- **逐整數幀取樣** `[ip, op)`（半開）：`goToAndStop(startFrame+f, true)` → `getImageData` 複製。
- `fps = anim.frameRate || 30`（3611，預設 30）；`delayMs = max(16, round(1000/fps))`（**16ms 下限**，與 resample 的 10ms 不同）。
- 正規化：前後**全透明幀修剪**（`boundingBox` 判空），更新 `frames`/`totalFrames`；計算 `sourceBox`（各幀 bbox 聯集）。

### 2.3 Per-item 狀態（`it`，4307）

`settings` 由 `G.tgsSettings.<fmt>` seeding：`frameIndex:null`（null → 跟隨全域 FRAME MODE）、每格 `{loops, autoFit, fps, maxColors}`（apng/discord/webp）、webp 另有 `lossless/quality`。快取 `lastApng/lastDc/lastWebp/lastFramePng/lastPadPng`；`timers`/`tokens` 每格一份。**frame/pad 無 autoFit/fps/maxColors** —— 它們是單張靜態 PNG，由全域 FRAME MODE + per-item slider 決定。

### 2.4 AUTO-FIT 階梯（`_encodeUnderBudget` 4072）—— 核心

目標：壓到 platform byte budget 以下。順序：**先降幀，再減色**。

- **Stage 1（fps）**：`ladder = [srcFps, ...TGS_FPS_LADDER.filter(f < srcFps)]`，即 `[native, 60, 30, 20, 10]` 過濾去重，native 永遠先試。每階 resample → encode(256色) → 第一個 `<= budget` 立即回傳。
- **Stage 2（色/質）**：取 stage-1 中**位元組最小**的 fps（不一定最低 fps），在該 fps 下走 `TGS_COLOR_LADDER = [128, 64]`（apng/discord/lossless-webp）或 `TGS_QUALITY_LADDER = [75, 60]`（lossy-webp）。第一個 `<= budget` 回傳。
- **fallback**：全都不過，回傳**見過的最小候選**，`underBudget: false`，面板保留 ⚠。
- 判定用 `<= budget`（非死碼舊版的「smaller-wins」）。每階間 `yieldToUi()` 讓面板重繪，`onRung` 即時把 size cell 文字改成 `FPS 30…` / `128 COLORS…` / `Q 60…`。

**budget 來源**：`STATS_FORMAT_META`（唯一預算表）—— apng 300KB、discord 512KB、webp 100KB、frame/pad 0（無上限）。

### 2.5 各輸出（`_buildTgsOutput` 4111）

- **apng/discord**：`quantizeFrames(src, colors)` → `encodeApng`。discord 特別之處：**先降尺寸再減色**（`_resizeTgsFrames` 在 quantize 前，因為平滑縮放會重新引入顏色），並以 `resizedByFps` Map 快取各 fps 的縮放結果。discord 尺寸 `_tgs320Size()` 64–512、預設 320。
- **webp**：`_buildTgsWebp(quantizeFrames(f,colors), lossless, quality, loops)`。lossy 的 stage2 只走 quality，`colors` 恆 256 → `quantizeFrames` 為 no-op（見問題 §4-6）。
- **frame/pad**：`_tgsSelectedFrame` → `encodePng`；pad 先 `boundingBox` 裁切，空幀回 `{skipped:true}`。
- 幀選取 `_tgsFrameIndex`（3776）：`frameIndex==null` 時用全域 FRAME MODE（first/last/mid/no.）；slider 一動即設明確覆蓋值。

### 2.6 排程與快取

- `scheduleTgsBuild`（4245）：**400ms** debounce、token 在 timer 內鑄造、**同步清空快取**（匯出永不吐舊 bytes）。
- `_primeTgsStats`（4271）：**上傳即建置所有啟用輸出**（面板建好後 80ms 觸發），每格間 yield 讓面板漸進顯示 BUILDING…。→ 匯出變成 cache read。
- `_collectTgsEntries`（4210）：cache-first —— 有暖快取直接用不重編（實測匯出 22ms）；冷快取才建。
- 新開啟某格（Settings 勾選）以 `delay=0` 立即建，不等 400ms。

### 2.7 全域設定（`G.tgsSettings` 1739、`installStableTgsController` 4935）

- 預設每格 `autoFit: true, fps: null, maxColors: 256`。
- **重要**：全域設定面板**沒有** autoFit/fps/maxColors 的控制項 —— 這三者只在 per-item 面板設定，全域值僅作**新上傳項目的 seed**。
- 持久化 `saveSettings`：localStorage **有**含完整 `tgsSettings`（見問題 §4-7 對照 Export/Import）。

---

## 3. 共用元件語意

- **`quantizeFrames(frames, maxColors)`**（2276）：`maxColors >= 256` 或 `0/null` 直接回原陣列（by reference）。否則取最高頻的前 N 色，其餘映射到最近色，距離為**加權平方距離**，alpha 通道**權重 3×**（`3*(Δa)²`）。透明像素恆保 `0,0,0,0`。永不 mutate。
- **`resampleFramesToFps(frames, srcFps, targetFps)`**（2325）：`targetFps >= srcFps` 或單幀 → 原樣回傳（**永不升幀**，rgba 共享 by reference）。`outCount = round(len * target/src)`，最近幀取樣，`delayMs = max(10, round(1000/target))`。
- **`fit3sDelays(frames)`**（2316）：總時長 `> 3000ms` 才縮，`k = 3000/total`，每幀延遲 floor 10ms。僅 Signal/512APNG 用。

---

## 4. 問題與建議（經審核，依嚴重度排序）

> 兩份子代理 SUSPECT 清單去重合併，逐項核對原始碼。標 ✅ 者我另外實測／覆核過。

### 🔴 高：影響正確性或使用者可感知

**4-1. Export/Import 設定會靜默丟失全部 TGS 設定** ✅ 已核對（3465–3504）
localStorage 自動存檔（`saveSettings`/`loadSettings`）**有**完整 round-trip `tgsSettings`；但使用者手動的 **Export/Import JSON 完全不含任何 TGS 狀態**：
- Export blob（3471）**既無 `tgsSettings` 也無 `tgsCells`**。
- Import（3491）有一條 `if (s.tgsCells)` 分支，但因 export 從不寫 `tgsCells`，**這條分支是死的**；`s.tgsSettings` 則連讀都沒讀。

> 子代理 B 原稱「只有 tgsCells 存活」—— 覆核後更正：**tgsCells 也不存活**，那條 import 分支從不觸發。
> **後果**：使用者匯出設定備份／分享再匯入，會靜默丟掉全部 TGS 調校（AUTO-FIT、每格 FPS/MAX COLORS、discord 尺寸、webp 品質、frame/pad 模式、5 個資料夾名）。
> **建議**：export blob 補上 `tgsSettings`/`tgsCells`；import 比照 `loadSettings` 加 `if (s.tgsSettings){...}` 的 per-subkey merge。

**4-2. 「還原預設」不會重置已載入的 TGS 項目** ✅ 已核對（3435–3450 vs 3411–3428）
GIF 側 reset 迴圈直接改寫每個已載入 item 的 `it.settings`；TGS 側只呼 `_refreshTgsPanel`（僅切換 cell 顯隱），**不動已開面板的 per-item 設定**。
> **後果**：兩條管線行為不一致 —— 「還原預設」對 GIF 立即生效、對已開的 TGS 面板無效（只影響未來上傳）。
> **建議**：TGS reset 也加 per-item 迴圈重置 `it.settings` 並重排建置，或明確在 UI 說明其只影響新上傳。

**4-3. `pad` 空幀跳過時未清快取，可能吐出過期裁切** ⚠️ 未完全追完全部呼叫點
`_buildTgsOutput` 對空選幀回 `{skipped:true}`，`_applyTgsBuild` 早退、**不動 `it.lastPadPng`**；快取清空只發生在 `scheduleTgsBuild`。若先前非空幀留下 `lastPadPng`，之後 slider 切到空幀，`_collectTgsEntries` 的 cache-first 可能沿用**上一幀的裁切 PNG**。
> **建議**：skip 分支明確 `it.lastPadPng = null` + `_clearOutputSize`。此項我未逐一追完呼叫點，建議實測確認再修。

### 🟡 中：一致性 / 潛在維護風險

**4-4. 512APNG 與 320APNG 共用 `apngLoops`/`apngMaxColors`**（GIF 側，2338/2345）
GIF 管線沒有 Discord 專屬的迴圈數／色數，但 TGS 管線**有**獨立的 `apng512` vs `discord`（1740）。同 app 兩側不對稱。

**4-5. GIF 側 Discord 縮放無全域預設**（2775）
`dcScaleMode:"fit"/dcManual:6` 硬編碼、無 `G` 對應，全域面板無法預設。

**4-6. lossy WebP 的 AUTO-FIT 靜默停用 MAX COLORS**（TGS，4160–4166）
lossy webp 的 ladder 只走 quality、`colors` 恆 256 → `quantizeFrames` no-op。設計上正確（調色盤對 VP8 無意義），但 AUTO-FIT 開啟時 MAX COLORS 輸入完全失效；手動模式則仍會 quantize。屬「刻意但隱晦」，建議 UI 註記。

**4-7. APNG 無重複幀 dedup，WebP 有**（1428 vs 2422）
`encodeApng` 對相同幀退回 1×1 dummy 仍發完整 fcTL/fdAT；WebP 直接併延遲。→ 近靜態 GIF 的 APNG 輸出比 WebP 多帶每幀 overhead。

**4-8. `collectEntries`（2598）與 `_collectGifFormatEntries`（4736）近乎重複**
同一套 per-format 建置/快取邏輯維護在兩處，且錯誤處理不一致（前者 webp 有 try/catch 吞例外，後者沒有）—— 未來易漂移。

### 🟢 低：死碼 / vestigial

- **4-9.** `skip` 輸入 UI 夾 0–30，但 `activeFrames` 內部支援 0–60（2698 vs 1881）—— 一半範圍面板無法輸入。
- **4-10.** `saveSettings` 持久化以第二個 late-bound listener 陣列（3506）附掛，非集中在 `applyGlobal` —— 新增全域控制若忘了加進陣列會靜默不存檔。
- **4-11.** `G.filePrefix`/`fileSuffix`（1728）GIF 管線未使用，也不在持久化清單 —— 疑似 vestigial。
- **4-12.** `_normalizeRenderedTgsFrames` 全透明分支未更新 `totalFrames`（3654）—— frame slider max / 標籤 / 時長計算會與實際 1 幀不符（極端邊界）。
- **4-13.** `normalized`/`normalizeScale`/`frameTransform`（3664）恆為 `false`/`1`/固定字串，無人讀取 —— 未實作功能的殘留 scaffold，命名誤導。
- **4-14.** `_renderLottieFrames` 內空 `if(){}` 區塊（3618）—— 剝除後留下的空 guard。
- **4-15.** `durationWarn`（Signal >3s）在面板建構時算一次、不隨 FPS 變更重算（3888）。因時長是來源屬性、resample 不改總時長，實務上無誤，但與其他 warn 皆 build-driven 的模式不一致。

> **子代理 B 的 SUSPECT 5（WebP lossless 可能非真無損）已推翻** —— 我實測 `toBlob(...,1)` 產出 VP8L + 逐 byte round-trip 一致，工具的無損聲明成立。

---

## 5. 平台支援缺口

> 每項數字附來源，標明一手（官方頁直接取得）或二手（第三方／論壇轉述）。查不到者明說，不杜撰。

### 5.1 已列在規格表、但工具未真正滿足

**LINE**（一手：[creator.line.me](https://creator.line.me/en/guideline/animationsticker/)）
- 官方：APNG、**≤320×270 且任一邊 ≥270px**、**≤1MB/張**、**5–20 幀**、**1–4 loops 且總長 ≤4s**。
- 規格表列的「≤320×270」正確，但**無專屬編碼路徑** —— 註記寫「USES GIF → APNG」，退回通用 APNG。若使用者拿 Discord 的 **320×320** 輸出來充當，會**違反 270 高度上限**、也不強制幀數/loop/1MB。
- **缺口**：需 `320×270 · LINE` 專屬 preset —— max(w,h)=320 且 min(w,h)≥270 的畫布夾制、5–20 幀 resample、1–4 loop 欄位（驗證 ≤4s）、1MB gate。

**iMessage**（一手 canvas：[Xcode Asset Catalog 參考](https://developer.apple.com/library/archive/documentation/Xcode/Reference/xcode_ref-Asset_Catalog_Format/StickerPack.html)；size 二手：[Apple 論壇 50394](https://developer.apple.com/forums/thread/50394) Apple 員工回覆）
- 修正規格表：不是「300–618 連續範圍」，而是**三個固定 preset**：small 300×300 / regular 408×408 / large 618×618（@3x）。
- **≤500KB/檔**（per-file，非 per-frame，論壇 Apple 員工確認，Xcode build 與 runtime 皆檢查）。
- 格式列（PNG/APNG/GIF/JPEG）—— HIG 頁 JS 渲染取不到本文，僅 WebSearch 摘要佐證，標**二手**。
- **缺口**：改成三個固定尺寸 preset + 500KB gate。

### 5.2 完全未涵蓋，值得補（規格已一手驗證）

| 平台 | 規格（來源） | 可重用 |
|---|---|---|
| **Steam Points Shop** | 150×150 APNG **<300KB** + 靜態 PNG（一手：[partner.steamgames.com](https://partner.steamgames.com/doc/marketing/pointsshopitems)） | APNG 編碼器，改畫布/預算 |
| **Slack 動態 emoji** | 方形 **<128KB**、GIF ≤50 幀（一手：[slack.com](https://slack.com/help/articles/206870177)；128×128 為第三方慣例、官方本文未明列） | 原生 GIF 路徑 |
| **MS Teams** | JPEG/PNG/GIF **<256KB**、方形（一手：[support.microsoft.com](https://support.microsoft.com/en-us/office/use-custom-emoji-in-microsoft-teams-84feb1c4-6d2b-4ecd-8e55-a93c828fc53a)） | 原生 GIF 路徑 |
| **Mastodon** | 預設 **256KB**（一手 PR：[mastodon#18788](https://github.com/mastodon/mastodon/pull/18788)）；聯邦制、各站可改；~128×128 為慣例 | GIF/PNG 路徑 |

### 5.3 二手 / 未能驗證（先不動）

- **Twitch 動態 emote**：112×112 GIF ≤1MB（多方第三方一致，但 help.twitch.tv JS 渲染取不到本文）—— **二手**。若補，用原生 GIF 路徑。
- **WeChat**：GIF 240×240 ≤500KB（官方 PDF 為圖片式無法擷取文字，第三方 blog 佐證；另有來源稱 60KB，**衝突未解**）—— **二手**。特別之處：要 **GIF 直出**。
- **KakaoTalk**：疑似 320×270 APNG（與 LINE 雷同，可能是來源混淆）—— **未能一手驗證**。若確認可直接重用 LINE 路徑。
- **Viber / Matrix**：Viber 無公開資產規格（app 內製作）；Matrix（MSC2545）不強制格式/大小，現有 PNG/APNG/WebP 已是有效 sticker —— **無可施力點**。

### 5.4 建議補齊順序

1. **LINE（修，非加）** —— 規格已列，補專屬 `320×270` preset 關掉現存的合規缺口；市場大（JP/TH/ID）。
2. **Twitch** —— `112×112 GIF ≤1MB`，直接建在原生 pixel-art GIF 路徑上（免 APNG/WebP），受眾高度重疊。〔規格待一手確認〕
3. **Slack** —— `GIF ≤128KB, ≤50 幀`，與 Twitch 同模式，Twitch 路徑做完幾乎零增量。
4. **Steam** —— `150×150 APNG ≤300KB`，clone 現有 APNG 編碼器；規格是剩餘平台中最明確的（Steamworks 一手）。

> MS Teams / Mastodon 一旦 GIF-reuse 路徑成形即幾乎免費，屬 fast-follow。

---

## 附錄：審核方法

- GIF/TGS 事實由兩個 Explore 子代理平行萃取，行號抽驗核對原始碼（`delayMs: raw < 20`、`fit3sDelays` 3000/10、`256+dx`、`TGS_*_LADDER`、`resampleFramesToFps` 等錨點全數命中）。
- **WebP 無損**：瀏覽器實測 `toBlob('image/webp',1)` → 解析 RIFF chunk 得 `VP8L`，解碼 round-trip `maxChannelDiff=0`。
- **Export/Import 掉 TGS**：直接讀 3465–3504 確認 export blob 鍵與 import 分支。
- 平台規格由第三個子代理研究，一手來源以官方 URL 直接取得者標「一手」，論壇/第三方/摘要標「二手」，取不到本文者明列「未能驗證」。
