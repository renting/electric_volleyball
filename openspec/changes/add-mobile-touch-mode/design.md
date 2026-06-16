## Context

`electric_volleyball.html` 是單一 HTML 檔的 Phaser 3 遊戲，畫布固定邏輯尺寸 `W=900, H=500`，使用 `Scale.FIT + CENTER_BOTH` 自動等比縮放。輸入完全依賴鍵盤：`GameScene._controlHuman(p, left, right, up, hit)` 直接讀取 Phaser Key 物件的 `.isDown`，並依「球距離 + 是否按住單一左右方向鍵」分流為強殺／撲球／揮空三種結果（見 `dive-action`）。`MenuScene` 以數字鍵 1/2/3 選模式，並有一個 `pointerdown` → 1P 普通的捷徑。

手機沒有實體鍵盤，因此遊戲在手機上不可操作。本設計在「不更動任何玩法數值、物理與 AI 行為」的前提下，疊加觸控輸入、行動版面與橫向處理。

## Goals / Non-Goals

**Goals:**
- 手機橫向（landscape）即可遊玩單人模式（普通／困難）。
- 觸控操作沿用既有控制路徑與三向分流，手感與桌面版一致；不重寫物理。
- 觸控面板僅在觸控裝置顯示，桌面鍵盤體驗零變動。
- 直向時以提示引導使用者轉橫向，不硬性鎖定方向（相容 iOS Safari）。
- 維持單檔架構與 Phaser 3 CDN，不新增建置工具或外部相依。

**Non-Goals:**
- 手機 2P 同機對戰（體驗差，列為未來另案）。
- 原生 App、PWA 安裝、離線快取。
- 觸覺回饋（vibration）、自訂按鈕配置等進階體驗。
- 變更分數、勝負、撲球數值或 AI 難度參數。

## Decisions

### 決策 1：輸入抽象層——統一「輸入狀態物件」取代直接讀 Key
目前 `_controlHuman` 收的是 Phaser Key 物件（含 `.isDown`）。改為收一個 plain object，例如 `{left:{isDown}, right:{isDown}, up:{isDown}, hit:{isDown}}`。每幀由兩個來源更新此物件：鍵盤（讀 `this.kbd`）與觸控（讀虛擬按鈕按下狀態）。`_controlHuman` 內部邏輯與三向分流**一行都不改**。

- **為何**：既有 1P 模式已把 `Space`/`Z` 合成 `{isDown:...}` 傳入，證明此抽象可行且風險低；只要把觸控狀態 OR 進同一物件即可。
- **替代方案**：在 `_controlHuman` 內直接判斷觸控 → 會讓控制邏輯分叉、難維護，且違反「分流邏輯僅實作一份」既有規格，否決。

### 決策 2：觸控面板用 HTML/CSS overlay，不畫在 canvas 內
虛擬按鈕以絕對定位的 `<div>`/`<button>` 疊在 `#wrap` 上層（左下方向群組、右下動作群組），透過 `touchstart`/`touchend`/`pointerdown`/`pointerup`/`touchcancel` 維護各鈕的 `pressed` 布林。

- **為何**：HTML 元素天生支援多點觸控與無障礙焦點，CSS 定位/RWD 成熟；不需在 Phaser 內自製碰撞判定與 hit-test，且不受 canvas 縮放影響、點擊區穩定。
- **替代方案 A**：在 Phaser 內以互動 Sprite 當按鈕 → 需處理多點觸控與座標換算，複雜度高。否決。
- **替代方案 B**：全螢幕分區點擊（左半移動／右半動作）→ 分區式新手不易發現，否決。
- **版面演進**：初版採「左下方向鈕」，迭代後改為**左側虛擬搖桿**（見決策 3），手感較佳。

### 決策 3：移動改用虛擬搖桿（joystick），類比偏移映射為布林
左側以圓形底座 + 可拖曳旋鈕構成搖桿：旋鈕相對中心的水平偏移過死區門檻 → `left`/`right`，向上偏移過門檻 → `up`（跳躍）；右下保留扣殺/撲球鈕。搖桿以 Pointer Events + `setPointerCapture` 只追蹤啟動它的那根手指，動作鈕為獨立元素，兩者可同時操作。所有觸控狀態仍映射為布林 `isDown`，`_controlHuman` 與玩法數值不變。

- **為何**：搖桿比固定方向鈕更貼近手機遊戲操作直覺，且向上推即可跳躍、左右連續控制更靈活；類比→布林（門檻）映射可完全沿用既有控制路徑，零玩法變更。
- **替代方案 A**：保留三顆方向鈕 → 連續移動與微調手感較差，使用者要求改搖桿，否決。
- **替代方案 B**：類比速度（依偏移量比例調整移動速度）→ 需改 `_controlHuman` 速度邏輯，違反「玩法數值不變」，否決。
- **取捨**：需處理 `pointercancel`/放開時旋鈕歸位與輸入清空，避免卡鍵；上推跳躍取代獨立跳躍鈕。

### 決策 7：視覺強化——球放大與扣殺彗星殘影
球半徑 17→22（同步套用於碰撞/網子/落地/繪製，維持物理一致）；以 `ball.trail`（每物理幀記錄、上限約 16 筆）繪製漸層淡出殘影，`_tryPowerHit` 觸發 `ball.smash` 計時器使扣殺殘影最明顯並呈黃→橘彗星漸層，高速飛行時顯示較淡拖尾、低速不顯示。

- **為何**：使用者要求球更大、扣殺更明顯；殘影為純繪製疊加，不介入物理。
- **取捨**：球變大略增加擊球判定範圍（`_inHitRange` 含 `ball.r`），屬可接受的手感微調；殘影繪製於球體之下避免遮蓋。

### 決策 4：橫向閘門以 `matchMedia('(orientation: portrait)')` 偵測，不用 `orientation.lock`
偵測直向時，顯示覆蓋整頁的「請將手機轉為橫向」HTML 提示層，並將輸入狀態物件全部視為未按下（暫停輸入）；轉為橫向自動隱藏提示。

- **為何**：`screen.orientation.lock('landscape')` 在 iOS Safari 不支援、且多數瀏覽器需全螢幕或安裝為 PWA 才生效，無法當作可靠主路徑。使用者問答已選「顯示提示」方案。
- **替代方案**：先試 lock 再退回提示 → 增加複雜度與不一致行為，使用者已選純提示，否決。

### 決策 5：觸控裝置偵測——決定是否顯示面板與隱藏 2P
以 `matchMedia('(pointer: coarse)')`（輔以 `'ontouchstart' in window`）判定觸控裝置。觸控裝置：顯示觸控面板、`MenuScene` 僅顯示單人（普通／困難）兩個可點選項、隱藏鍵盤說明文字。非觸控桌面：完全維持現狀。

- **為何**：`pointer: coarse` 比 UA sniffing 穩健，且能涵蓋平板。
- **取捨**：觸控筆電（同時有鍵盤與觸控）會被視為觸控裝置而顯示面板——可接受，因面板不影響鍵盤操作（兩來源 OR 合併）。

### 決策 6：行動版面（RWD / viewport）
`<meta viewport>` 加上 `viewport-fit=cover, user-scalable=no`。觸控裝置橫向時：隱藏 `<h1>` 標題與 `#hint` 鍵盤說明，讓 `#wrap` 以 `100vw/100vh` 範圍內最大化（維持 9:5 比例由 `Scale.FIT` 處理）。桌面版 CSS 維持 `width:min(96vw,920px)` 等既有規則不變。觸控面板定位使用 `env(safe-area-inset-*)` 避開瀏海/圓角。

## Risks / Trade-offs

- [iOS Safari 不支援 orientation.lock 與全螢幕受限] → 改以「請橫向」提示 + 輸入暫停處理，不依賴鎖定 API。
- [瀏覽器手勢（捲動、雙擊縮放、長按選單）干擾觸控操作] → 按鈕與 `#wrap` 設 `touch-action: none`、在 `touchstart` 呼叫 `preventDefault()`、CSS `user-select:none`、viewport `user-scalable=no`。
- [手指滑出按鈕造成 `pressed` 卡住] → 監聽 `touchend`/`touchcancel`/`pointerup` 與 `pointerleave`，並在橫向閘門啟動時強制清空所有 `pressed`。
- [`MenuScene` 既有 `pointerdown` → 直接開始 1P，與觸控選單按鈕衝突] → 觸控裝置上以可點擊的模式按鈕取代「點任意處開始」，避免誤觸；桌面點擊捷徑可保留。
- [觸控筆電顯示面板但使用者只想用鍵盤] → 兩輸入來源 OR 合併、互不干擾；面板僅佔畫面邊角，影響可接受。
- [回歸風險：輸入抽象重構動到 GameScene 控制路徑] → 重構限定在「來源 → 統一狀態物件」這層，`_controlHuman` 與物理不動；以桌面鍵盤對拍既有行為驗證無回歸。
