## 1. 資料結構與常數

- [x] 1.1 在全域常數區（`W`、`H`、`GROUND` 同區塊）定義 `DIVE_TIME = 14`、`DIVE_SPD = 9`、`DIVE_RECOVER = 16`
- [x] 1.2 `makePlayer(side)` 新增欄位：`diveT: 0`、`diveRecover: 0`、`diveDir` 初始與 `face` 同向（side 0 為 `1`、side 1 為 `-1`）

## 2. 控制層：扣殺鍵分流

- [x] 2.1 抽出 `_inHitRange(p)` 共用判定函式（`Math.hypot(dx, dy) < p.r + ball.r + 24`，判定中心 `(p.x, p.y - p.r*0.4)`），`_tryPowerHit` 改用此函式做範圍判斷（行為不變）
- [x] 2.2 1P 模式將 `Space`/`Z` 合併為單一虛擬 hit 狀態（`{isDown: space.isDown || z.isDown}`）傳入 `_controlHuman`，移除 `update()` 內 1P 獨立的扣殺判斷，統一 1P/2P 控制路徑
- [x] 2.3 `_controlHuman` 開頭加入輸入鎖定：`p.diveT > 0 || p.diveRecover > 0` 時直接 return
- [x] 2.4 `_controlHuman` 實作扣殺鍵三向分流：範圍內 → `_tryPowerHit`；範圍外 + `onGround` + 恰好按住一個左右方向鍵 → `_startDive(p, dir)`；其餘 → `p.hitCooldown = 10`（同時按住左右視為無方向）
- [x] 2.5 實作 `_startDive(p, dir)`：設定 `diveT = DIVE_TIME`、`diveDir = dir`、`face = dir`，播放撲球音效（`beep`）與腳邊沙塵粒子（`_burst`，沙色 `0xe0bf66`）
- [ ] 2.6 驗證：1P 範圍內按 Space 仍為扣殺；範圍外 + → 觸發撲球；範圍外無方向鍵為揮空冷卻；空中按扣殺鍵不撲球；撲球/硬直中按鍵無反應

## 3. 物理：撲球位移與碰撞

- [x] 3.1 `_updatePhysics()` 玩家迴圈加入 dive 步進：`diveT > 0` 時 `p.x += diveDir * DIVE_SPD`（沿用既有左右邊界 clamp）、`diveT--`，歸零時 `diveRecover = DIVE_RECOVER`；`diveRecover > 0` 時每幀遞減
- [x] 3.2 玩家與球碰撞：`diveT > 0 || diveRecover > 0` 時碰撞圓心改為 `(p.x + diveDir*p.r*0.7, p.y - p.r*0.5)`，半徑維持 `p.r + b.r`
- [x] 3.3 位移階段（`diveT > 0`）碰撞改為墊球回擊：沿法線推出重疊後設定 `b.vy = -11`、`b.vx = diveDir * 3 + b.vx * 0.25`、`b.spin = diveDir * 0.25`、`p.smashFlash = 8`，觸發 `_burst(b.x, b.y, 0xfff3b0, 10)` 與音效，並立即 `diveT = 0`、`diveRecover = DIVE_RECOVER`
- [x] 3.4 起身階段（`diveRecover > 0`）碰撞沿用既有一般速度轉移公式（僅圓心位置不同）
- [x] 3.5 `_resetRally()` 將兩名玩家的 `diveT`、`diveRecover` 歸零
- [ ] 3.6 驗證：撲球可救到站立搆不到的遠球且球高彈起、水平減弱留在己方附近；同一次撲球僅墊球一次；朝網撲球被邊界擋下仍可救網前球

## 4. 撲球姿態繪製（drawMonGraphics）

- [x] 4.1 `drawMonGraphics` 依 dive 狀態計算形變參數：`diveT > 0` 為固定撲球姿態（水平 ×1.6、垂直 ×0.5、中心 `(x + diveDir*r*0.45, GROUND - r*0.55)`）；`diveRecover > 0` 以 `t = diveRecover / DIVE_RECOVER` 線性插值回站姿
- [x] 4.2 身體與肚子橢圓套用形變參數與貼地中心
- [x] 4.3 尾巴、耳朵、眼睛、臉頰、嘴巴以撲球身體中心為基準重新定位（朝 `diveDir`），所有元素不得缺漏
- [x] 4.4 地面投影橢圓寬度隨身體水平延伸同步加寬
- [ ] 4.5 驗證：向左/向右撲球姿態鏡像正確；起身過程逐幀漸變無跳變；非撲球狀態的繪製與現有版本完全一致（站立、跳躍壓扁、扣殺光環）

## 5. 操作提示文案

- [x] 5.1 更新頁面 `#hint`：加入撲球操作說明（1P：球不在身邊時 `Space`/`Z` + 方向鍵；2P：P1 `G` + `A`/`D`、P2 `L` + 方向鍵）
- [x] 5.2 更新 MenuScene 說明文字，加入撲球提示一行

## 6. 整合驗證

- [ ] 6.1 1P 普通完整一局：扣殺、撲球、揮空、跳躍均正常，AI 全程不出現撲球姿態
- [ ] 6.2 2P 一局：P1（`G` + `A`/`D`）與 P2（`L` + 方向鍵）可各自獨立撲球，互不干擾
- [ ] 6.3 邊界情境：撲球位移中按 `P` 暫停再恢復，位移與硬直正確續算；撲球中對方得分後 `_resetRally()` 清除狀態，發球倒數可正常操作
- [ ] 6.4 平衡 playtest：故意撲空確認 16 幀硬直有明顯懲罰感、撲球未取代普通移動；按住扣殺鍵 + 方向鍵的連續撲球行為可接受（若過強，依 design 改為 `JustDown` 觸發並記錄）
