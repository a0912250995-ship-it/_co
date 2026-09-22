# Project 01：布林邏輯基礎閘

計算機結構作業 01，使用 Nand2Tetris 的 HDL 語言實作基本邏輯閘。所有晶片皆由 `Nand` 閘開始，逐步建構而成。每題附 `.tst` 測試腳本與 `.cmp` 期望輸出，可用 Nand2Tetris 的 Hardware Simulator 驗證。

## 快速總覽

| 晶片 | 說明 | 輸入 | 輸出 |
|------|------|------|------|
| Not.hdl | 反相器 | in | out |
| And.hdl | 且閘 | a, b | out |
| Or.hdl | 或閘 | a, b | out |
| Xor.hdl | 互斥或 | a, b | out |
| Not16.hdl | 16-bit 反相器 | in[16] | out[16] |
| And16.hdl | 16-bit 且閘 | a[16], b[16] | out[16] |
| Or16.hdl | 16-bit 或閘 | a[16], b[16] | out[16] |
| Or8Way.hdl | 8 輸入或閘 | in[8] | out |
| Mux.hdl | 多工器 | a, b, sel | out |
| Mux16.hdl | 16-bit 多工器 | a[16], b[16], sel | out[16] |
| Mux4Way16.hdl | 4 路 16-bit 多工器 | a..d[16], sel[2] | out[16] |
| Mux8Way16.hdl | 8 路 16-bit 多工器 | a..h[16], sel[3] | out[16] |
| DMux.hdl | 解多工器 | in, sel | a, b |
| DMux4Way.hdl | 4 路解多工器 | in, sel[2] | a, b, c, d |
| DMux8Way.hdl | 8 路解多工器 | in, sel[3] | a..h |

---

## 逐題解說

### 1. Not.hdl — 反相器
`out = not in`。因為基礎閘只有 Nand，把兩個輸入接在一起即可：`Nand(a=in, b=in)` 恰好就是 `not (in and in) = not in`。

```hdl
PARTS:
    Nand(a=in, b=in, out=out);
```

### 2. And.hdl — 且閘
`out = 1 if (a == 1 and b == 1)`。直接用「Nand 再接一個 Not」：「先 Nand（等於 not-and），再反一次就回到 and」。

```hdl
PARTS:
    Nand(a=a, b=b, out=nandOut);
    Not(in=nandOut, out=out);
```

### 3. Or.hdl — 或閘
`out = 1 if (a == 1 or b == 1)`。利用迪摩根定律 `a or b = not(not a and not b)`：先把 a、b 各自反相，再對兩個反相結果做 Nand（即「兩個假的都假」時才為 0）。

```hdl
PARTS:
    Not(in=a, out=notA);
    Not(in=b, out=notB);
    Nand(a=notA, b=notB, out=out);
```

### 4. Xor.hdl — 互斥或閘
`out = 1 if a != b`。剛好一個高一個低：組合 `a and not b` 或 `not a and b`，再把兩者 OR 起來。

```hdl
PARTS:
    Not(in=a, out=notA);
    Not(in=b, out=notB);
    And(a=a, b=notB, out=w1);
    And(a=notA, b=b, out=w2);
    Or(a=w1, b=w2, out=out);
```

### 5. Not16.hdl — 16-bit 反相器
把單 bit 的 Not 平行展開 16 份，每個 bit 各自反相。這是所有「16」版晶片共同的手法：**逐位元平行處理**。

```hdl
PARTS:
    Not(in=in[0],  out=out[0]);
    Not(in=in[1],  out=out[1]);
    // ... 一直到
    Not(in=in[15], out=out[15]);
```

### 6. And16.hdl — 16-bit 且閘
對 16 個 bit 逐位元做 And，`out[i] = a[i] and b[i]`。

```hdl
PARTS:
    And(a=a[0], b=b[0], out=out[0]);
    And(a=a[1], b=b[1], out=out[1]);
    // ... 一直到 out[15]
```

### 7. Or16.hdl — 16-bit 或閘
對 16 個 bit 逐位元做 Or，`out[i] = a[i] or b[i]`。

```hdl
PARTS:
    Or(a=a[0], b=b[0], out=out[0]);
    Or(a=a[1], b=b[1], out=out[1]);
    // ... 一直到 out[15]
```

### 8. Or8Way.hdl — 8 輸入或閘
`out = in[0] or in[1] or ... or in[7]`，只要任一個輸入為 1 就輸出 1。把 8 個輸入兩兩串接成 OR 鏈。

```hdl
PARTS:
    Or(a=in[0], b=in[1], out=w1);
    Or(a=w1,    b=in[2], out=w2);
    Or(a=w2,    b=in[3], out=w3);
    Or(a=w3,    b=in[4], out=w4);
    Or(a=w4,    b=in[5], out=w5);
    Or(a=w5,    b=in[6], out=w6);
    Or(a=w6,    b=in[7], out=out);
```

### 9. Mux.hdl — 多工器
`sel == 0` 輸出 a，`sel == 1` 輸出 b。做法：`a and not sel` 與 `b and sel` 兩路，最後 OR——sel 為 0 時只放行 a，為 1 時只放行 b，永遠只會有一個通過。

```hdl
PARTS:
    Not(in=sel, out=notSel);
    And(a=a, b=notSel, out=aAndNotSel);
    And(a=b, b=sel, out=bAndSel);
    Or(a=aAndNotSel, b=bAndSel, out=out);
```

### 10. Mux16.hdl — 16-bit 多工器
一個共享的 sel 控制 16 道 Mux 平行工作，`out[i] = a[i] if sel == 0 else b[i]`。

```hdl
PARTS:
    Mux(a=a[0], b=b[0], sel=sel, out=out[0]);
    Mux(a=a[1], b=b[1], sel=sel, out=out[1]);
    // ... 一直到 out[15]
```

### 11. Mux4Way16.hdl — 4 路 16-bit 多工器
`sel[2]` 選出 a/b/c/d 其中一路。**分層選擇**：先用 `sel[0]` 兩兩合併（ab、cd），再用 `sel[1]` 從兩組中挑最終輸出。

```hdl
PARTS:
    Mux16(a=a, b=b, sel=sel[0], out=ab);
    Mux16(a=c, b=d, sel=sel[0], out=cd);
    Mux16(a=ab, b=cd, sel=sel[1], out=out);
```

### 12. Mux8Way16.hdl — 8 路 16-bit 多工器
`sel[3]` 選出 a~h 其中一路。同樣分層：先用 `sel[0..1]` 把 8 路收成兩組 4 路結果，再用 `sel[2]` 決定最終輸出。

```hdl
PARTS:
    Mux4Way16(a=a, b=b, c=c, d=d, sel=sel[0..1], out=abcd);
    Mux4Way16(a=e, b=f, c=g, d=h, sel=sel[0..1], out=efgh);
    Mux16(a=abcd, b=efgh, sel=sel[2], out=out);
```

### 13. DMux.hdl — 解多工器
多工器的反向：一個輸入依 sel 送到 a 或 b。`sel == 0` 輸入進 a（b 為 0），`sel == 1` 輸入進 b。

```hdl
PARTS:
    Not(in=sel, out=notSel);
    And(a=in, b=notSel, out=a);
    And(a=in, b=sel, out=b);
```

### 14. DMux4Way.hdl — 4 路解多工器
`sel[2]` 決定輸入送往 a/b/c/d 哪一個，其餘輸出 0。**分層路由**：先用 `sel[1]` 切成兩路（上半/下半），再用 `sel[0]` 從兩路中決定具體輸出端。

```hdl
PARTS:
    DMux(in=in, sel=sel[1], a=ab, b=cd);
    DMux(in=ab, sel=sel[0], a=a, b=b);
    DMux(in=cd, sel=sel[0], a=c, b=d);
```

### 15. DMux8Way.hdl — 8 路解多工器
`sel[3]` 決定輸入送往 a~h 哪一個。先用 `sel[2]` 分成兩半，再用兩個 DMux4Way（以 `sel[0..1]`）精準送到輸出端。

```hdl
PARTS:
    DMux(in=in, sel=sel[2], a=abcd, b=efgh);
    DMux4Way(in=abcd, sel=sel[0..1], a=a, b=b, c=c, d=d);
    DMux4Way(in=efgh, sel=sel[0..1], a=e, b=f, c=g, d=h);
```

---

## 重點設計

- **基本邏輯**：Not、And、Or、Xor 全部由 Nand 閘組合而成。
- **多位元電路**：16-bit 版本把單 bit 閘逐位元平行展開 16 次。
- **線路選擇**：Mux 依 sel 選擇輸入、DMux 依 sel 決定輸出端。
- **多路版本**：Mux4Way16/Mux8Way16 與 DMux4Way/DMux8Way 都採「分層路由」，把 sel 分段使用，用較小單元遞迴組成。