## ADDED Requirements

### Requirement: 觸控裝置偵測
系統 SHALL 以 `matchMedia('(pointer: coarse)')`（輔以 `'ontouchstart' in window`）判定是否為觸控裝置，並以此決定：是否顯示觸控面板、`MenuScene` 是否僅顯示單人選項、是否隱藏鍵盤說明文字。MUST NOT 僅以 User-Agent 字串判定。

#### Scenario: 判定為觸控裝置
- **WHEN** 開啟遊戲的裝置主要指標為 coarse（手機／平板）
- **THEN** 系統進入觸控模式：顯示觸控面板、隱藏鍵盤說明、選單僅顯示單人選項

#### Scenario: 判定為桌面裝置
- **WHEN** 開啟遊戲的裝置為純鍵鼠桌面
- **THEN** 系統維持原狀：不顯示面板、保留鍵盤說明與 1/2/3 三選項

---

### Requirement: viewport 與行動版面
HTML `<head>` 的 `<meta name="viewport">` SHALL 設定 `width=device-width, initial-scale=1.0, viewport-fit=cover, user-scalable=no`。觸控裝置且橫向時，頁面 SHALL 隱藏 `<h1>` 標題與 `#hint` 鍵盤說明，使 `#wrap` 在 `100vw × 100vh` 範圍內最大化（畫面比例由 `Scale.FIT` 維持，不裁切遊戲內容）。桌面版 CSS 版面規則 MUST 維持不變。

#### Scenario: 手機橫向最大化遊戲區
- **WHEN** 觸控裝置以橫向開啟遊戲
- **THEN** 標題與鍵盤說明隱藏，遊戲畫布盡量填滿可視區域，內容完整不裁切

#### Scenario: 桌面版面不變
- **WHEN** 桌面瀏覽器開啟遊戲
- **THEN** 標題、鍵盤說明與 `width: min(96vw, 920px)` 等既有版面維持原狀

---

### Requirement: 橫向閘門與直向提示
系統 SHALL 以 `matchMedia('(orientation: portrait)')` 偵測方向。在觸控裝置且為直向時，SHALL 顯示覆蓋整頁的半透明「請將手機轉為橫向」提示層，並暫停輸入（統一輸入狀態全部視為未按下，且清空所有觸控 `pressed`）。轉為橫向時 SHALL 自動隱藏提示並恢復輸入。系統 MUST NOT 依賴 `screen.orientation.lock` 作為主要手段。

#### Scenario: 直向顯示提示並暫停輸入
- **WHEN** 觸控裝置以直向持握
- **THEN** 顯示「請將手機轉為橫向」提示層，且此時任何觸控不會操作遊戲角色

#### Scenario: 轉為橫向自動恢復
- **WHEN** 使用者將直向的裝置旋轉為橫向
- **THEN** 提示層自動消失，觸控面板可正常操作遊戲

#### Scenario: 桌面不受方向偵測影響
- **WHEN** 桌面瀏覽器以任意視窗比例開啟
- **THEN** 不顯示橫向提示，遊戲照常運作
