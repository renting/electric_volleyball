## Why

目前的 `electric_volleyball.html` 是一個約 560 行的單體式 vanilla Canvas 遊戲，所有系統（物理、渲染、輸入、音效、狀態機）都寫在一個 `loop()` 函式中，隨著功能增加維護難度將快速上升。遷移至 Phaser 3 可獲得成熟的 Scene 生命週期管理、Keyboard 輸入抽象、Particle 系統與未來的 Sprite 動畫支援，為後續擴充建立可維護的基礎。

## What Changes

- **BREAKING**：`electric_volleyball.html` 改為引入 Phaser 3 CDN，並以 Phaser.Game 初始化取代現有的 `requestAnimationFrame` 主迴圈。
- 遊戲邏輯拆分為兩個 Phaser Scene：`MenuScene`（選單）與 `GameScene`（遊戲本體）。
- 玩家角色渲染改以 `renderPlayer(scene, graphics, player)` 函式封裝，內部維持現有 Canvas Graphics 程式碼繪製；函式簽名預留 `player.texture` 判斷點，供未來切換 Sprite 圖片使用。
- 輸入系統改用 `Phaser.Input.Keyboard.addKeys()`，取代現有的 `window.addEventListener('keydown')`。
- 粒子特效改用 Phaser `Graphics` 手動繪製（保持與現有行為一致），或視需要改用 `ParticleEmitter`。
- 現有自製物理邏輯（重力、牆壁、網子、`tryPowerHit`）全部保留，移入 `GameScene.update()` 中執行。
- Web Audio 音效函式（`beep`）保留現有實作，在 Phaser Scene 中直接呼叫。

## Capabilities

### New Capabilities

- `phaser-bootstrap`：HTML 引入 Phaser 3 CDN，建立 `Phaser.Game` 設定檔，管理 Scene 註冊與全域參數（畫布尺寸、背景色、物理設定）。
- `menu-scene`：`MenuScene` 處理選單畫面的繪製與模式選擇（1P 普通、1P 困難、2P），遊戲開始後切換至 `GameScene`。
- `game-scene`：`GameScene` 包含完整遊戲狀態機（serve → play → point → over）、物理更新、輸入讀取、得分與結束判定，取代現有 `loop()`。
- `player-renderer`：`renderPlayer(scene, graphics, player)` 封裝角色繪製邏輯，含 `player.texture` 判斷點以利未來切換 Sprite；現階段以 Phaser `Graphics` API 重現現有外觀。

### Modified Capabilities

（目前 `openspec/specs/` 中無既有 spec，無 Modified Capabilities。）

## Impact

- **檔案**：`electric_volleyball.html` 全面改寫；無新增獨立 JS 檔（維持單一 HTML 部署方式）。
- **相依性**：新增 Phaser 3 CDN（`https://cdn.jsdelivr.net/npm/phaser@3/dist/phaser.min.js`）；移除所有自行管理的 `requestAnimationFrame`、`keydown`/`keyup` 監聽。
- **行為相容性**：玩法規則、計分、AI 邏輯、音效、鍵盤配置與現有版本保持一致；視覺外觀以現有 Canvas 繪圖程式碼為基準重現，可能有像素級微差。
- **風險**：Phaser `Graphics` API 與原生 Canvas 2D API 有差異，`drawMon` 轉寫時需逐一驗證曲線、橢圓與旋轉繪圖的正確性。
