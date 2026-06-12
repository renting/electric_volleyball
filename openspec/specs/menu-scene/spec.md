### Requirement: MenuScene 繪製選單畫面
`MenuScene` SHALL 以 Phaser Graphics 與 Text 物件繪製與現有 `drawMenu()` 視覺一致的選單畫面，包含：背景（天空漸層、雲、沙地、網子）、兩個靜止角色、半透明遮罩、標題文字、三個選項文字。

#### Scenario: 選單正確顯示
- **WHEN** 遊戲啟動或從遊戲結束畫面返回
- **THEN** MenuScene 顯示標題「⚡ 電氣鼠排球大作戰 ⚡」與三個選項（按 1 普通、按 2 困難、按 3 雙人）

---

### Requirement: 鍵盤選擇遊戲模式
`MenuScene` SHALL 透過 Phaser Keyboard API 偵測按鍵，並依下表啟動遊戲：

| 按鍵 | 動作 |
|------|------|
| `1`  | 以 `{mode:'1P', aiLevel:'normal'}` 啟動 GameScene |
| `2`  | 以 `{mode:'1P', aiLevel:'hard'}` 啟動 GameScene |
| `3`  | 以 `{mode:'2P', aiLevel:null}` 啟動 GameScene |

#### Scenario: 按下 1 啟動單人普通
- **WHEN** MenuScene 顯示中，玩家按下數字鍵 `1`
- **THEN** `scene.start('Game', {mode:'1P', aiLevel:'normal'})` 被呼叫，畫面切換至 GameScene

#### Scenario: 按下 3 啟動雙人
- **WHEN** MenuScene 顯示中，玩家按下數字鍵 `3`
- **THEN** `scene.start('Game', {mode:'2P', aiLevel:null})` 被呼叫

---

### Requirement: 觸控 / 點擊啟動
`MenuScene` SHALL 在玩家點擊或觸控畫布時，以 `{mode:'1P', aiLevel:'normal'}` 啟動 GameScene。

#### Scenario: 點擊畫面
- **WHEN** MenuScene 顯示中，玩家點擊畫布任意位置
- **THEN** 以單人普通模式啟動 GameScene

---

### Requirement: 從 GameScene 返回 MenuScene
GameScene 結束後（遊戲結束畫面），玩家 SHALL 能按下 `Enter` 或點擊畫面返回 MenuScene。

#### Scenario: 遊戲結束後返回選單
- **WHEN** GameScene 處於 `over` 狀態，玩家按下 `Enter` 或點擊畫布
- **THEN** `scene.start('Menu')` 被呼叫，畫面返回 MenuScene
