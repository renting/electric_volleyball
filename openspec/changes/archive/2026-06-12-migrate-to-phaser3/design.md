## Context

現有的 `electric_volleyball.html` 是單一 HTML 檔案部署的 vanilla Canvas 遊戲。遷移目標是在維持單檔部署方式的前提下，將架構重構為 Phaser 3 Scene 體系，並在渲染層建立 Sprite 抽象以供未來擴充。

技術約束：
- 部署方式維持單一 HTML 檔案（無 bundler、無 npm）
- 瀏覽器直接開啟即可執行，不需要本地伺服器
- 現有玩法、AI 邏輯、音效行為必須與遷移前一致

## Goals / Non-Goals

**Goals:**
- 建立 Phaser 3 Scene 架構（MenuScene / GameScene）
- 將所有輸入改為 Phaser Keyboard API
- 以 Phaser Graphics 重現現有角色與場景繪圖
- 封裝 `renderPlayer()` 並預留 `player.texture` Sprite 切換點
- 物理邏輯完整保留並移入 `GameScene.update()`

**Non-Goals:**
- 不實作 Sprite 圖片載入（未來 change 處理）
- 不引入 Arcade Physics 或 Matter.js（維持自製物理）
- 不新增遊戲功能（新模式、新角色、新關卡等）
- 不改變現有鍵盤配置

## Decisions

### 決策一：維持單一 HTML 檔案，使用 Phaser CDN

**選擇**：`<script src="https://cdn.jsdelivr.net/npm/phaser@3/dist/phaser.min.js">` 直接引入，遊戲程式碼寫在同一 HTML 的 `<script>` 標籤內。

**理由**：現有部署方式是「瀏覽器直接開啟 HTML」。引入 bundler（Vite、webpack）或 npm 工作流會改變開發/部署環境，超出此 change 範圍。CDN 引入的 Phaser 體積約 ~1MB，可接受。

**放棄方案**：npm + Vite — 架構更現代，但開發環境複雜度大增，非此 change 目標。

---

### 決策二：維持自製物理，不使用 Phaser Arcade Physics

**選擇**：現有物理邏輯（重力、牆壁反彈、網子碰撞、`tryPowerHit` 打擊邏輯）原封不動移入 `GameScene.update()`。

**理由**：`tryPowerHit` 的打球手感（方向、力道、旋轉）是遊戲核心，是自定義商業邏輯，Arcade Physics 無法直接表達。強行套入 Arcade Physics 會增加複雜度且無實質收益。Phaser 的 `update()` 迴圈完全支援手動物理計算。

**放棄方案**：Arcade Physics — 適合一般平台跳躍，但對「排球打擊手感」的控制不足。

---

### 決策三：渲染改用 Phaser Graphics，封裝 renderPlayer()

**選擇**：建立一個 Phaser `Graphics` 物件，在每幀 `update()` 呼叫前 `clear()`，再呼叫 `renderPlayer(graphics, player)` 重新繪製所有角色。

**理由**：現有的 `drawMon()` 使用 Canvas 2D API（`ctx.arc`、`ctx.ellipse`、`ctx.quadraticCurveTo`）。Phaser `Graphics` 的 API（`fillCircle`、`fillEllipse`、`lineBetween` 等）語義相同但語法不同，需要逐一轉換。封裝成獨立函式後，未來切換 Sprite 只需修改這一層。

`player.texture` 判斷邏輯：
```js
function renderPlayer(graphics, player) {
  if (player.texture) {
    // 未來：player.sprite.setPosition(player.x, player.y)
    return;
  }
  // 現階段：Graphics 程式碼繪製
  drawMonGraphics(graphics, player);
}
```

**放棄方案**：直接在 GameScene 內散落繪圖呼叫 — 日後難以切換到 Sprite，違反 explore 階段定案的設計原則。

---

### 決策四：Web Audio 音效保留原有實作

**選擇**：`beep()` 函式直接複製至 GameScene，在需要音效的地方呼叫，不使用 `Phaser.Sound`。

**理由**：現有 `beep()` 使用 Web Audio API 動態合成聲音，不需要載入音效檔案，符合單一 HTML 無外部資源的約束。`Phaser.Sound` 主要管理音效檔案的載入與播放，對合成音效沒有優勢。

---

### 決策五：Scene 結構

```
Phaser.Game
  ├── MenuScene   (key: 'Menu')
  │     └── 選單繪製、模式選擇 → scene.start('Game', {mode, aiLevel})
  └── GameScene   (key: 'Game')
        ├── preload()   (空，無外部資源)
        ├── create()    (初始化 Graphics、Keyboard、遊戲物件)
        └── update()    (輸入 → 物理 → 粒子 → 繪圖)
```

選單結束時以 `scene.start('Game', data)` 傳遞 `{mode, aiLevel}`，GameScene 於 `create()` 的 `this.scene.settings.data` 讀取。

## Risks / Trade-offs

- **Phaser Graphics API 轉換成本**：`drawMon` 約 60 行使用 `quadraticCurveTo`、`ellipse` 等原生 Canvas 方法，Phaser Graphics 的對應方法語義相似但不完全一致（例如 `strokePath` 流程不同），轉換時需要逐項驗證外觀。→ 在任務中安排每個繪圖元素獨立驗證。

- **Graphics 每幀 clear + redraw 效能**：Phaser Graphics 物件每幀清除再重繪，與原生 Canvas 行為相同，在此遊戲規模（2 個角色 + 1 顆球 + 粒子）不會有效能問題。→ 可接受，無需 mitigation。

- **CDN 網路相依性**：離線環境無法載入 Phaser CDN。→ 接受此限制，屬部署環境問題，非此 change 範圍。

## Open Questions

- 雲端粒子系統：目前粒子以自製陣列管理，是否在此 change 內改為 Phaser `ParticleEmitter`？
  → **暫定保留自製粒子邏輯**，以降低轉換複雜度；若日後新增大量粒子效果再切換。
