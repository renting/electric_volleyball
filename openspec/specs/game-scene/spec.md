### Requirement: GameScene 狀態機
`GameScene` SHALL 實作與現有版本一致的五狀態機：`serve → play → point → over`，加上 `paused` 旗標（非獨立狀態）。

狀態轉換規則：
- `serve`：倒數 50 幀後轉 `play`
- `play`：球落地後依得分判斷轉 `point` 或 `over`
- `point`：倒數 60 幀後轉 `serve`（換邊）
- `over`：等待玩家按 `Enter`/點擊（→ MenuScene）或 `R`（→ 重新開始）

#### Scenario: serve 倒數結束
- **WHEN** GameScene 進入 `serve` 狀態且已過 50 幀
- **THEN** 狀態轉為 `play`，球開始受物理影響

#### Scenario: 球落地得分
- **WHEN** `play` 狀態中球的 `y >= GROUND - ball.r`
- **THEN** 對應方得分加一，狀態轉為 `point`（未達 15 分）或 `over`（達 15 分）

---

### Requirement: 輸入系統使用 Phaser Keyboard API
`GameScene` SHALL 以 `this.input.keyboard.addKeys({...})` 建立按鍵映射，取代現有的 `window.addEventListener('keydown')`。

鍵盤配置 MUST 與現有版本一致：

| 功能 | 1P 模式 | 2P P1 | 2P P2 |
|------|---------|-------|-------|
| 左移 | ←       | A     | ←     |
| 右移 | →       | D     | →     |
| 跳躍 | ↑       | W     | ↑     |
| 扣殺 | Space / Z | G   | L     |
| 暫停 | P / Esc | P / Esc | P / Esc |
| 重新開始 | R | R | R |

#### Scenario: 1P 模式左右移動
- **WHEN** `play` 或 `serve` 狀態中玩家按住方向鍵 ←
- **THEN** P1 的 `x` 每幀減少 5.4（與現有速度一致）

#### Scenario: 暫停與恢復
- **WHEN** 遊戲進行中玩家按下 `P` 或 `Esc`
- **THEN** `paused` 旗標切換，物理更新停止（或繼續），畫面顯示暫停提示

---

### Requirement: 物理更新完整移植
`GameScene.update()` SHALL 執行與現有版本功能等價的物理計算，包含：

- 玩家重力（`vy += 0.7`）與落地偵測
- 玩家左右邊界（依 `side` 限制，不可跨越網子）
- 球重力（`vy += 0.42`）、旋轉衰減（`spin *= 0.99`）
- 牆壁與天花板反彈（含速度衰減係數）
- 網子碰撞（頂部反彈、側面推開）
- 玩家與球的圓形碰撞偵測與速度轉移
- 球速限制（最大 17）

#### Scenario: 球碰網頂反彈
- **WHEN** 球從上方接觸網頂（`ball.y > NET_TOP - ball.r` 且 `ball.vy > 0`）
- **THEN** `ball.vy` 反向並乘以衰減係數 0.75，`ball.y` 修正至網頂以上

#### Scenario: 玩家不可越過網子
- **WHEN** 玩家移動至網子邊緣
- **THEN** 玩家 `x` 被限制在 `NET_X ± (NET_W/2 + player.r)` 範圍內

---

### Requirement: AI 邏輯完整保留
單人模式下，P2 的 AI 行為（移動追球、跳躍時機、`tryPowerHit` 機率）SHALL 與現有版本一致，包含普通與困難兩個難度的速度與命中率差異。

#### Scenario: AI 困難模式速度
- **WHEN** 選擇 `aiLevel:'hard'`，AI 控制的 P2 移動
- **THEN** 每幀移動速度為 5.6（普通模式為 4.6）

---

### Requirement: 粒子系統保留自製邏輯
粒子系統 SHALL 維持現有自製陣列實作（`burst()`、`stepParticles()`），在 `GameScene.update()` 中每幀更新，並以 Phaser Graphics 繪製。

#### Scenario: 得分時爆炸粒子
- **WHEN** 球落地得分
- **THEN** 落點位置產生約 26 個橙色粒子，向四周擴散並逐幀消亡

---

### Requirement: 得分顯示
GameScene SHALL 在畫面上方顯示雙方即時分數與標籤（YOU/COM 或 PLAYER 1/PLAYER 2），字體大小與位置與現有版本一致。

#### Scenario: 得分更新顯示
- **WHEN** 任一方得分
- **THEN** 對應的分數 Text 物件即時更新為新數字

---

### Requirement: 畫面震動效果
得分或強力扣殺觸發時，GameScene SHALL 模擬畫面震動效果（`shakeT` 倒數幀數內隨機偏移 camera 或 Graphics 位置）。

#### Scenario: 強力扣殺觸發震動
- **WHEN** `tryPowerHit` 成功命中
- **THEN** 後續 8 幀內畫面有隨機偏移震動效果
