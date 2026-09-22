# Project 01：布林邏輯基礎閘

計算機結構作業 01，使用 Nand2Tetris 的 HDL 語言實作基本邏輯閘。所有晶片皆由 `Nand` 閘開始，逐步建構而成。

## 檔案說明

| 晶片 | 說明 | 輸入 | 輸出 |
|------|------|------|------|
| Not.hdl | 反相器：out = not in | in | out |
| And.hdl | 且閘：out = 1 if (a == 1 and b == 1) | a, b | out |
| Or.hdl | 或閘：out = 1 if (a == 1 or b == 1) | a, b | out |
| Xor.hdl | 互斥或：out = 1 if a != b | a, b | out |
| Not16.hdl | 16-bit 反相器 | in[16] | out[16] |
| And16.hdl | 16-bit 且閘 | a[16], b[16] | out[16] |
| Or16.hdl | 16-bit 或閘 | a[16], b[16] | out[16] |
| Or8Way.hdl | 8 輸入或閘 | in[8] | out |
| Mux.hdl | 多工器：sel == 0 選 a，否則選 b | a, b, sel | out |
| Mux16.hdl | 16-bit 多工器 | a[16], b[16], sel | out[16] |
| Mux4Way16.hdl | 4 路 16-bit 多工器 | a[16], b[16], c[16], d[16], sel[2] | out[16] |
| Mux8Way16.hdl | 8 路 16-bit 多工器 | a..h[16], sel[3] | out[16] |
| DMux.hdl | 解多工器：依 sel 路由到 a 或 b | in, sel | a, b |
| DMux4Way.hdl | 4 路解多工器 | in, sel[2] | a, b, c, d |
| DMux8Way.hdl | 8 路解多工器 | in, sel[3] | a..h[16] |

## 重點設計

- **基本邏輯**：Not、And、Or、Xor 直接由 Nand 閘組合。
- **多位元電路**：16-bit 的閘將單 bit 版本平行展開 16 次。
- **線路選擇**：Mux 依 sel 選擇輸入，DMux 依 sel 決定輸出端。
- **多路版本**：Mux4Way16/Mux8Way16 以較小單元遞迴組成，sel 分段選取。

每個晶片都附有 `.tst` 測試腳本與 `.cmp` 期望輸出，可用 Nand2Tetris 的 Hardware Simulator 驗證。