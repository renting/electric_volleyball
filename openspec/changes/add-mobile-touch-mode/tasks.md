## 1. 裝置偵測與輸入抽象基礎

- [x] 1.1 在 `electric_volleyball.html` script 頂端新增 `isTouchDevice()` 工具函式（`matchMedia('(pointer: coarse)').matches || 'ontouchstart' in window`），與既有常數放一起。
- [x] 1.2 新增模組級的觸控輸入狀態物件 `touchInput`（含 `left/right/up/hit` 各為布林），預設全 `false`，供 GameScene 每幀讀取。
- [x] 1.3 在 `GameScene.create()` 既有 `this.kbd` 之後，建立統一輸入狀態組裝函式：將鍵盤 Key 物件與 `touchInput` 以 OR 合併為 `{left:{isDown}, right:{isDown}, up:{isDown}, hit:{isDown}}`。（實作為 `GameScene._p1Input()` 方法）

## 2. GameScene 輸入路徑改接統一輸入狀態

- [x] 2.1 修改 `GameScene.update()` 控制段：1P 模式改以「鍵盤 OR 觸控」合併後的統一輸入狀態呼叫 `_controlHuman(this.players[0], ...)`，移除直接傳 Key 物件的寫法。
- [x] 2.2 確認 `_controlHuman` 內三向分流（強殺／撲球／揮空）與所有玩法數值（`SPD=5.4`、`vy=-13.5` 等）一字未改。（僅於其上方新增 `_p1Input()`，未動其內容）
- [ ] 2.3 桌面鍵盤回歸驗證：以鍵盤完整打一局（移動、跳躍、強殺、撲球、暫停、重開），行為與改動前一致。（需瀏覽器實測；邏輯上桌面 `touchInput` 恆為 `false`，OR 合併為恆等，行為等價）

## 3. 觸控操作面板（虛擬搖桿 + 動作鈕，HTML / CSS overlay）

- [x] 3.1 在 `#wrap` 內新增觸控面板 DOM：左側虛擬搖桿（`#stick` 底座 + `#stickKnob` 旋鈕）與右下動作鈕（扣殺/撲球 `hit`），預設以 class 隱藏。
- [x] 3.2 撰寫面板 CSS：搖桿底座/旋鈕與動作鈕為絕對定位半透明圓形、`touch-action:none`、`user-select:none`，並以 `env(safe-area-inset-*)` 避開瀏海/圓角。
- [x] 3.3 實作搖桿邏輯：以 Pointer Events + `setPointerCapture` 追蹤單一手指，旋鈕偏移過死區門檻映射為 `touchInput.left/right/up`（上推=跳躍）；放開/`pointercancel` 歸位並清空。動作鈕綁定 `touchstart`/`pointerdown`→`hit=true`、`touchend`/`touchcancel`/`pointerup`/`pointerleave`→`false`，並於 `touchstart` 呼叫 `preventDefault()`。
- [x] 3.4 僅在 `isTouchDevice()` 為真時移除隱藏 class 顯示面板；桌面維持不顯示。
- [ ] 3.5 驗證多點觸控：搖桿推方向 + 扣殺鈕可觸發移動中扣殺與「方向 + 扣殺」撲球；搖桿在死區內視為無方向、不撲球。（需觸控裝置實測）
- [ ] 3.6 驗證放開搖桿與 `pointercancel` 後旋鈕歸位、不卡鍵。（需觸控裝置實測）

## 4. 行動版面與 viewport（RWD）

- [x] 4.1 修改 `<head>` 的 `<meta viewport>` 為 `width=device-width, initial-scale=1.0, viewport-fit=cover, user-scalable=no`。
- [x] 4.2 新增觸控裝置橫向時的 CSS：隱藏 `<h1>` 與 `#hint`，讓 `#wrap` 在 `100vw × 100vh` 內最大化；桌面 CSS（`width:min(96vw,920px)` 等）維持不變。
- [ ] 4.3 驗證手機橫向時遊戲畫布最大化且 `Scale.FIT` 不裁切內容；桌面外觀無變化。（需瀏覽器/裝置實測）

## 5. 橫向閘門與直向提示

- [x] 5.1 新增覆蓋整頁的「請將手機轉為橫向」提示層 DOM 與 CSS（半透明、置中、預設隱藏）。
- [x] 5.2 以 `matchMedia('(orientation: portrait)')` 監聽方向變化；觸控裝置且直向時顯示提示層。
- [x] 5.3 直向時暫停輸入：強制清空所有 `touchInput.*` 為 `false`，使統一輸入狀態全部視為未按下。
- [x] 5.4 轉為橫向時自動隱藏提示並恢復輸入；桌面任意視窗比例皆不顯示提示。

## 6. MenuScene 觸控選單

- [x] 6.1 `MenuScene.create()` 依 `isTouchDevice()` 分支：觸控裝置僅渲染「普通」「困難」兩個可點選項，隱藏 2P 雙人選項與鍵盤說明，提示文字改為觸控導向。
- [x] 6.2 為觸控選項設定對應命中區並綁定點擊：「普通」→ `_start('1P','normal')`、「困難」→ `_start('1P','hard')`，命中區對應顯示文字範圍避免誤觸。
- [x] 6.3 桌面維持 1/2/3 鍵與「點任意處 → 1P 普通」既有捷徑不變。
- [ ] 6.4 驗證觸控裝置選單僅出現單人兩選項且各自啟動正確模式；桌面三選項與捷徑正常。（需瀏覽器/裝置實測）

## 7. 整合驗證與收尾

- [ ] 7.1 真機（或瀏覽器裝置模擬，含 iOS Safari）橫向實測：選單 → 遊玩 → 暫停 → 結束 → 返回選單全流程可用。（需裝置實測）
- [ ] 7.2 確認觸控操作時頁面不捲動、不縮放、不彈長按選單。（需觸控裝置實測）
- [ ] 7.3 桌面鍵盤體驗最終回歸確認：外觀、玩法數值、物理、AI 行為與改動前完全一致。（需瀏覽器實測）
- [x] 7.4 執行 `openspec validate "add-mobile-touch-mode"` 確認規格與變更一致。

## 8. 視覺強化（球體放大與扣殺彗星殘影）

- [x] 8.1 將 `this.ball` 的 `r` 由 17 改為 22；確認碰撞、網子、落地、繪製皆沿用 `ball.r`，物理一致。
- [x] 8.2 為 `ball` 新增 `trail`（軌跡陣列，上限 16）與 `smash`（殘影強度計時器）欄位；於 `_resetRally` 清空兩者。
- [x] 8.3 在 `_tryPowerHit` 成功命中時設定 `ball.smash`（觸發彗星殘影）；於物理更新每幀記錄球心至 `trail` 並遞減 `smash`。
- [x] 8.4 在繪製球體前依強度繪製漸層淡出殘影：`smash>0` 時黃→橘彗星漸層最明顯、高速時較淡、低速不顯示，且殘影位於球體之下不遮蓋。
- [ ] 8.5 驗證：扣殺時出現明顯彗星拖尾、低速無殘影、回合重置不殘留；桌面與手機外觀皆正常。（需瀏覽器/裝置實測）
