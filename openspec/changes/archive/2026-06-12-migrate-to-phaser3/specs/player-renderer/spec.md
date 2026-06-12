## ADDED Requirements

### Requirement: renderPlayer 函式封裝
系統 SHALL 提供 `renderPlayer(graphics, player)` 函式，作為角色繪製的唯一入口點。呼叫端（GameScene）不得直接呼叫低層繪圖 API 繪製角色。

#### Scenario: 無 texture 時執行 Graphics 繪製
- **WHEN** `player.texture` 為 `undefined` 或 `null`
- **THEN** `renderPlayer` 呼叫 `drawMonGraphics(graphics, player)`，以 Phaser Graphics API 繪製角色

#### Scenario: 有 texture 時預留 Sprite 路徑
- **WHEN** `player.texture` 為非空字串
- **THEN** `renderPlayer` 執行 Sprite 更新路徑（此 change 範圍內為 `return` 或 `console.warn`，不拋出例外），不執行 Graphics 繪製

---

### Requirement: drawMonGraphics 視覺保真
`drawMonGraphics(graphics, player)` SHALL 以 Phaser Graphics API 重現現有 `drawMon()` 的所有視覺元素，包含：

1. 閃電尾巴（折線多邊形）
2. 耳朵（兩側 quadratic 曲線）
3. 身體橢圓（含落地時的壓扁形變 `squash = onGround ? 1 : 0.92`）
4. 肚子半透明橢圓
5. 臉頰橙色圓點（兩個，位置依 `face` 方向調整）
6. 眼睛（黑色大圓 + 白色反光小圓，各兩個）
7. 嘴巴弧線（`strokePath`）
8. 扣殺閃光光環（`smashFlash > 0` 時）
9. 地面投影橢圓

顏色參數 SHALL 由呼叫端傳入（P1：`#ffd92e` / `#e8a800`；P2：`#ffb84d` / `#e07b00`），函式本身不硬寫顏色值。

#### Scenario: P1 角色顏色
- **WHEN** `renderPlayer(graphics, players[0])` 被呼叫
- **THEN** 身體填色為 `#ffd92e`，尾巴填色為 `#e8a800`

#### Scenario: 跳躍時身體壓扁
- **WHEN** `player.onGround` 為 `false`
- **THEN** 身體橢圓的垂直半徑乘以 `0.92`（水平半徑不變）

#### Scenario: 扣殺閃光
- **WHEN** `player.smashFlash > 0`
- **THEN** 角色外圍繪製一圈漸變透明的黃色光環，光環半徑隨 `smashFlash` 遞減而擴張

---

### Requirement: 每幀重繪機制
GameScene SHALL 在每幀 `update()` 開頭呼叫 `graphics.clear()`，再依序繪製背景、角色、球、粒子，確保無殘影。

#### Scenario: 移動後無殘影
- **WHEN** 玩家或球在畫面上移動
- **THEN** 前一幀的繪製內容不殘留，畫面只顯示當幀位置

---

### Requirement: player 物件結構
每個 player 物件 SHALL 包含以下欄位，供 `renderPlayer` 讀取：

| 欄位 | 型別 | 說明 |
|------|------|------|
| `x` | number | 中心 X 座標 |
| `y` | number | 腳底 Y 座標 |
| `r` | number | 半徑（預設 34） |
| `face` | -1 \| 1 | 朝向（-1 朝左，1 朝右） |
| `onGround` | boolean | 是否在地面（影響壓扁） |
| `smashFlash` | number | 扣殺閃光剩餘幀數 |
| `texture` | string \| null | Sprite 紋理識別字（null 代表使用 Graphics 繪製） |
| `color1` | string | 主體顏色（hex） |
| `color2` | string | 尾巴顏色（hex） |
