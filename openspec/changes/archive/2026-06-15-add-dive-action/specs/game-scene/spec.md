## MODIFIED Requirements

### Requirement: 輸入系統使用 Phaser Keyboard API
`GameScene` SHALL 以 `this.input.keyboard.addKeys({...})` 建立按鍵映射。

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

1P 模式的 `Space`/`Z` SHALL 合併為單一 hit 狀態傳入與 2P 相同的控制路徑，分流邏輯僅實作一份。

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

---

### Requirement: 物理更新完整移植
`GameScene.update()` SHALL 執行與現有版本功能等價的物理計算，包含：

- 玩家重力（`vy += 0.7`）與落地偵測
- 玩家左右邊界（依 `side` 限制，不可跨越網子）
- 撲球位移與起身硬直的每幀步進（`diveT`/`diveRecover`，行為依 `dive-action` capability）
- 球重力（`vy += 0.42`）、旋轉衰減（`spin *= 0.99`）
- 牆壁與天花板反彈（含速度衰減係數）
- 網子碰撞（頂部反彈、側面推開）
- 玩家與球的圓形碰撞偵測與速度轉移（非撲球狀態；撲球期間的判定區與墊球回擊依 `dive-action` capability）
- 球速限制（最大 17）

#### Scenario: 球碰網頂反彈
- **WHEN** 球從上方接觸網頂（`ball.y > NET_TOP - ball.r` 且 `ball.vy > 0`）
- **THEN** `ball.vy` 反向並乘以衰減係數 0.75，`ball.y` 修正至網頂以上

#### Scenario: 玩家不可越過網子
- **WHEN** 玩家移動至網子邊緣
- **THEN** 玩家 `x` 被限制在 `NET_X ± (NET_W/2 + player.r)` 範圍內
