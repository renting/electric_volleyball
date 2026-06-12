## 1. 基礎架構建立（phaser-bootstrap）

- [x] 1.1 在 `electric_volleyball.html` 的 `<head>` 加入 Phaser 3 CDN `<script>` 標籤（`phaser.min.js`）
- [x] 1.2 移除現有 `<canvas id="game">` 元素及其相關 CSS（Phaser 將自動建立 canvas）
- [x] 1.3 建立 `Phaser.Game` config 物件：設定 `type: Phaser.AUTO`、`width: 900`、`height: 500`、`backgroundColor`、`scale`（FIT + CENTER）、`scene: [MenuScene, GameScene]`
- [x] 1.4 驗證：開啟 HTML 後瀏覽器 console 無錯誤，畫布正確渲染於 `#wrap` 容器內，視窗縮放時等比縮放

## 2. 共用常數與資料結構

- [x] 2.1 定義全域常數：`W`、`H`、`GROUND`、`NET_X`、`NET_W`、`NET_TOP`、`WIN_SCORE`（數值與現有版本一致）
- [x] 2.2 定義 `makePlayer(side)` 工廠函式，player 物件包含 `texture: null`、`color1`、`color2` 欄位（依 spec `player-renderer`）
- [x] 2.3 定義 `beep(freq, dur, type, vol)` 函式（直接複製現有實作，維持 Web Audio API）

## 3. renderPlayer 與 drawMonGraphics（player-renderer）

- [x] 3.1 建立 `renderPlayer(graphics, player)` 函式，含 `player.texture` 判斷分支（有 texture 時 `return`，無則呼叫 `drawMonGraphics`）
- [x] 3.2 實作 `drawMonGraphics(graphics, player)`：以 Phaser Graphics API 繪製閃電尾巴（折線 polygon）
- [x] 3.3 實作耳朵（兩側 quadratic 曲線，使用 `lineTo` + `moveTo` 模擬）
- [x] 3.4 實作身體橢圓（`graphics.fillEllipse`，套用 `squash` 壓扁係數）
- [x] 3.5 實作肚子半透明橢圓（`fillStyle` 帶 alpha）
- [x] 3.6 實作臉頰橙色圓點、眼睛黑圓 + 白色反光
- [x] 3.7 實作嘴巴弧線（`strokePath` + `arc`）
- [x] 3.8 實作扣殺閃光光環（`smashFlash > 0` 時以 `strokeCircle` 繪製）
- [x] 3.9 實作地面投影橢圓（固定位於 `GROUND + 8`）
- [x] 3.10 驗證：兩個角色外觀與現有版本視覺一致（顏色、比例、朝向、壓扁、閃光）

## 4. MenuScene

- [x] 4.1 建立 `MenuScene` class（`extends Phaser.Scene`，key: `'Menu'`），實作 `create()` 與 `update()`
- [x] 4.2 在 `create()` 以 Phaser `Graphics` 物件建立背景繪製層（天空、太陽、雲動畫、沙地、網子）
- [x] 4.3 在 `create()` 繪製兩個靜止角色（呼叫 `renderPlayer`）與半透明遮罩
- [x] 4.4 加入標題與選項 `Text` 物件（字體、顏色、位置與現有 `drawMenu` 一致）
- [x] 4.5 以 `this.input.keyboard.addKeys` 設定 `Digit1`、`Digit2`、`Digit3` 按鍵，`update()` 中偵測並呼叫 `this.scene.start('Game', {...})`
- [x] 4.6 加入 `pointerdown` 事件，點擊畫布以單人普通模式啟動 GameScene
- [x] 4.7 驗證：選單畫面顯示正確，按 1/2/3 能正確切換至 GameScene（以 console.log 確認傳入 data）

## 5. GameScene 骨架與初始化

- [x] 5.1 建立 `GameScene` class（`extends Phaser.Scene`，key: `'Game'`），實作 `create()` 與 `update()`
- [x] 5.2 在 `create()` 從 `this.scene.settings.data` 讀取 `{mode, aiLevel}`，初始化 `score`、`state`、`players`、`ball`、`particles` 等遊戲物件
- [x] 5.3 在 `create()` 建立 Phaser `Graphics` 物件（背景層、角色層各一，或合用一個）
- [x] 5.4 以 `this.input.keyboard.addKeys` 建立完整按鍵映射（移動、跳躍、扣殺、暫停 P/Esc、重新開始 R、結束返回 Enter）

## 6. GameScene 物理系統

- [x] 6.1 將現有 `physics()` 函式內容移植至 `GameScene` 的私有方法 `_updatePhysics()`，在 `update()` 中呼叫
- [x] 6.2 移植玩家重力、落地、邊界限制邏輯（含網子左右界限）
- [x] 6.3 移植球的重力、旋轉衰減、牆壁 / 天花板反彈
- [x] 6.4 移植網子碰撞邏輯（頂部反彈、側面推開）
- [x] 6.5 移植玩家與球的圓形碰撞偵測及速度轉移
- [x] 6.6 移植球速上限（17）與 `tryPowerHit` 邏輯
- [x] 6.7 驗證：球能正確在場地中滾動、碰牆反彈、碰網子反彈、被玩家打到後飛出

## 7. GameScene 控制系統（玩家 + AI）

- [x] 7.1 移植 `controlHuman(p, left, right, up, hit)` 邏輯，改為讀取 Phaser keyboard 物件的 `isDown` 屬性
- [x] 7.2 移植 `controlAI(p)` 及 `predictBallX()` 邏輯（含普通 / 困難難度參數）
- [x] 7.3 移植 2P 模式下 P2 的 human 控制（方向鍵 + L）
- [x] 7.4 驗證：1P 模式玩家可正常移動跳躍扣殺，AI 能追球並還擊；2P 模式雙人可各自獨立操控

## 8. GameScene 狀態機與得分

- [x] 8.1 移植 `serve → play → point → over` 狀態轉換邏輯（`serveTimer`、`pointTimer`）
- [x] 8.2 移植 `resetRally(side)` 函式
- [x] 8.3 移植 `paused` 旗標與暫停 / 恢復邏輯
- [x] 8.4 移植重新開始（R 鍵）與返回選單（Enter / 點擊）邏輯
- [x] 8.5 以 Phaser `Text` 物件顯示即時分數（`score[0]`、`score[1]`），得分時呼叫 `setText` 更新
- [x] 8.6 驗證：完整跑完一局遊戲（serve → play → 得分 → 下一球 → 勝利），所有狀態轉換正確

## 9. GameScene 繪圖系統

- [x] 9.1 移植 `drawBackground()` 邏輯為 Phaser Graphics 繪製（天空漸層、太陽、雲動畫、沙地、網子橫線）
- [x] 9.2 在 `update()` 每幀呼叫 `graphics.clear()` 後依序繪製：背景 → 角色（`renderPlayer`）→ 球 → 粒子
- [x] 9.3 移植 `drawBall()` 為 Phaser Graphics（白底、紅色扇形、邊框，含旋轉與地面投影）
- [x] 9.4 移植畫面震動效果（`shakeT`：以 `this.cameras.main.shake` 或手動偏移 Graphics 位置實作）
- [x] 9.5 移植粒子的 `burst()` / `stepParticles()` 邏輯，以 Phaser Graphics `fillRect` 繪製
- [x] 9.6 移植所有 overlay 文字（READY、得分提示、暫停、遊戲結束）為 Phaser `Text` 物件或 Graphics 填色 + Text
- [x] 9.7 驗證：整體視覺外觀與現有版本一致（背景、角色、球、粒子、文字皆正確顯示）

## 10. 最終整合驗證

- [x] 10.1 完整遊玩一局 1P 普通，確認：選單 → 遊戲 → 得分 → 結束 → 返回選單 全流程無錯誤
- [x] 10.2 完整遊玩一局 1P 困難，確認 AI 速度與命中率明顯高於普通
- [x] 10.3 完整遊玩一局 2P，確認雙人各自操控、暫停、重新開始均正常
- [x] 10.4 測試響應式縮放：縮小瀏覽器視窗至 400px 寬，確認畫布等比縮放無變形
- [x] 10.5 確認 `player.texture = null`（現有狀態）時角色以 Graphics 繪製；在 console 手動設定 `players[0].texture = 'test'` 後確認 renderPlayer 走 Sprite 分支且不拋出例外
- [x] 10.6 移除現有 `electric_volleyball.html` 的舊版 JS（確認無殘留的 `requestAnimationFrame loop`、`window.addEventListener keydown`）
