# Project 02：布林算術與 ALU

計算機結構作業 02，使用 Nand2Tetris 的 HDL 語言實作加法器系列與算術邏輯單元（ALU）。由 1 位元加法器開始，逐步建構出 16 位元加法器與可執行 18 種運算的 ALU。每題附 `.tst` 測試腳本與 `.cmp` 期望輸出，可用 Nand2Tetris 的 Hardware Simulator 驗證。

## 快速總覽

| 晶片 | 說明 | 輸入 | 輸出 |
|------|------|------|------|
| HalfAdder.hdl | 半加器 | a, b | sum, carry |
| FullAdder.hdl | 全加器 | a, b, c | sum, carry |
| Add16.hdl | 16-bit 加法器 | a[16], b[16] | out[16] |
| Inc16.hdl | 16-bit 加一器 | in[16] | out[16] |
| ALU.hdl | 算術邏輯單元 | x[16], y[16], zx, nx, zy, ny, f, no | out[16], zr, ng |

## 實作順序（相依關係）

```
[ Project 01 基礎邏輯閘 ] (And, Xor, Or, Mux16, Not16, And16, Or8Way ...)
             │
             ▼
    1. HalfAdder (半加器)
             │
             ▼
    2. FullAdder (全加器)
             │
             ▼
    3. Add16 (16 位元加法器)
       ┌──────┴──────┐
       ▼             ▼
  4. Inc16      5. ALU (算術邏輯單元)
```

高層晶片會呼叫底層晶片，若底層尚未實作，模擬器會讀到空的模板而輸出全 0 導致測試失敗，因此務必依序完成。

---

## 逐題解說

### 1. HalfAdder.hdl — 半加器
計算兩個 1-bit 輸入 a、b 的相加：
- **sum**：兩輸入相異時為 1，相同時為 0，即 `Xor(a, b)`。
- **carry**：只有當 a、b 同時為 1 時才進位，即 `And(a, b)`。

```hdl
PARTS:
    Xor(a=a, b=b, out=sum);
    And(a=a, b=b, out=carry);
```

### 2. FullAdder.hdl — 全加器
計算三個 1-bit 輸入（a、b 與前一級進位 c）的相加。用兩個 HalfAdder 串接：先算 a+b 得局部總和與進位，再將局部總和與 c 相加得最終的 sum；兩次加法只要有任一次進位，carry 即為 1。

```hdl
PARTS:
    HalfAdder(a=a, b=b, sum=s1, carry=c1);
    HalfAdder(a=s1, b=c, sum=sum, carry=c2);
    Or(a=c1, b=c2, out=carry);
```

### 3. Add16.hdl — 16 位元加法器
採用**行波進位（Ripple-Carry）**架構，將兩個 16-bit 二補數逐位元相加：
- 第 0 位沒有前一級進位，用 HalfAdder。
- 第 1~15 位各用一個 FullAdder，把上一級的進位逐級往上傳。
- 第 15 位的溢位進位依規格直接忽略。

```hdl
PARTS:
    HalfAdder(a=a[0], b=b[0], sum=out[0], carry=c0);

    FullAdder(a=a[1],  b=b[1],  c=c0,  sum=out[1],  carry=c1);
    // ... 依此類推 ...
    FullAdder(a=a[15], b=b[15], c=c14, sum=out[15], carry=drop);
```

### 4. Inc16.hdl — 16 位元加一器
直接複用 Add16，將輸入 in 加上常數 1。在 HDL 中指定 `b[0]=true`，其餘未指定的位元自動補 0（false）。

```hdl
PARTS:
    Add16(a=in, b[0]=true, out=out);
```

### 5. ALU.hdl — 算術邏輯單元
Hack CPU 的運算中樞，由 6 個控制位元（zx, nx, zy, ny, f, no）操控兩組 16-bit 輸入，計算 18 種功能並輸出狀態旗標。流程分五階段：

1. **輸入前處理 (zx, nx / zy, ny)**：用 `Mux16` 決定是否清零，用 `Not16 + Mux16` 決定是否取反。
2. **核心運算 (f)**：同時計算 `Add16` 與 `And16`，由 `Mux16` 依 f 挑選輸出（f=1 選加法，f=0 選 AND）。
3. **輸出後處理 (no)**：若 no=1 用 `Not16` 將結果取反。
4. **負數旗標 (ng)**：二補數中最高位 out[15] 即正負號，直接拉線輸出。
5. **零值旗標 (zr)**：將 16-bit 輸出拆成 `out[0..7]` 與 `out[8..15]` 兩段，各接 `Or8Way` 再 `Or` 合併，全部為 0 時經 `Not` 反轉得 zr=1。

```hdl
PARTS:
    // --- 1. 處理 X / Y 輸入（清零 + 取反）---
    Mux16(a=x, b=false, sel=zx, out=xZeroed);
    Not16(in=xZeroed, out=notX);
    Mux16(a=xZeroed, b=notX, sel=nx, out=xFinal);

    Mux16(a=y, b=false, sel=zy, out=yZeroed);
    Not16(in=yZeroed, out=notY);
    Mux16(a=yZeroed, b=notY, sel=ny, out=yFinal);

    // --- 2. 核心運算：加法 or 逐位元 AND ---
    Add16(a=xFinal, b=yFinal, out=addOut);
    And16(a=xFinal, b=yFinal, out=andOut);
    Mux16(a=andOut, b=addOut, sel=f, out=fOut);

    // --- 3. 輸出取反與分流 ---
    Not16(in=fOut, out=notFOut);
    Mux16(a=fOut, b=notFOut, sel=no,
          out=out,
          out[15]=ng,
          out[0..7]=lowHalf,
          out[8..15]=highHalf);

    // --- 4. 零值判斷 ---
    Or8Way(in=lowHalf, out=orLow);
    Or8Way(in=highHalf, out=orHigh);
    Or(a=orLow, b=orHigh, out=orAll);
    Not(in=orAll, out=zr);
```

---

## 重點設計

- **加法器系列**：HalfAdder → FullAdder → Add16 → Inc16，進位逐級傳遞（Ripple-Carry）。
- **ALU 控制流程**：先處理輸入（清零/取反），再做核心運算（加法/AND），最後處理輸出與旗標。
- **旗標輸出**：`ng` 直接取最高位；`zr` 用 Or8Way + Or + Not 判斷「全 0」。
- **HDL 技巧**：匯流排分流（`out[15]`、`out[0..7]`、`out[8..15]`）與未指定位元自動補 0（`b[0]=true`）。