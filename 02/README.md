Skip to main content
ALU.hdl Implementation for Nand2Tetris
1,081 3,672 3,673 3,673 6,431 9,006 9,009 9,010 9,012 26,139 tokens

Model 9:37 AM
ThinkingThoughts
Expand to view model thoughts

chevron_right
Here is the complete implementation for ALU.hdl from Project 2 of Nand to Tetris.
Complete HDL Code
code
Hdl
// This file is part of www.nand2tetris.org
// and the book "The Elements of Computing Systems"
// by Nisan and Schocken, MIT Press.
// File name: projects/02/ALU.hdl

/**
 * The ALU (Arithmetic Logic Unit).
 * Computes one of the following functions:
 * x+y, x-y, y-x, 0, 1, -1, x, y, -x, -y, !x, !y,
 * x+1, y+1, x-1, y-1, x&y, x|y on two 16-bit inputs, 
 * according to 6 input bits denoted zx,nx,zy,ny,f,no.
 * In addition, the ALU computes two 1-bit outputs:
 * if the ALU output == 0, zr is set to 1; otherwise zr is set to 0;
 * if the ALU output < 0, ng is set to 1; otherwise ng is set to 0.
 */

CHIP ALU {
    IN  
        x[16], y[16],  // 16-bit inputs        
        zx, // zero the x input?
        nx, // negate the x input?
        zy, // zero the y input?
        ny, // negate the y input?
        f,  // compute out = x + y (if 1) or x & y (if 0)
        no; // negate the out output?

    OUT 
        out[16], // 16-bit output
        zr, // 1 if (out == 0), 0 otherwise
        ng; // 1 if (out < 0),  0 otherwise

    PARTS:
    // --- 1. Process X input (zx, nx) ---
    // If zx == 1, set x = 0
    Mux16(a=x, b=false, sel=zx, out=xZeroed);
    // If nx == 1, set x = !x
    Not16(in=xZeroed, out=notX);
    Mux16(a=xZeroed, b=notX, sel=nx, out=xFinal);

    // --- 2. Process Y input (zy, ny) ---
    // If zy == 1, set y = 0
    Mux16(a=y, b=false, sel=zy, out=yZeroed);
    // If ny == 1, set y = !y
    Not16(in=yZeroed, out=notY);
    Mux16(a=yZeroed, b=notY, sel=ny, out=yFinal);

    // --- 3. Compute function f (f == 1 -> Add, f == 0 -> And) ---
    Add16(a=xFinal, b=yFinal, out=addOut);
    And16(a=xFinal, b=yFinal, out=andOut);
    Mux16(a=andOut, b=addOut, sel=f, out=fOut);

    // --- 4. Negate output (no) and split buses for flags ---
    Not16(in=fOut, out=notFOut);
    // Split the 16-bit output into two 8-bit halves for zr, and bit 15 for ng:
    Mux16(a=fOut, b=notFOut, sel=no, 
          out=out, 
          out[15]=ng, 
          out[0..7]=lowHalf, 
          out[8..15]=highHalf);

    // --- 5. Determine if out == 0 (zr) ---
    // If any bit is 1 in either half, orAll will be 1
    Or8Way(in=lowHalf, out=orLow);
    Or8Way(in=highHalf, out=orHigh);
    Or(a=orLow, b=orHigh, out=orAll);
    // zr = 1 only if all bits are 0
    Not(in=orAll, out=zr);
}
How It Wor
Input Pre-processing (zx, nx, zy, ny):
Mux16 selectsfalse) using the zeroingzx/zy).
The result is invNot16,Mux16 selects the normal or inverted signal bnx/ny.
ALU Operation (f):
Both addAdd16) andAnd16) are calculated sim
Mux16 selects the adf == 1,f == 0.
Output Post-processing (no):
Not16 inverts the outpu
Mux16 outputs eithno.
Flag Generation:
ng (Negative flag): In 2's complemout[15]) s1. This is tMux16.
zr (Zero flag): The 16-bit outpuout[0..7] and `out[8..15out[8..15]). Using two `Or8WOr8Way gates and a stanOr gate, we cNot yields 1 only if t0.
Citations

http://blog.tojiru.net/article/426464326.html
https://programmermagazine.github.io/mag/ymag201506/book.html
https://github.com/JackDraak/NAND2Teris
Model 9:38 AM
ThinkingThoughts
Expand to view model thoughts

chevron_right
以ALU.hdl 程式碼
code
Hd
// 此檔案為 www.nand2tetris.org 與
// 書籍《The Elements of Computing Systems》（MIT Press 出版）的一部分。
// 檔名：projects/02/ALU.hdl

/**
 * 算術邏輯單元（ALU, Arithmetic Logic Unit）。
 * 根據 6 個控制位元（zx, nx, zy, ny, f, no）對兩個 16 位元輸入（x, y）
 * 計算下列功能之一：
 * x+y, x-y, y-x, 0, 1, -1, x, y, -x, -y, !x, !y,
 * x+1, y+1, x-1, y-1, x&y, x|y。
 *
 * 此外，ALU 還會計算出兩個 1 位元的狀態標誌（flags）：
 * 如果輸出 == 0，則 zr = 1；否則 zr = 0。
 * 如果輸出 < 0，則 ng = 1；否則 ng = 0。
 */

CHIP ALU {
    IN  
        x[16], y[16],  // 16 位元輸入        
        zx, // 是否將 x 輸入清零？
        nx, // 是否將 x 輸入取反（NOT）？
        zy, // 是否將 y 輸入清零？
        ny, // 是否將 y 輸入取反（NOT）？
        f,  // 運算選擇：1 代表加法 (x + y)，0 代表逐位元及運算 (x & y)
        no; // 是否將輸出結果取反（NOT）？

    OUT 
        out[16], // 16 位元輸出
        zr,      // 若 out == 0 則為 1，否則為 0
        ng;      // 若 out < 0  則為 1，否則為 0

    PARTS:
    // --- 1. 處理 X 輸入（zx, nx）---
    // 若 zx == 1，將 x 設為 0；否則保持原本的 x
    Mux16(a=x, b=false, sel=zx, out=xZeroed);
    // 計算非（NOT）值
    Not16(in=xZeroed, out=notX);
    // 若 nx == 1，將 x 取反；否則保持清零後的結果
    Mux16(a=xZeroed, b=notX, sel=nx, out=xFinal);

    // --- 2. 處理 Y 輸入（zy, ny）---
    // 若 zy == 1，將 y 設為 0；否則保持原本的 y
    Mux16(a=y, b=false, sel=zy, out=yZeroed);
    // 計算非（NOT）值
    Not16(in=yZeroed, out=notY);
    // 若 ny == 1，將 y 取反；否則保持清零後的結果
    Mux16(a=yZeroed, b=notY, sel=ny, out=yFinal);

    // --- 3. 核心運算（f）---
    // 同時計算加法與逐位元 AND
    Add16(a=xFinal, b=yFinal, out=addOut);
    And16(a=xFinal, b=yFinal, out=andOut);
    // 若 f == 1 選擇加法結果，若 f == 0 選擇 AND 結果
    Mux16(a=andOut, b=addOut, sel=f, out=fOut);

    // --- 4. 輸出取反（no）與分流拉線 ---
    // 計算取反後的結果
    Not16(in=fOut, out=notFOut);
    // 根據 no 決定輸出，並同時分流出：
    // - 最終輸出 out
    // - 最高有效位元 out[15]（作為負數標誌 ng）
    // - 低 8 位元 out[0..7] 與高 8 位元 out[8..15]（用於判斷是否全為 0）
    Mux16(a=fOut, b=notFOut, sel=no, 
          out=out, 
          out[15]=ng, 
          out[0..7]=lowHalf, 
          out[8..15]=highHalf);

    // --- 5. 判斷輸出是否為零（zr 標誌）---
    // 分別檢查低 8 位元與高 8 位元是否有任何一個位元為 1
    Or8Way(in=lowHalf, out=orLow);
    Or8Way(in=highHalf, out=orHigh);
    // 將兩個 8 位元的結果合併 OR 運算
    Or(a=orLow, b=orHigh, out=orAll);
    // 若 16 個位元全部皆為 0（orAll 為 0），取 NOT 後 zr 才會為 1
    Not(in=orAll, out=zr);
}
Model 9:42 AM
ThinkingThoughts
Expand to view model thoughts

chevron_right
Model 9:43 AM
在Project 2（布林運算與加法器） 中
建議依照下列順序實
推薦實作與測試
code
Code
1. HalfAdder (半加器)
      ↓
2. FullAdder (全加器)
      ↓
3. Add16 (16位元加法器)
      ↓
4. Inc16 (16位元加一器)
      ↓
5. ALU-nostat (不帶狀態標誌的 ALU，選填)
      ↓
6. ALU (完整 ALU)
每個階段的相
1. **`HalfAddHalfAdder.hdl（最基礎）
相依晶片：使用Xor, `AnAnd）。
功能：sum 與進位 carry。
驗證：在模擬器載入 HalfAdder.tst 跑測
2. *FullAdder.hdl（全加器）
相依晶片：使用你剛寫好的 **`HHalfAdder（通常用
功能：計算 3 個 1-bit（
c
c
）的加法
驗證：在模擬器FullAdder.tst 跑測試。
3. **`Add16.hdlAdd16.hdl（16 位元加法器）
相依晶片：串接 16FullAdder（或第 0 位用 `HaHalfAdder，後續用 15FullAdder）
功能：計算兩個
驗證：在模擬器載入Add16.tst 跑測試這步非常關鍵，ALU 的加法運算完全依賴它！）
4. Inc16.hdl（加一器）
相依晶片：使用剛寫Add16（
i
n
in
 與 
1
1
 相加）。
功能：輸
i
n
+
1
in+1
。
驗證：在模擬器載入 `IInc16.tst 跑測試。
5. **`ALUALU-nostat.tst（選填，輔助測試）
這是官方簡化版測試腳本，只驗證輸出數值 out，不會檢查狀態旗標 `zzr 和 ng。若你的 A
6. ALU.hdl（最
相依晶片：使用 Project 1 的Mux16, `Not1Not16, And16, `OOr8Way, Or, Not，以及這單元的 **`Add16。
驗證：在模擬器載入 `ALUALU.tst，比對 ALU.cmp。
為什麼你剛才 ALU 會報錯？
因為你剛才直接跳到了第 6ALU.hdl），而同HalfAdder、FullAdder、Add16 都還是空的模板、Add16 時無法算出正確的
請依序從 **HalfAdder ➜ FullAdder ➜ Add16 ➜ `F.tst 測試，最後ALU.tst 就
Model 9:44 AM
這是順序中的第一步：HalfAdder.hdl（半加器）。
原理說明
半加器負責將兩個 1 位元的輸入 
a
a
 與 
b
b
 相加：
sum（總和位元）：只有當 
a
a
 與 
b
b
 恰好有一個為 1 時，結果才是 1（即 
0
+
1
=
1
,
1
+
0
=
1
0+1=1,1+0=1
；若 
0
+
0
=
0
,
1
+
1
=
0
0+0=0,1+1=0
）。這正是 Xor（互斥或） 邏輯。
carry（進位位元）：只有當 
a
a
 與 
b
b
 同時為 1 時才會產生進位（即 
1
+
1
=
10
(
2
)
1+1=10 
(2)
​
 
，進位為 1）。這正是 And（及運算） 邏輯。
完整程式碼（附中文註解）
code
Hdl
// 此檔案為 www.nand2tetris.org 與
// 書籍《The Elements of Computing Systems》（MIT Press 出版）的一部分。
// 檔名：projects/02/HalfAdder.hdl

/**
 * 計算兩個 1 位元的總和。
 */

CHIP HalfAdder {
    IN a, b;    // 1 位元輸入
    OUT sum,    // 加法結果的右側低位 (LSB)
        carry;  // 加法結果的左側進位 (MSB)

    PARTS:
    // 當 a 與 b 只有一個是 1 時，sum = 1 (使用 Xor)
    Xor(a=a, b=b, out=sum);

    // 當 a 與 b 同時為 1 時產生進位，carry = 1 (使用 And)
    And(a=a, b=b, out=carry);
}
下一步
存檔後，在硬體模擬器中載入 HalfAdder.tst 執行測試，確認成功後就可以接著寫第二步 FullAdder.hdl！
Model 9:45 AM
以下是保留最上方 3 行英文版權宣告，內部說明為繁體中文註解的 HalfAdder.hdl：
code
Hdl
// This file is part of www.nand2tetris.org
// and the book "The Elements of Computing Systems"
// by Nisan and Schocken, MIT Press.
// File name: projects/02/HalfAdder.hdl

/**
 * 計算兩個 1 位元的總和。
 */

CHIP HalfAdder {
    IN a, b;    // 1 位元輸入
    OUT sum,    // 加法結果的右側低位 (LSB)
        carry;  // 加法結果的左側進位 (MSB)

    PARTS:
    // 當 a 與 b 只有一個是 1 時，sum = 1 (使用 Xor)
    Xor(a=a, b=b, out=sum);

    // 當 a 與 b 同時為 1 時產生進位，carry = 1 (使用 And)
    And(a=a, b=b, out=carry);
}
Model 9:49 AM
這是順序中的第二步：FullAdder.hdl（全加器）。
原理說明
全加器負責將三個 1 位元的輸入 
a
,
b
,
c
a,b,c
（
c
c
 通常為前一位的進位）相加：
先用第一個 HalfAdder 將 
a
a
 與 
b
b
 相加，得到暫存的總和 s1 與進位 c1。
再用第二個 HalfAdder 將 s1 與 
c
c
 相加，得到最終的總和 sum 與進位 c2。
只要 c1 或 c2 其中一個產生進位，最終的 carry 就是 1（使用 Or 閘連接）。
完整程式碼（保留前 3 行英文，內附中文註解）
code
Hdl
// This file is part of www.nand2tetris.org
// and the book "The Elements of Computing Systems"
// by Nisan and Schocken, MIT Press.
// File name: projects/02/FullAdder.hdl

/**
 * 計算三個 1 位元的總和 (a + b + c)。
 */

CHIP FullAdder {
    IN a, b, c;  // 1 位元輸入
    OUT sum,     // 加法結果的右側低位 (LSB)
        carry;   // 加法結果的左側進位 (MSB)

    PARTS:
    // 1. 將 a 和 b 相加，產生局部總和 s1 與進位 c1
    HalfAdder(a=a, b=b, sum=s1, carry=c1);

    // 2. 將 s1 和 c 相加，得到最終總和 sum 與第二個進位 c2
    HalfAdder(a=s1, b=c, sum=sum, carry=c2);

    // 3. 兩次相加只要有任何一次產生進位，carry 就為 1
    Or(a=c1, b=c2, out=carry);
}
下一步
存檔後，在硬體模擬器中載入 FullAdder.tst 執行測試，通過後即可進行第三步 Add16.hdl！
Model 9:53 AM
這是順序中的第三步：Add16.hdl（16 位元加法器）。
原理說明
16 位元加法器是將兩個 16-bit 數字（
a
a
 與 
b
b
）逐位元相加：
第 0 位（最右側最低位）：因為沒有前一級的進位，可以使用一個 HalfAdder（或將 FullAdder 的進位輸入 
c
c
 接 false）。
第 1 位到第 15 位：依序使用 FullAdder，將當前的位元 
a
[
i
]
a[i]
、
b
[
i
]
b[i]
 以及上一位傳過來的進位 
c
c
 串接相加。
溢位（第 15 位的進位）：根據規格說明直接忽略不接。
完整程式碼（保留前 3 行英文，內附中文註解）
code
Hdl
// This file is part of www.nand2tetris.org
// and the book "The Elements of Computing Systems"
// by Nisan and Schocken, MIT Press.
// File name: projects/02/Adder16.hdl

/**
 * 將兩個 16 位元數值相加。
 * 忽略最高位的溢位進位 (overflow carry)。
 */

CHIP Add16 {
    IN a[16], b[16];
    OUT out[16];

    PARTS:
    // 第 0 位：沒有前一級進位，使用 HalfAdder
    HalfAdder(a=a[0], b=b[0], sum=out[0], carry=c0);

    // 第 1 到 15 位：依序使用 FullAdder 串接前一位的進位
    FullAdder(a=a[1],  b=b[1],  c=c0,  sum=out[1],  carry=c1);
    FullAdder(a=a[2],  b=b[2],  c=c1,  sum=out[2],  carry=c2);
    FullAdder(a=a[3],  b=b[3],  c=c2,  sum=out[3],  carry=c3);
    FullAdder(a=a[4],  b=b[4],  c=c3,  sum=out[4],  carry=c4);
    FullAdder(a=a[5],  b=b[5],  c=c4,  sum=out[5],  carry=c5);
    FullAdder(a=a[6],  b=b[6],  c=c5,  sum=out[6],  carry=c6);
    FullAdder(a=a[7],  b=b[7],  c=c6,  sum=out[7],  carry=c7);
    FullAdder(a=a[8],  b=b[8],  c=c7,  sum=out[8],  carry=c8);
    FullAdder(a=a[9],  b=b[9],  c=c8,  sum=out[9],  carry=c9);
    FullAdder(a=a[10], b=b[10], c=c9,  sum=out[10], carry=c10);
    FullAdder(a=a[11], b=b[11], c=c10, sum=out[11], carry=c11);
    FullAdder(a=a[12], b=b[12], c=c11, sum=out[12], carry=c12);
    FullAdder(a=a[13], b=b[13], c=c12, sum=out[13], carry=c13);
    FullAdder(a=a[14], b=b[14], c=c13, sum=out[14], carry=c14);
    // 第 15 位的進位不需接出 (直接忽略)
    FullAdder(a=a[15], b=b[15], c=c14, sum=out[15], carry=drop);
}
下一步
存檔後，在硬體模擬器中載入 Add16.tst 執行測試，確認通過！
Add16 通過後，原本報錯的 ALU.tst 就可以重新載入測試了！
Citations

https://blog.csdn.net/qq_17065591/article/details/107029998
Model 9:55 AM
這是順序中的第四步：Inc16.hdl（16 位元加一器）。
原理說明
加一器（Incrementer）的功能是將 16 位元輸入 
i
n
in
 加上數值 
1
1
：
直接調用剛剛寫好的 Add16。
將 
a
a
 接上輸入 in。
將 
b
b
 設為數值 
1
1
：在硬體中就是將最低位（b[0]）設為 true（即 1），其餘所有位元預設皆為 0（在 Nand to Tetris HDL 中，未特別指定的輸入位元會自動為 0 / false）。
完整程式碼（保留前 3 行英文，內附中文註解）
code
Hdl
// This file is part of www.nand2tetris.org
// and the book "The Elements of Computing Systems"
// by Nisan and Schocken, MIT Press.
// File name: projects/02/Inc16.hdl

/**
 * 16 位元加一器 (Incrementer)：
 * out = in + 1 (算術加法)
 */

CHIP Inc16 {
    IN in[16];
    OUT out[16];

    PARTS:
    // 將輸入 in 與 1 (即 b[0]=true，其餘位元自動為 0) 相加
    Add16(a=in, b[0]=true, out=out);
}
下一步
存檔後，在硬體模擬器中載入 Inc16.tst 執行測試！至此加法器系列已全部齊全，你可以直接載入 ALU.tst 進行最終驗證了！
Model 9:59 AM
這裡為你整理好一份完整的 README.md 文件內容。
格式採用標準 Markdown，包含了模組依賴與實作順序圖解、各晶片的原理與逐題詳細解析，你可以直接複製存成專案資料夾下的 README.md。
code
Markdown
# Nand to Tetris - Project 02: 布林算術與 ALU (Boolean Arithmetic & ALU)

本專案實作了從最基礎的加法邏輯閘到計算機運算核心——算術邏輯單元（ALU, Arithmetic Logic Unit）的硬體設計。

---

## 目錄
1. [實作順序與晶片相依關係說明](#一實作順序與晶片相依關係說明)
2. [各晶片設計與原理解析](#二各晶片設計與原理解析)
   - [1. HalfAdder (半加器)](#1-halfadder-半加器)
   - [2. FullAdder (全加器)](#2-fulladder-全加器)
   - [3. Add16 (16 位元加法器)](#3-add16-16-位元加法器)
   - [4. Inc16 (16 位元加一器)](#4-inc16-16-位元加一器)
   - [5. ALU (算術邏輯單元)](#5-alu-算術邏輯單元)
3. [常見問題排查 (FAQ)](#三常見問題排查-faq)

---

## 一、實作順序與晶片相依關係說明

在硬體描述語言（HDL）中，高層晶片往往直接呼叫底層晶片。若底層晶片尚未完成或有錯誤，模擬器將無法載入正確元件，導致高層晶片測試失敗。

### 依賴流程圖：
code
Code
[ Project 01 基礎邏輯閘 ] (And, Xor, Or, Mux, Not...)
             │
             ▼
   1. HalfAdder (半加器)
             │
             ▼
   2. FullAdder (全加器)
             │
             ▼
   3. Add16 (16位元加法器)
      ┌──────┴──────┐
      ▼             ▼
Inc16 (加一器) 5. ALU (算術邏輯單元)
code
Code
### 推薦開發步驟：
1. **HalfAdder (半加器)**：最基礎的單元，僅使用 Project 1 的 `Xor` 與 `And`。
2. **FullAdder (全加器)**：使用 2 個 `HalfAdder` 與 1 個 `Or` 組合而成，具備接收進位輸入（Carry-in）的能力。
3. **Add16 (16 位元加法器)**：串聯 16 個加法單元，將低位元的進位逐級傳遞到高位元（Ripple-carry）。
4. **Inc16 (16 位元加一器)**：直接利用 `Add16` 將輸入數值加上二進位的 `1`。
5. **ALU (算術邏輯單元)**：結合 `Add16` 與 Project 1 的多工器及位元邏輯閘，完成具備 18 種計算功能與狀態旗標的完整運算核心。

---

## 二、各晶片設計與原理解析

### 1. HalfAdder (半加器)
* **功能**：計算兩個 1-bit 輸入 $a$ 與 $b$ 的加法。
* **真值表**：
  | $a$ | $b$ | sum (總和) | carry (進位) |
  |:---:|:---:|:----------:|:------------:|
  |  0  |  0  |     0      |      0       |
  |  0  |  1  |     1      |      0       |
  |  1  |  0  |     1      |      0       |
  |  1  |  1  |     0      |      1       |
* **原理解析**：
  - **`sum`**：當兩輸入相異時為 1，相同時為 0，剛好符合互斥或邏輯：`Xor(a, b)`。
  - **`carry`**：只有當兩者皆為 1 時才會向高位進位，符合及運算邏輯：`And(a, b)`。

---

### 2. FullAdder (全加器)
* **功能**：計算三個 1-bit 輸入（包含本級的輸入 $a, b$ 與前一級傳入的進位 $c$）的相加。
* **原理解析**：
  1. 先用第一個 `HalfAdder` 將 $a$ 與 $b$ 相加，得到局部總和 $s_1$ 與局部進位 $c_1$。
  2. 再用第二個 `HalfAdder` 將 $s_1$ 與進位 $c$ 相加，得到最終的 **`sum`** 與第二個進位 $c_2$。
  3. 只要兩次加法中有任何一次產生進位（二者不可能同時為 1），最終的 **`carry`** 便為 1，因此使用 `Or(c1, c2)` 輸出進位。

---

### 3. Add16 (16 位元加法器)
* **功能**：將兩個 16-bit 數值（二補數系統）逐位元相加，產生 16-bit 的輸出結果。
* **原理解析**：
  - 採用**行波進位加法器（Ripple-Carry Adder）**架構。
  - 最低位（第 0 位）：因為沒有前一級的進位，直接使用 `HalfAdder`（或將 `FullAdder` 的進位端接 `false`）。
  - 第 1 位至第 15 位：每一級使用一個 `FullAdder`，輸入當前位元的 $a[i]$、$b[i]$，並接上上一級傳來的進位訊號。
  - 最高位（第 15 位）產生的進位為溢位（Overflow），根據 Hack 架構規格直接忽略。

---

### 4. Inc16 (16 位元加一器)
* **功能**：將 16-bit 輸入數值加上 1（即 $out = in + 1$）。
* **原理解析**：
  - 核心邏輯直接複用 **`Add16`**。
  - 第一個輸入端接原始訊號 `in`；第二個輸入端設定為常數 1。
  - 在 Hack HDL 語法中，指定 `b[0]=true` 即可，未指定的其他 15 個高位元（`b[1..15]`）模擬器會自動補 0（`false`），簡潔且有效率。

---

### 5. ALU (算術邏輯單元)
* **功能**：Hack CPU 的運算中樞，根據 6 個控制位元（`zx, nx, zy, ny, f, no`）操控兩組輸入並進行運算，同時輸出結果與兩個狀態旗標（`zr, ng`）。
* **流程階段解析**：
  1. **輸入前處理 ($x$ 與 $y$)**：
     - 若 `zx == 1`（清零），透過 `Mux16` 將輸入強制轉為全 0。
     - 若 `nx == 1`（取反），透過 `Not16` 與 `Mux16` 將數值逐位元反轉。
     - $y$ 的輸入依同樣邏輯由 `zy` 與 `ny` 控制。
  2. **核心運算 ($f$)**：
     - 同步平行計算加法（`Add16`）與逐位元及運算（`And16`）。
     - 由 `Mux16` 依據控制位元 `f` 挑選輸出（`f=1` 輸出加法，`f=0` 輸出 AND）。
  3. **輸出後處理 ($no$)**：
     - 若 `no == 1`，將運算結果透過 `Not16` 反轉。
  4. **狀態標誌輸出 ($zr, ng$)**：
     - **負數標誌 `ng`**：二補數中最高有效位元（MSB，即 bit 15）代表正負號。若 `out[15] == 1` 則為負數，可直接拉線輸出給 `ng`。
     - **零值標誌 `zr`**：將 16-bit 輸出拆成前後兩段 8-bit（`out[0..7]` 與 `out[8..15]`），分別通過 `Or8Way`，再將兩者以 `Or` 合併。若所有位元皆為 0，最終結果為 0，經過最後的 `Not` 反轉後即可輸出 `zr = 1`。

---

## 三、常見問題排查 (FAQ)

* **Q：跑 `ALU.tst` 時出現 `Comparison failure at line 3` 是為什麼？**
  * **原因**：通常是因為當前目錄下的 `Add16.hdl`、`FullAdder.hdl` 或 `HalfAdder.hdl` 尚未實作或內部有誤。
  * **排查重點**：檢查硬體模擬器中 Internal Pins 的 `addOut`。如果 $xFinal$ 與 $yFinal$ 皆為 $-1$，但 `addOut` 卻是 $0$（正確應為 $-2$），即代表加法器鏈條故障。請依實作順序先通過底層晶片的測試。
* **Q：程式碼在模擬器中註解變成亂碼？**
  * **原因**：檔案編碼問題。請使用支援 UTF-8 的編輯器（如 VS Code），並將檔案儲存為 **UTF-8 (無 BOM)** 編碼。
User 10:00 AM
我是說合成一份
Model 10:00 AM
ThinkingThoughts
Expand to view model thoughts

chevron_right
這是一份將實作順序、完整 HDL 程式碼、每題原理解析、除錯重點全部整合在同一份的完整 README.md：
Nand to Tetris - Project 02: 布林算術與 ALU (Boolean Arithmetic & ALU)
本專案實作了從最底層的 1 位元加法器到計算機運算核心——算術邏輯單元（ALU, Arithmetic Logic Unit）的硬體設計。
目錄
實作順序與晶片相依關係
各晶片實作程式碼與詳細解析
1. HalfAdder (半加器)
2. FullAdder (全加器)
3. Add16 (16 位元加法器)
4. Inc16 (16 位元加一器)
5. ALU (算術邏輯單元)
常見除錯與踩坑指南 (FAQ)
一、實作順序與晶片相依關係
在硬體模擬器中，高層晶片呼叫底層元件時會優先讀取同一資料夾下的檔案。如果順序相反或底層未完成，模擬器會讀取到未實作的空檔案，導致輸出全為 0 而報錯（例如 Comparison failure at line 3）。
晶片相依流程圖
code
Code
[ Project 01 基礎邏輯閘 ] (And, Xor, Or, Mux16, Not16...)
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
   4. Inc16 (加一器)   5. ALU (算術邏輯單元)
推薦開發步驟
HalfAdder：最基礎的單元，僅使用 Project 1 的 Xor 與 And。
FullAdder：使用 2 個 HalfAdder 與 1 個 Or 組成，可接收前級進位。
Add16：串接 16 個加法器完成 16 位元二補數加法（Ripple-Carry）。
Inc16：利用 Add16 將輸入數值加上常數 1。
ALU：運算中樞，結合 Add16 與多工器、位元邏輯閘，實現 18 種計算功能與旗標輸出。
二、各晶片實作程式碼與詳細解析
1. HalfAdder (半加器)
【功能與原理】
功能：計算兩個 1-bit 輸入 
a
a
 與 
b
b
 相加。
原理解析：
sum (總和)：
0
+
1
=
1
,
1
+
0
=
1
0+1=1,1+0=1
，但 
0
+
0
=
0
,
1
+
1
=
0
0+0=0,1+1=0
。兩者相異為 1，相同為 0，符合 Xor 邏輯。
carry (進位)：只有在 
1
+
1
1+1
 時才會產生進位向高位送，符合 And 邏輯。
【HDL 程式碼】
code
Hdl
// This file is part of www.nand2tetris.org
// and the book "The Elements of Computing Systems"
// by Nisan and Schocken, MIT Press.
// File name: projects/02/HalfAdder.hdl

/**
 * 計算兩個 1 位元的總和。
 */

CHIP HalfAdder {
    IN a, b;    // 1 位元輸入
    OUT sum,    // 加法結果的右側低位 (LSB)
        carry;  // 加法結果的左側進位 (MSB)

    PARTS:
    // 當 a 與 b 只有一個是 1 時，sum = 1 (使用 Xor)
    Xor(a=a, b=b, out=sum);

    // 當 a 與 b 同時為 1 時產生進位，carry = 1 (使用 And)
    And(a=a, b=b, out=carry);
}
2. FullAdder (全加器)
【功能與原理】
功能：計算三個 1-bit 輸入相加（
a
+
b
+
c
a+b+c
），其中 
c
c
 為前一級傳入的進位。
原理解析：
使用第 1 個 HalfAdder 將 
a
a
 與 
b
b
 相加，得到局部總和 
s
1
s 
1
​
 
 與進位 
c
1
c 
1
​
 
。
使用第 2 個 HalfAdder 將 
s
1
s 
1
​
 
 與 
c
c
 相加，得到最終總和 sum 與第二個進位 
c
2
c 
2
​
 
。
兩次加法中只要有任何一次進位（
c
1
c 
1
​
 
 或 
c
2
c 
2
​
 
 為 1，兩者不可能同時為 1），最終進位 carry 即為 1，因此使用 Or 閘連接。
【HDL 程式碼】
code
Hdl
// This file is part of www.nand2tetris.org
// and the book "The Elements of Computing Systems"
// by Nisan and Schocken, MIT Press.
// File name: projects/02/FullAdder.hdl

/**
 * 計算三個 1 位元的總和 (a + b + c)。
 */

CHIP FullAdder {
    IN a, b, c;  // 1 位元輸入
    OUT sum,     // 加法結果的右側低位 (LSB)
        carry;   // 加法結果的左側進位 (MSB)

    PARTS:
    // 1. 將 a 和 b 相加，產生局部總和 s1 與進位 c1
    HalfAdder(a=a, b=b, sum=s1, carry=c1);

    // 2. 將 s1 和 c 相加，得到最終總和 sum 與第二個進位 c2
    HalfAdder(a=s1, b=c, sum=sum, carry=c2);

    // 3. 兩次相加只要有任何一次產生進位，carry 就為 1
    Or(a=c1, b=c2, out=carry);
}
3. Add16 (16 位元加法器)
【功能與原理】
功能：將兩個 16-bit 二進位數值相加（二補數系統）。
原理解析：
採用行波進位架構（Ripple-Carry Adder）。
第 0 位：沒有前級進位，使用 HalfAdder（也可以用 FullAdder 並把進位端接 false）。
第 1 至 15 位：依序使用 15 個 FullAdder 串接上一位傳來的進位訊號。
第 15 位溢位：最高位的進位訊號根據規格直接捨棄不接。
【HDL 程式碼】
code
Hdl
// This file is part of www.nand2tetris.org
// and the book "The Elements of Computing Systems"
// by Nisan and Schocken, MIT Press.
// File name: projects/02/Adder16.hdl

/**
 * 將兩個 16 位元數值相加。
 * 忽略最高位的溢位進位 (overflow carry)。
 */

CHIP Add16 {
    IN a[16], b[16];
    OUT out[16];

    PARTS:
    // 第 0 位：沒有前一級進位，使用 HalfAdder
    HalfAdder(a=a[0], b=b[0], sum=out[0], carry=c0);

    // 第 1 到 15 位：依序使用 FullAdder 串接前一位的進位
    FullAdder(a=a[1],  b=b[1],  c=c0,  sum=out[1],  carry=c1);
    FullAdder(a=a[2],  b=b[2],  c=c1,  sum=out[2],  carry=c2);
    FullAdder(a=a[3],  b=b[3],  c=c2,  sum=out[3],  carry=c3);
    FullAdder(a=a[4],  b=b[4],  c=c3,  sum=out[4],  carry=c4);
    FullAdder(a=a[5],  b=b[5],  c=c4,  sum=out[5],  carry=c5);
    FullAdder(a=a[6],  b=b[6],  c=c5,  sum=out[6],  carry=c6);
    FullAdder(a=a[7],  b=b[7],  c=c6,  sum=out[7],  carry=c7);
    FullAdder(a=a[8],  b=b[8],  c=c7,  sum=out[8],  carry=c8);
    FullAdder(a=a[9],  b=b[9],  c=c8,  sum=out[9],  carry=c9);
    FullAdder(a=a[10], b=b[10], c=c9,  sum=out[10], carry=c10);
    FullAdder(a=a[11], b=b[11], c=c10, sum=out[11], carry=c11);
    FullAdder(a=a[12], b=b[12], c=c10, sum=out[12], carry=c12);
    FullAdder(a=a[13], b=b[13], c=c12, sum=out[13], carry=c13);
    FullAdder(a=a[14], b=b[14], c=c13, sum=out[14], carry=c14);
    // 第 15 位的進位不需接出 (直接忽略)
    FullAdder(a=a[15], b=b[15], c=c14, sum=out[15], carry=drop);
}
4. Inc16 (16 位元加一器)
【功能與原理】
功能：計算 
o
u
t
=
i
n
+
1
out=in+1
。
原理解析：
直接呼叫寫好的 Add16。
第一個輸入端接 
i
n
in
；第二個輸入端接常數 
1
1
。
在 HDL 語法中設定 b[0]=true 即可，未指定的其餘位元會自動補 0。
【HDL 程式碼】
code
Hdl
// This file is part of www.nand2tetris.org
// and the book "The Elements of Computing Systems"
// by Nisan and Schocken, MIT Press.
// File name: projects/02/Inc16.hdl

/**
 * 16 位元加一器 (Incrementer)：
 * out = in + 1 (算術加法)
 */

CHIP Inc16 {
    IN in[16];
    OUT out[16];

    PARTS:
    // 將輸入 in 與 1 (即 b[0]=true，其餘位元自動為 0) 相加
    Add16(a=in, b[0]=true, out=out);
}
5. ALU (算術邏輯單元)
【功能與原理】
功能：由 6 個控制位元（zx, nx, zy, ny, f, no）控制，計算出 18 種運算結果，並附帶 zr 與 ng 狀態旗標。
原理解析：
輸入前處理 (
x
,
y
x,y
)：
zx/zy：使用 Mux16 決定是否清零（換成 false）。
nx/ny：使用 Not16 加上 Mux16 決定是否取反。
核心運算 (
f
f
)：
同時透過 Add16 計算加法、透過 And16 計算逐位元及運算。
由 Mux16 依 f 挑選輸出（
f
=
1
f=1
 選加法，
f
=
0
f=0
 選 AND）。
輸出後處理 (
n
o
no
) 與分流：
依 no 決定輸出是否透過 Not16 取反。
利用 HDL 的匯流排分流技巧，在最後的 Mux16 同時拉出 out、out[15]（給 ng 判斷負數）、以及低 8 位 out[0..7] 與高 8 位 out[8..15]。
零值標誌 (
z
r
zr
)：
透過兩顆 Or8Way 與一顆 Or 檢查所有位元。若有任何一個位元是 1，結果就為 1。
最後接上 Not，只有全為 0 時輸出才會是 1。
【HDL 程式碼】
code
Hdl
// This file is part of www.nand2tetris.org
// and the book "The Elements of Computing Systems"
// by Nisan and Schocken, MIT Press.
// File name: projects/02/ALU.hdl

/**
 * 算術邏輯單元（ALU）。
 * 根據 6 個控制位元（zx, nx, zy, ny, f, no）計算 18 種功能之一，
 * 並輸出 zr (是否為 0) 與 ng (是否為負數) 旗標。
 */

CHIP ALU {
    IN  
        x[16], y[16],  // 16 位元輸入        
        zx, // 是否將 x 輸入清零？
        nx, // 是否將 x 輸入取反？
        zy, // 是否將 y 輸入清零？
        ny, // 是否將 y 輸入取反？
        f,  // 運算選擇：1 代表加法 (x + y)，0 代表逐位元及運算 (x & y)
        no; // 是否將輸出結果取反？

    OUT 
        out[16], // 16 位元輸出
        zr,      // 若 out == 0 則為 1，否則為 0
        ng;      // 若 out < 0  則為 1，否則為 0

    PARTS:
    // --- 1. 處理 X 輸入（zx, nx）---
    Mux16(a=x, b=false, sel=zx, out=xZeroed);
    Not16(in=xZeroed, out=notX);
    Mux16(a=xZeroed, b=notX, sel=nx, out=xFinal);

    // --- 2. 處理 Y 輸入（zy, ny）---
    Mux16(a=y, b=false, sel=zy, out=yZeroed);
    Not16(in=yZeroed, out=notY);
    Mux16(a=yZeroed, b=notY, sel=ny, out=yFinal);

    // --- 3. 核心運算（f）---
    Add16(a=xFinal, b=yFinal, out=addOut);
    And16(a=xFinal, b=yFinal, out=andOut);
    Mux16(a=andOut, b=addOut, sel=f, out=fOut);

    // --- 4. 輸出取反（no）與分流拉線 ---
    Not16(in=fOut, out=notFOut);
    Mux16(a=fOut, b=notFOut, sel=no, 
          out=out, 
          out[15]=ng, 
          out[0..7]=lowHalf, 
          out[8..15]=highHalf);

    // --- 5. 判斷輸出是否為零（zr 標誌）---
    Or8Way(in=lowHalf, out=orLow);
    Or8Way(in=highHalf, out=orHigh);
    Or(a=orLow, b=orHigh, out=orAll);
    Not(in=orAll, out=zr);
}
三、常見除錯與踩坑指南 (FAQ)
問：執行 ALU.tst 出現 Comparison failure at line 3？
原因：ALU 內使用了 Add16，模擬器會載入同一目錄下的 Add16.hdl。若 Add16 尚未實作或其相依的 HalfAdder、FullAdder 有錯，加法結果會輸出全 0（例如截圖中 addOut = 0），導致第 3 行的比對失敗。
解法：必須依照順序（HalfAdder ➜ FullAdder ➜ Add16）先通過各自的 .tst 測試後，再來測試 ALU。
問：模擬器左下角預覽出現亂碼？
原因：檔案編碼不是標準的 UTF-8。
解法：使用 VS Code 另存檔案，確認右下角編碼為 UTF-8（無 BOM），避免中文字元影響模擬器的語法剖析器。
Citations

https://blog.csdn.net/qq_17065591/article/details/107029998
19s
Model 10:01 AM
ThinkingThoughts
Expand to view model thoughts

chevron_right
從你之前的截圖路徑可以看出，你的專案原本就放在本機的 GitHub 資料夾：
C:\Users\Echoooooo\Documents\GitHub\_co-111410508
以下教你如何用 VS Code 開啟、建立 README.md，並一步一步推送到 GitHub：
步驟 1：用 VS Code 開啟你的專案資料夾
開啟 VS Code。
點擊上方選單的 「檔案 (File)」 ➜ 「開啟資料夾 (Open Folder...)」。
找到並選擇你的 GitHub 倉庫資料夾：
C:\Users\Echoooooo\Documents\GitHub\_co-111410508
點選「選取資料夾」，這時左側的檔案總管就會顯示你所有的作業檔案。
步驟 2：建立 README.md 並存檔
在左側檔案列表找到你的 02 資料夾（或是看教授規定要放在專案最外層還是 02 資料夾內）。
在該資料夾上按右鍵 ➜「新增檔案 (New File)」。
檔名輸入 README.md（注意副檔名是 .md，代表 Markdown）。
把上一則回覆中的所有內容複製並貼上。
按 Ctrl + S 存檔。
小提醒：看一眼 VS Code 右下角狀態列，確認編碼顯示為 UTF-8。
步驟 3：推送到 GitHub（提供兩種方法，任選一種即可）
方法 A：使用 VS Code 圖形介面（推薦，最簡單）
點擊 VS Code 左側工具列的 「原始檔控制」 圖示（外觀是三個圓點相連的 Git 分支圖示，或按快速鍵 Ctrl + Shift + G）。
你會看到「變更 (Changes)」清單列出剛修改或新增的檔案（如 README.md、ALU.hdl 等）。
在上方的文字輸入框內輸入這次的更新說明（Commit Message），例如：
code
Text
Complete Project 02 and add README
點擊藍色的 「認可 (Commit)」 按鈕（或打勾符號 ✔）。
如果跳出提示問你「是否要將所有變更暫存 (Stage All)？」，請點選 「是 (Yes)」。
Commit 完成後，藍色按鈕會變成 「同步變更 (Sync Changes)」（或在右上方 ... 點選 推送 (Push)），點下去就會自動上傳到你的 GitHub 了！
方法 B：使用 VS Code 內建終端機（指令操作）
如果你習慣使用終端機，可以按鍵盤快捷鍵 Ctrl + ~（Esc 下方那個鍵）開啟終端機，然後依序輸入以下指令：
code
Bash
# 1. 將所有修改與新增的檔案加入暫存區
git add .

# 2. 建立提交並附上說明
git commit -m "Complete Project 02 and add README"

# 3. 推送到遠端 GitHub 儲存庫
git push
步驟 4：確認上傳成功
打開瀏覽器進入你的 GitHub 專案頁面（github.com/你的帳號/_co-111410508），重新整理頁面，點進對應資料夾確認剛剛寫好的程式碼與美化排版的 README.md 都已成功顯示！
info
Google AI models may make mistakes, so double-check outputs.
Use Arrow Up and Arrow Down to select a turn, Enter to jump to it, and Escape to return to the chat.
Start typing a prompt to see what our models can do


code
Code execution

google
Grounding with Google Search


Response ready.