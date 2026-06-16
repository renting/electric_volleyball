## ADDED Requirements

### Requirement: 觸控操作面板版面
系統 SHALL 在觸控裝置上於 `#wrap` 上層以 HTML/CSS overlay 顯示虛擬操作面板，採「左下方向群組 + 右下動作群組」配置：

- 左下群組：左移鈕（←）、右移鈕（→）、跳躍鈕（↑）。
- 右下動作群組：跳躍鈕（與左下跳躍擇一實作，建議右側保留扣殺）、扣殺/撲球鈕（hit）。
- 面板元素 MUST 使用絕對定位疊在畫布上，且以 `env(safe-area-inset-*)` 避開瀏海與圓角。

面板 SHALL 僅在偵測為觸控裝置時顯示（見 `mobile-layout` 的觸控裝置偵測），非觸控桌面環境 MUST NOT 顯示。

#### Scenario: 觸控裝置顯示面板
- **WHEN** 以觸控裝置（`pointer: coarse`）開啟遊戲並進入橫向遊戲畫面
- **THEN** 畫面左下出現方向鈕群組、右下出現動作鈕群組

#### Scenario: 桌面不顯示面板
- **WHEN** 以非觸控桌面瀏覽器開啟遊戲
- **THEN** 不顯示任何虛擬按鈕，鍵盤操作與外觀維持原狀

---

### Requirement: 觸控狀態映射為統一輸入狀態
每個虛擬按鈕 SHALL 獨立維護 `pressed` 布林，並由 `touchstart`/`pointerdown` 設為 `true`、由 `touchend`/`touchcancel`/`pointerup`/`pointerleave` 設為 `false`。系統 SHALL 每幀將觸控按鈕的 `pressed` 映射為與鍵盤等價的統一輸入狀態（`left`/`right`/`up`/`hit`），與鍵盤來源以邏輯 OR 合併後傳入既有 `_controlHuman` 控制路徑。

觸控輸入 MUST NOT 改變既有三向分流（強殺／撲球／揮空）與任何玩法數值。

#### Scenario: 按住右移鈕
- **WHEN** 玩家按住虛擬「→」鈕
- **THEN** 統一輸入狀態的 `right.isDown` 為 `true`，玩家角色每幀右移 5.4（與鍵盤行為一致）

#### Scenario: 放開或滑出按鈕
- **WHEN** 玩家放開按鈕，或手指滑出按鈕邊界觸發 `touchcancel`/`pointerleave`
- **THEN** 對應按鈕 `pressed` 設為 `false`，不會發生卡鍵

---

### Requirement: 多點觸控同時操作
系統 SHALL 支援同時按住方向鈕與動作鈕，使「移動中扣殺」與「方向 + 扣殺鈕」的撲球可被觸發。左右移動鈕同時被按住時 MUST 視為無方向（`dirHeld = 0`，與既有鍵盤規格一致）。

#### Scenario: 按住方向加扣殺鈕撲球
- **WHEN** `play` 狀態中球不在扣殺判定範圍內，玩家在地面同時按住虛擬「→」與「扣殺」鈕
- **THEN** 觸發向右撲球（依 `dive-action` 觸發條件），與鍵盤「→ + Space」結果一致

#### Scenario: 同時按住左右移動鈕
- **WHEN** 玩家同時按住虛擬「←」與「→」鈕並按扣殺鈕
- **THEN** 方向視為無（`dirHeld = 0`），不觸發撲球（與既有規格一致）

---

### Requirement: 阻止瀏覽器預設觸控手勢
觸控面板按鈕與 `#wrap` SHALL 設定 `touch-action: none` 並在 `touchstart` 呼叫 `event.preventDefault()`，且以 CSS `user-select: none` 禁止文字選取，避免捲動、雙擊縮放、長按選單干擾操作。

#### Scenario: 操作時不觸發頁面捲動或縮放
- **WHEN** 玩家在遊戲中連續快速點按方向鈕與動作鈕
- **THEN** 頁面不捲動、不縮放、不彈出長按選單，輸入正常響應
