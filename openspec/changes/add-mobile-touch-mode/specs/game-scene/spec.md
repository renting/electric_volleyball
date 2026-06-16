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
