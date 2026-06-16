## ADDED Requirements

### Requirement: 觸控操作面板版面
系統 SHALL 在觸控裝置上於 `#wrap` 上層以 HTML/CSS overlay 顯示虛擬操作面板，採「左側虛擬搖桿（joystick）+ 右下動作鈕」配置：

- 左側：虛擬搖桿（圓形底座 + 可拖曳旋鈕），負責左右移動與跳躍。
- 右下：扣殺/撲球鈕（hit）。
- 面板元素 MUST 使用絕對定位疊在畫布上，且以 `env(safe-area-inset-*)` 避開瀏海與圓角。

面板 SHALL 僅在偵測為觸控裝置時顯示（見 `mobile-layout` 的觸控裝置偵測），非觸控桌面環境 MUST NOT 顯示。

#### Scenario: 觸控裝置顯示面板
- **WHEN** 以觸控裝置（`pointer: coarse`）開啟遊戲並進入橫向遊戲畫面
- **THEN** 畫面左側出現虛擬搖桿、右下出現扣殺鈕

#### Scenario: 桌面不顯示面板
- **WHEN** 以非觸控桌面瀏覽器開啟遊戲
- **THEN** 不顯示搖桿與按鈕，鍵盤操作與外觀維持原狀

---

### Requirement: 搖桿與動作鈕映射為統一輸入狀態
虛擬搖桿 SHALL 以類比偏移量映射為與鍵盤等價的布林輸入：旋鈕相對底座中心的水平偏移超過死區（deadzone）門檻時對應 `left`/`right`，向上偏移超過門檻時對應 `up`（跳躍）。動作鈕 SHALL 獨立維護 `pressed` 並映射為 `hit`。系統 SHALL 每幀將觸控輸入與鍵盤來源以邏輯 OR 合併後傳入既有 `_controlHuman` 控制路徑。

死區門檻內 MUST 視為未輸入，避免輕觸即移動。觸控輸入 MUST NOT 改變既有三向分流（強殺／撲球／揮空）與任何玩法數值。

#### Scenario: 搖桿右推移動
- **WHEN** 玩家將搖桿向右推超過死區門檻
- **THEN** 統一輸入狀態的 `right.isDown` 為 `true`，玩家角色每幀右移 5.4（與鍵盤行為一致）

#### Scenario: 搖桿上推跳躍
- **WHEN** 玩家在地面將搖桿向上推超過死區門檻
- **THEN** 統一輸入狀態的 `up.isDown` 為 `true`，觸發跳躍（與鍵盤 ↑ 行為一致）

#### Scenario: 死區內不觸發移動
- **WHEN** 玩家僅輕微觸碰搖桿、偏移量未超過死區門檻
- **THEN** `left`/`right`/`up` 皆為 `false`，角色不移動、不跳躍

---

### Requirement: 多點觸控與搖桿釋放
系統 SHALL 支援同時操作搖桿與動作鈕，使「移動中扣殺」與「搖桿方向 + 扣殺鈕」的撲球可被觸發。搖桿 SHALL 只追蹤啟動它的那一根手指（pointer id），動作鈕為獨立元素。手指放開或取消（`pointerup`/`pointercancel`）時搖桿 SHALL 歸位並清空 `left`/`right`/`up`，動作鈕釋放時 SHALL 清空 `hit`，皆不得卡住。搖桿左右偏移同時跨越兩側為不可能狀態；當水平偏移未過門檻即視為無方向（`dirHeld = 0`，與既有鍵盤規格一致）。

#### Scenario: 搖桿加扣殺鈕撲球
- **WHEN** `play` 狀態中球不在扣殺判定範圍內，玩家在地面將搖桿推向右側並同時按住「扣殺」鈕
- **THEN** 觸發向右撲球（依 `dive-action` 觸發條件），與鍵盤「→ + Space」結果一致

#### Scenario: 放開搖桿即歸位
- **WHEN** 玩家放開搖桿（`pointerup`）或觸控被系統取消（`pointercancel`）
- **THEN** 旋鈕回到中心，`left`/`right`/`up` 全部設為 `false`，不會發生卡鍵

---

### Requirement: 阻止瀏覽器預設觸控手勢
觸控搖桿、動作鈕與 `#wrap` SHALL 設定 `touch-action: none` 並在 `touchstart` 呼叫 `event.preventDefault()`，且以 CSS `user-select: none` 禁止文字選取，避免捲動、雙擊縮放、長按選單干擾操作。

#### Scenario: 操作時不觸發頁面捲動或縮放
- **WHEN** 玩家在遊戲中連續拖曳搖桿並快速點按動作鈕
- **THEN** 頁面不捲動、不縮放、不彈出長按選單，輸入正常響應
