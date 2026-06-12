### Requirement: Phaser 3 CDN 引入
HTML 文件 SHALL 在 `<head>` 或 `<body>` 末端以 `<script>` 標籤引入 Phaser 3 CDN，引入後全域可存取 `Phaser` 物件。

#### Scenario: CDN 載入成功
- **WHEN** 瀏覽器開啟 `electric_volleyball.html`
- **THEN** `typeof Phaser` 為 `'object'`，且 `Phaser.VERSION` 為 3.x

#### Scenario: CDN 載入失敗
- **WHEN** 網路不可用導致 CDN 腳本載入失敗
- **THEN** 頁面顯示錯誤提示或靜默失敗（不拋出未捕捉例外）

---

### Requirement: Phaser.Game 初始化設定
遊戲 SHALL 以 `new Phaser.Game(config)` 初始化，config 物件 MUST 包含以下欄位：
- `type: Phaser.AUTO`（優先 WebGL，回退 Canvas）
- `width: 900`、`height: 500`（與現有 canvas 尺寸一致）
- `backgroundColor: '#d8f1ff'`（與現有天空底色一致）
- `physics: { default: 'arcade', arcade: { gravity: { y: 0 }, debug: false } }`（保留設定項，物理計算由 Scene 自行處理）
- `scene: [MenuScene, GameScene]`

#### Scenario: 遊戲正確初始化
- **WHEN** 頁面載入完成且 Phaser CDN 可用
- **THEN** 畫布以 900×500 尺寸渲染於 `#wrap` 容器內，自動套用響應式縮放

#### Scenario: 畫布嵌入頁面
- **WHEN** Phaser.Game 初始化完成
- **THEN** 產生的 `<canvas>` 元素存在於 DOM，`border-radius`、`box-shadow` 等外觀 CSS MUST 透過 `style` 設定或外部 CSS 套用，與現有視覺一致

---

### Requirement: 響應式縮放
Phaser.Game config SHALL 設定 `scale` 物件，使畫布在不同螢幕寬度下等比縮放，行為與現有 CSS `width: 100%; height: auto` 一致。

#### Scenario: 視窗寬度小於 900px
- **WHEN** 瀏覽器視窗寬度為 600px
- **THEN** 畫布縮放至 600px 寬，高度等比調整，遊戲內容不裁切

#### Scenario: 視窗寬度大於 900px
- **WHEN** 瀏覽器視窗寬度為 1400px
- **THEN** 畫布維持 900px 寬（或依 `maxWidth` 設定上限），不無限放大
