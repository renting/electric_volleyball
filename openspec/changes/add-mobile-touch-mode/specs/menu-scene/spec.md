## MODIFIED Requirements

### Requirement: MenuScene 繪製選單畫面
`MenuScene` SHALL 以 Phaser Graphics 與 Text 物件繪製與現有 `drawMenu()` 視覺一致的選單畫面，包含：背景（天空漸層、雲、沙地、網子）、兩個靜止角色、半透明遮罩、標題文字、選項文字。

選項顯示 SHALL 依裝置型態調整：

- 非觸控桌面：顯示三個選項（按 1 普通、按 2 困難、按 3 雙人），維持現有外觀。
- 觸控裝置：僅顯示單人兩個可點選項（普通、困難），MUST NOT 顯示 2P 雙人選項；提示文字改為觸控導向（點選開始）。

#### Scenario: 桌面選單正確顯示
- **WHEN** 以非觸控桌面開啟或從遊戲結束畫面返回
- **THEN** MenuScene 顯示標題「⚡ 電氣鼠排球大作戰 ⚡」與三個選項（按 1 普通、按 2 困難、按 3 雙人）

#### Scenario: 觸控裝置僅顯示單人選項
- **WHEN** 以觸控裝置開啟 MenuScene
- **THEN** 僅顯示「普通」「困難」兩個可點擊選項，不顯示 2P 雙人選項

---

### Requirement: 觸控 / 點擊啟動
在觸控裝置上，`MenuScene` SHALL 提供可點擊的模式選項：點擊「普通」以 `{mode:'1P', aiLevel:'normal'}` 啟動、點擊「困難」以 `{mode:'1P', aiLevel:'hard'}` 啟動 GameScene。各選項命中區 MUST 對應其顯示文字範圍，避免誤觸。在非觸控桌面上，保留「點擊畫布任意位置以單人普通模式啟動」之既有捷徑。

#### Scenario: 觸控點擊困難選項
- **WHEN** 觸控裝置上玩家點擊「困難」選項
- **THEN** 以 `{mode:'1P', aiLevel:'hard'}` 啟動 GameScene

#### Scenario: 桌面點擊畫面
- **WHEN** 非觸控桌面上玩家點擊畫布任意位置
- **THEN** 以單人普通模式啟動 GameScene（既有捷徑保留）
