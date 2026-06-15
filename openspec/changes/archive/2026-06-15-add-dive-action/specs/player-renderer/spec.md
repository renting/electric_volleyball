## MODIFIED Requirements

### Requirement: drawMonGraphics 視覺保真
`drawMonGraphics(graphics, player)` SHALL 以 Phaser Graphics API 繪製以下視覺元素：

1. 閃電尾巴（折線多邊形）
2. 耳朵（兩側 quadratic 曲線）
3. 身體橢圓（含落地時的壓扁形變 `squash = onGround ? 1 : 0.92`）
4. 肚子半透明橢圓
5. 臉頰橙色圓點（兩個，位置依 `face` 方向調整）
6. 眼睛（黑色大圓 + 白色反光小圓，各兩個）
7. 嘴巴弧線（`strokePath`）
8. 扣殺閃光光環（`smashFlash > 0` 時）
9. 地面投影橢圓

主體（身體、耳朵）與尾巴的顏色 SHALL 取自 player 物件的 `color1` / `color2`（P1：`0xffd92e` / `0xe8a800`；P2：`0xffb84d` / `0xe07b00`），函式不得硬寫主體配色；臉頰（`0xff7a3d`）、眼睛與嘴巴（`0x23272e`）、肚子、閃光等固定點綴色維持函式內既有值。

撲球姿態 SHALL 依 dive 狀態欄位繪製：

- 位移階段（`diveT > 0`）：身體橢圓水平直徑 ×1.6、垂直直徑 ×0.5，身體中心移至 `(x + diveDir*r*0.45, GROUND - r*0.55)`，呈扁平延伸貼地姿態
- 起身階段（`diveRecover > 0`）：令 `t = diveRecover / DIVE_RECOVER`，形變參數（水平 `1 + 0.6t`、垂直 `1 - 0.5t`、中心水平偏移 `diveDir*r*0.45*t`、中心高度）自撲球姿態線性插值回站姿，四項皆插值以避免 `t = 0` 時跳變
- 尾巴、耳朵、眼睛、臉頰、嘴巴等元素以撲球時的身體中心為基準重新定位，不得缺漏
- 地面投影橢圓寬度隨身體水平延伸同步加寬

#### Scenario: P1 角色顏色
- **WHEN** `renderPlayer(graphics, players[0])` 被呼叫
- **THEN** 身體填色為 `0xffd92e`，尾巴填色為 `0xe8a800`

#### Scenario: 跳躍時身體壓扁
- **WHEN** `player.onGround` 為 `false` 且非撲球狀態
- **THEN** 身體橢圓的垂直半徑乘以 `0.92`（水平半徑不變）

#### Scenario: 扣殺閃光
- **WHEN** `player.smashFlash > 0`
- **THEN** 角色外圍繪製一圈漸變透明的黃色光環，光環半徑隨 `smashFlash` 遞減而擴張

#### Scenario: 撲球姿態扁平延伸
- **WHEN** `player.diveT > 0` 且 `diveDir = 1`
- **THEN** 身體橢圓以 `(x + r*0.45, GROUND - r*0.55)` 為中心、水平直徑 ×1.6、垂直直徑 ×0.5 繪製，臉部元素朝 `diveDir` 方向

#### Scenario: 起身過渡漸變
- **WHEN** `player.diveRecover` 由 `DIVE_RECOVER` 遞減至 0
- **THEN** 身體形變逐幀自扁平延伸線性回復至站姿，過程無跳變

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
| `diveT` | number | 撲球位移剩餘幀數（0 = 非撲球中） |
| `diveDir` | -1 \| 1 | 撲球方向（初始與 `face` 同向） |
| `diveRecover` | number | 起身硬直剩餘幀數（0 = 非硬直中） |
| `texture` | string \| null | Sprite 紋理識別字（null 代表使用 Graphics 繪製） |
| `color1` | number | 主體顏色（`0xRRGGBB`） |
| `color2` | number | 尾巴顏色（`0xRRGGBB`） |

#### Scenario: 撲球欄位初始值
- **WHEN** `makePlayer(side)` 建立玩家物件
- **THEN** `diveT = 0`、`diveRecover = 0`、`diveDir` 與初始 `face` 同向
