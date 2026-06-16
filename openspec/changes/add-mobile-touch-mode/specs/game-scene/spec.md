## ADDED Requirements

### Requirement: 球體尺寸放大
球的半徑 `ball.r` SHALL 由原本的 17 放大為 28，以提升手機與桌面的可視性並降低接球難度。放大後的半徑 MUST 一致套用於碰撞偵測、網子判定、落地判定與繪製，維持物理一致性。

#### Scenario: 球以放大尺寸渲染與碰撞
- **WHEN** 任一回合進行中
- **THEN** 球以半徑 28 繪製，且玩家碰撞、牆面/網子反彈、落地得分皆以相同半徑計算

---

### Requirement: 扣殺彗星殘影
`GameScene` SHALL 為球維護近期球心位置的軌跡陣列（`ball.trail`，上限約 16 筆，每物理幀記錄一次），並據此在球後方繪製漸層淡出的殘影。強力扣殺（`_tryPowerHit` 成功命中）SHALL 觸發殘影強度計時器（`ball.smash`），使殘影最為明顯並呈現黃→橘的彗星漸層；非扣殺但高速飛行時 SHALL 顯示較淡的拖尾；低速時不顯示。每回合重置（`_resetRally`）SHALL 清空軌跡與計時器。殘影 MUST 繪製於球體之下、不得遮蓋球本體，且不影響任何物理計算。

#### Scenario: 扣殺出現彗星殘影
- **WHEN** 玩家成功觸發強力扣殺、球高速飛出
- **THEN** 球後方出現黃→橘漸層、由尾至頭逐漸變亮變大的彗星殘影

#### Scenario: 低速時不顯示殘影
- **WHEN** 球以低速移動（例如發球前或輕墊球）
- **THEN** 不繪製彗星殘影

#### Scenario: 回合重置清空殘影
- **WHEN** 得分後進入下一回合（`_resetRally`）
- **THEN** `ball.trail` 與 `ball.smash` 被清空，畫面不殘留上一球的拖尾

## MODIFIED Requirements

### Requirement: 輸入系統使用 Phaser Keyboard API
`GameScene` SHALL 以統一的「輸入狀態抽象」驅動玩家控制：鍵盤與觸控兩個來源每幀更新同一組輸入狀態物件（`left`/`right`/`up`/`hit`，各具 `isDown` 屬性），再傳入既有 `_controlHuman` 控制路徑。鍵盤來源 SHALL 以 `this.input.keyboard.addKeys({...})` 建立映射；觸控來源依 `touch-controls` capability 提供。兩來源以邏輯 OR 合併。

鍵盤配置 MUST 與現有版本一致：

| 功能 | 1P 模式 | 2P P1 | 2P P2 |
|------|---------|-------|-------|
| 左移 | ←       | A     | ←     |
| 右移 | →       | D     | →     |
| 跳躍 | ↑       | W     | ↑     |
| 扣殺 / 撲球 | Space / Z | G   | L     |
| 暫停 | P / Esc | P / Esc | P / Esc |
| 重新開始 | R | R | R |

扣殺鍵 SHALL 依球的距離與方向鍵組合分流為三種結果（詳細行為見 `dive-action` capability）：

1. 球在扣殺判定範圍內 → 強力扣殺（既有行為）
2. `play` 狀態 + 球不在範圍內 + 在地面 + 恰好按住一個左右方向鍵 → 撲球
3. 其餘情況 → 揮空冷卻（既有行為）

1P 模式的 `Space`/`Z` SHALL 合併為單一 hit 狀態傳入與 2P 相同的控制路徑，分流邏輯僅實作一份。觸控輸入 MUST 經由同一統一輸入狀態進入此控制路徑，MUST NOT 另立分流邏輯，且 MUST NOT 改變任何玩法數值與物理行為。

#### Scenario: 1P 模式左右移動
- **WHEN** `play` 或 `serve` 狀態中玩家按住方向鍵 ←（非撲球位移／起身硬直中）
- **THEN** P1 的 `x` 每幀減少 5.4（與現有速度一致）

#### Scenario: 暫停與恢復
- **WHEN** 遊戲進行中玩家按下 `P` 或 `Esc`
- **THEN** `paused` 旗標切換，物理更新停止（或繼續），畫面顯示暫停提示

#### Scenario: 扣殺鍵分流至撲球
- **WHEN** `play` 狀態中球不在扣殺判定範圍內，玩家在地面按住 → 並按下 `Space`
- **THEN** 觸發撲球（依 `dive-action` capability 的觸發條件），不執行扣殺也不進入揮空冷卻

#### Scenario: 2P 模式雙方皆可撲球
- **WHEN** 2P 模式中 P1 按住 `D` + `G`、或 P2 按住 → + `L`，且各自滿足撲球觸發條件
- **THEN** 對應玩家觸發撲球，行為與 1P 模式一致

#### Scenario: 觸控與鍵盤共用控制路徑
- **WHEN** 觸控裝置上玩家以虛擬方向鈕與動作鈕操作（或同一裝置兼用鍵盤）
- **THEN** 觸控狀態與鍵盤狀態以 OR 合併為統一輸入狀態後進入 `_controlHuman`，三向分流結果與純鍵盤操作完全一致
