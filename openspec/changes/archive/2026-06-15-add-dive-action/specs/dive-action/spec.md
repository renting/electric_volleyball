## ADDED Requirements

### Requirement: 撲球觸發條件
人類玩家按下扣殺鍵（1P：`Space`/`Z`；2P：P1 `G`、P2 `L`）且 `hitCooldown <= 0` 時，系統 SHALL 依下列優先序分流，且僅執行其中一項：

1. 球在扣殺判定範圍內（`Math.hypot(dx, dy) < p.r + ball.r + 24`，判定中心 `(p.x, p.y - p.r*0.4)`）→ 執行既有強力扣殺（行為不變）。
2. 球不在範圍內、遊戲處於 `play` 狀態、玩家在地面（`onGround === true`）、且「恰好按住一個」左右方向鍵 → 觸發撲球：設定 `diveT = DIVE_TIME`、`diveDir` 為該方向（左 `-1`、右 `+1`）、`face = diveDir`，並播放撲球音效與腳邊沙塵粒子。
3. 其餘情況（球不在範圍內且無方向鍵、同時按住左右、在空中、或處於 `serve` 倒數）→ 維持既有揮空冷卻（`hitCooldown = 10`）。

撲球位移中（`diveT > 0`）或起身硬直中（`diveRecover > 0`）SHALL 不可再次觸發扣殺或撲球。

#### Scenario: 球在範圍外搭配方向鍵觸發撲球
- **WHEN** `play` 狀態中玩家在地面、球不在扣殺判定範圍內，玩家按住 → 並按下扣殺鍵
- **THEN** 玩家進入撲球狀態（`diveT = DIVE_TIME`、`diveDir = 1`、`face = 1`），不進入揮空冷卻

#### Scenario: 球在範圍內維持扣殺
- **WHEN** 球在扣殺判定範圍內，玩家按住方向鍵並按下扣殺鍵
- **THEN** 執行既有 `_tryPowerHit` 強力扣殺，不觸發撲球

#### Scenario: 範圍外且未按方向鍵維持揮空
- **WHEN** 球不在扣殺判定範圍內，玩家未按住任何方向鍵（或同時按住左右）並按下扣殺鍵
- **THEN** `hitCooldown` 設為 10（與既有揮空行為一致），不觸發撲球

#### Scenario: 發球倒數期間不可撲球
- **WHEN** `serve` 狀態倒數期間玩家在地面按住方向鍵並按下扣殺鍵
- **THEN** 不觸發撲球，僅進入揮空冷卻（`hitCooldown = 10`，與現況一致）

#### Scenario: 空中不可撲球
- **WHEN** 玩家在空中（`onGround === false`）、球不在扣殺判定範圍內，按住方向鍵並按下扣殺鍵
- **THEN** 不觸發撲球，僅進入揮空冷卻

#### Scenario: 撲球期間不可重複觸發
- **WHEN** 玩家 `diveT > 0` 或 `diveRecover > 0` 時按下扣殺鍵
- **THEN** 不觸發扣殺、撲球或揮空冷卻（輸入被鎖定）

---

### Requirement: 撲球位移
撲球位移階段（`diveT > 0`）SHALL 於 `_updatePhysics()` 的玩家迴圈中每幀執行：

- `p.x += diveDir * DIVE_SPD`（`DIVE_SPD = 9`），並套用既有左右邊界 clamp（網側 `NET_X ± (NET_W/2 + p.r)`、牆側 `p.r` / `W - p.r`）；撞到邊界時位移停止但 `diveT` 照常遞減
- `diveT` 每幀減一；歸零時設定 `diveRecover = DIVE_RECOVER`（`DIVE_RECOVER = 16`）
- 位移階段與起身階段玩家的移動、跳躍、扣殺輸入 SHALL 全部鎖定（`_controlHuman` 直接略過）

`DIVE_TIME = 14`、`DIVE_SPD = 9`、`DIVE_RECOVER = 16` SHALL 定義為具名常數，便於調整。

#### Scenario: 撲球位移與時長
- **WHEN** 玩家觸發撲球（`diveDir = 1`）且未撞到邊界
- **THEN** 接下來 14 幀內每幀 `x` 增加 9，第 14 幀結束後進入 `diveRecover = 16` 的起身硬直

#### Scenario: 撲球撞到網邊停止位移
- **WHEN** 左側玩家朝網撲球並到達 `NET_X - NET_W/2 - p.r`
- **THEN** `x` 停在邊界不再前進，`diveT` 仍照常遞減直到轉入起身硬直

#### Scenario: 撲球期間輸入鎖定
- **WHEN** 玩家 `diveT > 0` 時按住反方向鍵與跳躍鍵
- **THEN** 玩家不轉向、不跳躍，位移仍沿 `diveDir` 進行

---

### Requirement: 撲球判定區與墊球回擊
撲球位移或起身硬直期間（`diveT > 0 || diveRecover > 0`），玩家與球的碰撞圓心 SHALL 由 `(p.x, p.y - p.r*0.4)` 改為 `(p.x + diveDir*p.r*0.7, p.y - p.r*0.5)`，半徑維持 `p.r + ball.r`，並受下列兩項限制：

- **自己半場限制**：撲球判定區僅對球心位於自己半場的球生效（side 0：`ball.x < NET_X`；side 1：`ball.x > NET_X`）；球心在對側時，該幀 SHALL 改用既有站立判定圓與一般碰撞（防止前移的判定圓隔網搆到對側貼網球）
- **單次觸球**：撲球狀態下的碰撞判定 SHALL 僅在 `p.hitCooldown <= 0` 時進行

位移階段（`diveT > 0`）碰到球時 SHALL 執行墊球回擊（取代一般碰撞速度轉移）：

- 先沿碰撞法線將球推出重疊（與既有碰撞一致）；推出後若 `ball.y > GROUND - ball.r - 1`，SHALL 修正 `ball.y = GROUND - ball.r - 1`，使成功墊球優先於同幀的落地得分判定
- `ball.vy = -11`、`ball.vx = diveDir * 3 + ball.vx * 0.25`、`ball.spin = diveDir * 0.25`
- `p.smashFlash = 8`，並產生粒子（`_burst(ball.x, ball.y, 0xfff3b0, 10)`）與音效回饋
- 立即結束位移階段轉入起身硬直（`diveT = 0`、`diveRecover = DIVE_RECOVER`），並設定 `p.hitCooldown = DIVE_RECOVER`，使同一次撲球僅墊球一次、起身期間不再與球碰撞

#### Scenario: 撲球墊起球
- **WHEN** 玩家撲球位移中（`diveT > 0`）球進入撲球判定區
- **THEN** 球被推出重疊後 `vy = -11` 高彈起、水平速度大幅減弱，玩家立即轉入起身硬直

#### Scenario: 撲球判定區前移觸及遠球
- **WHEN** 球的位置距玩家中心水平超過站立判定範圍、但在 `(p.x + diveDir*p.r*0.7, p.y - p.r*0.5)` 的 `p.r + ball.r` 半徑內，球心在自己半場，且玩家 `diveT > 0`
- **THEN** 判定為碰撞並執行墊球回擊

#### Scenario: 不可隔網撲到對側球
- **WHEN** 左方玩家朝網撲球被網側邊界（`NET_X - NET_W/2 - p.r`）擋下，球心位於對側半場（`ball.x > NET_X`，例如被網側推開貼在 `NET_X + NET_W/2 + ball.r`）
- **THEN** 不以撲球判定區觸碰該球；該幀改用既有站立判定圓（維持現況搆不到的行為）

#### Scenario: 落地幀墊球成功
- **WHEN** 撲球位移中球於同一幀同時進入撲球判定區並到達落地門檻（`ball.y >= GROUND - ball.r`）
- **THEN** 墊球回擊生效，`ball.y` 修正至 `GROUND - ball.r - 1`、`vy = -11`，該幀不觸發落地得分

#### Scenario: 墊球後無二次碰撞
- **WHEN** 墊球後的下一幀球仍與撲球判定區重疊
- **THEN** 因 `p.hitCooldown > 0` 不再觸發任何碰撞，球維持墊起軌跡，無重複粒子與音效

---

### Requirement: 起身硬直
起身硬直階段（`diveRecover > 0`）SHALL：

- 每幀 `diveRecover` 減一，歸零後玩家恢復所有操作
- 期間移動、跳躍、扣殺輸入維持鎖定
- 位移自然結束（未墊球）進入的起身，期間碰到球時使用撲球判定區的圓心位置（含自己半場限制），但執行既有的一般碰撞速度轉移（不觸發墊球）；墊球後進入的起身因 `hitCooldown > 0` 不與球碰撞

#### Scenario: 硬直結束恢復操作
- **WHEN** 玩家 `diveRecover` 由 1 減至 0 後按住方向鍵
- **THEN** 玩家恢復正常移動（每幀 5.4 px）

#### Scenario: 硬直期間碰球為一般反彈
- **WHEN** 玩家因位移自然結束（未墊球）進入起身、`diveRecover > 0` 時球接觸撲球判定區
- **THEN** 球依既有一般碰撞公式反彈，不套用 `vy = -11` 的墊球回擊

---

### Requirement: 回合重置清除撲球狀態
`_resetRally()` SHALL 將兩名玩家的 `diveT` 與 `diveRecover` 歸零，確保新回合開始時無殘留撲球或硬直狀態。

#### Scenario: 得分後重置
- **WHEN** 玩家撲球位移中對方得分、`point` 倒數結束呼叫 `_resetRally()`
- **THEN** 該玩家 `diveT = 0`、`diveRecover = 0`，發球倒數期間可正常操作

---

### Requirement: AI 不撲球
單人模式 AI 控制的 P2（`_controlAI`）SHALL 不觸發撲球，其移動、跳躍、`_tryPowerHit` 行為與既有版本完全一致。

#### Scenario: AI 球在範圍外不撲球
- **WHEN** 單人模式中球不在 AI 的扣殺判定範圍內
- **THEN** AI 僅以既有移動邏輯追球，`diveT` 恆為 0

---

### Requirement: 操作提示更新
頁面 `#hint` 區塊與 MenuScene 說明文字 SHALL 加入撲球操作提示，說明「扣殺鍵 + 方向鍵（球不在身邊時）= 撲球」的操作方式，1P 與 2P 鍵位皆需涵蓋。

#### Scenario: 提示文字包含撲球說明
- **WHEN** 玩家開啟頁面或進入選單
- **THEN** `#hint` 與選單文案可見撲球操作說明（含對應按鍵組合）
