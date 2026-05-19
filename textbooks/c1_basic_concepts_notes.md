# Chapter 1：Basic Concepts 完整筆記

> 課程主軸：組合語言（Assembly Language）基礎、資料表示法、布林運算、x86 處理器架構

---

## 📚 目錄
1. [組譯器與相關工具](#一組譯器assembler與相關工具)
2. [資料表示法 Data Representation](#二資料表示法-data-representation)
3. [布林運算 Boolean Operations](#三布林運算-boolean-operations)
4. [x86 處理器架構](#四x86-處理器架構)
5. [章節重點](#五章節重點)

---

## 一、組譯器（Assembler）與相關工具

### 1. 什麼是 Assembler？
- **定義**：把 **組合語言程式（Assembly Language）** 轉成 **機器語言程式（Machine Language）** 的程式。
- **流程圖解**：

```
   組合語言                         機器語言
 ┌─────────────┐                ┌──────────────┐
 │ mov ecx,ebx │  Assembler +   │ 100101011001 │
 │ mov esp,edx │  ───Linker──▶  │ 010011111011 │
 │ mov edx,r9d │                │ 111010101101 │
 │ mov rax,rdx │                │ 01010101010  │
 └─────────────┘                └──────────────┘
   Programmer                      Processor
```

### 2. 語言之間的翻譯關係
```
英文：Display the sum of A times B plus C.
   │
   ▼
C++：cout << (A * B + C);
   │                    │ Compiler
   ▼                    ▼
組合語言                Intel 機器語言
mov eax, A             A1 00000000
mul B          ─────▶  F7 25 00000004
add eax, C    Assembler 03 05 00000008
call WriteInt          E8 00500000
```

### 3. 什麼是 Linker？
- **定義**：將 Assembler 產生的多個 **object files** 結合成一個 **executable file**（可執行檔）。
- 產生的可執行檔可以被載入記憶體並執行。

### 📝 例題：連結兩個 C 檔案
```c
// main.c
#include <stdio.h>
int add(int a, int b);
int main() {
    printf("Sum: %d\n", add(3, 5));
    return 0;
}

// add.c
int add(int a, int b) {
    return a + b;
}
```
編譯與連結指令：
```bash
gcc -c main.c -o main.o   # 編譯 main.c 為 object file
gcc -c add.c  -o add.o    # 編譯 add.c 為 object file
gcc main.o add.o -o program   # 連結成可執行檔
./program                 # 執行
# 輸出：Sum: 8
```

### 4. 什麼是 Debugger？
- 讓程式設計師可以 **trace（追蹤）** 程式執行並 **examine（檢查）** 記憶體內容的工具。

### 5. 開發環境需求
- **硬體**：x86 處理器家族
- **作業系統**：Windows
- **開發工具**：Microsoft Visual Studio，包含 Editor、Assembler、Linker、Debugger。

### 6. 組合語言 vs 其他語言
| 比較對象 | 關係 |
|---------|------|
| AL ↔ Machine Language | **One-to-one**（一對一） |
| AL ↔ C++/Java | **One-to-many**（一對多） |

📝 範例：`X = (Y + 4) * 3` 對應的組合語言
```
mov eax, Y;
add eax, 4;
mov ebx, 3;
imul ebx;
mov X, eax
```

### 7. 組合語言可移植嗎（Portable）？
- **不可移植**：因為 AL 綁定在特定處理器家族（processor family）。

---

## 二、資料表示法 Data Representation

### 1. Binary Numbers（二進位數）
- 數字只有 **0** 和 **1**（1 = true、0 = false）。
- **MSB**（Most Significant Bit）：最高有效位元（最左邊）
- **LSB**（Least Significant Bit）：最低有效位元（最右邊）

```
MSB                            LSB
[1 0 1 1 0 0 1 0 1 0 0 1 1 1 0 0]
 15                             0
```

### 2. 二進位每個 bit 代表的權重
```
[1][1][1][1][1][1][1][1]
 2⁷ 2⁶ 2⁵ 2⁴ 2³ 2² 2¹ 2⁰
```

| 2ⁿ | 值 | 2ⁿ | 值 |
|----|----|----|----|
| 2⁰ | 1 | 2⁸ | 256 |
| 2¹ | 2 | 2⁹ | 512 |
| 2² | 4 | 2¹⁰ | 1024 |
| 2³ | 8 | 2¹¹ | 2048 |
| 2⁴ | 16 | 2¹² | 4096 |
| 2⁵ | 32 | 2¹³ | 8192 |
| 2⁶ | 64 | 2¹⁴ | 16384 |
| 2⁷ | 128 | 2¹⁵ | 32768 |

### 3. Binary → Decimal（加權位置記號法）
公式：
$$Dec = (D_{n-1} \times 2^{n-1}) + (D_{n-2} \times 2^{n-2}) + ... + (D_1 \times 2^1) + (D_0 \times 2^0)$$

📝 範例：`00001001₂ = ?`
- (1 × 2³) + (1 × 2⁰) = 8 + 1 = **9**

### 4. Decimal → Binary（重複除以 2 法）
📝 範例：`37 = ?₂`

| 除法 | 商 | 餘 |
|------|----|----|
| 37 / 2 | 18 | **1** |
| 18 / 2 | 9 | **0** |
| 9 / 2 | 4 | **1** |
| 4 / 2 | 2 | **0** |
| 2 / 2 | 1 | **0** |
| 1 / 2 | 0 | **1** |

由下往上讀：**37 = 100101₂**

### 5. Binary Addition（二進位加法）
- 從 LSB 開始相加，必要時把 carry（進位）一併加上。

📝 範例：4 + 7 = 11
```
       carry: 1
        0 0 0 0 0 1 0 0   (4)
   +    0 0 0 0 0 1 1 1   (7)
   ────────────────────
        0 0 0 0 1 0 1 1   (11)
bit:    7 6 5 4 3 2 1 0
```

### 6. Integer Storage Sizes（整數儲存大小）

| 名稱 | bits |
|------|------|
| byte | 8 |
| word | 16 |
| doubleword | 32 |
| quadword | 64 |

**Unsigned 整數範圍**：

| 型別 | 範圍 | 2 的次方 |
|------|------|---------|
| Unsigned byte | 0 ~ 255 | 0 ~ (2⁸ − 1) |
| Unsigned word | 0 ~ 65,535 | 0 ~ (2¹⁶ − 1) |
| Unsigned doubleword | 0 ~ 4,294,967,295 | 0 ~ (2³² − 1) |
| Unsigned quadword | 0 ~ 18,446,744,073,709,551,615 | 0 ~ (2⁶⁴ − 1) |

📝 **練習**：20 bits 可以儲存的最大 unsigned 整數是多少？
→ **答：2²⁰ − 1 = 1,048,575**

### 7. Hexadecimal Integers（16 進位）

| Binary | Dec | Hex | Binary | Dec | Hex |
|--------|-----|-----|--------|-----|-----|
| 0000 | 0 | 0 | 1000 | 8 | 8 |
| 0001 | 1 | 1 | 1001 | 9 | 9 |
| 0010 | 2 | 2 | 1010 | 10 | A |
| 0011 | 3 | 3 | 1011 | 11 | B |
| 0100 | 4 | 4 | 1100 | 12 | C |
| 0101 | 5 | 5 | 1101 | 13 | D |
| 0110 | 6 | 6 | 1110 | 14 | E |
| 0111 | 7 | 7 | 1111 | 15 | F |

### 8. Binary → Hex
- 每 4 個 binary bits 對應 1 個 hex digit。

📝 範例：`000101101010011110010100₂ → ?`
```
0001 | 0110 | 1010 | 0111 | 1001 | 0100
  1  |   6  |   A  |   7  |   9  |   4
```
→ **16A794₁₆**

### 9. Hex → Decimal
公式：$dec = (D_3 \times 16^3) + (D_2 \times 16^2) + (D_1 \times 16^1) + (D_0 \times 16^0)$

📝 範例：
- `1234₁₆ = (1×16³)+(2×16²)+(3×16¹)+(4×16⁰) = 4,660`
- `3BA4₁₆ = (3×16³)+(11×16²)+(10×16¹)+(4×16⁰) = 15,268`

**16 的次方表（背重點）**：
| 16ⁿ | 值 | 16ⁿ | 值 |
|-----|----|----|-----|
| 16⁰ | 1 | 16⁴ | 65,536 |
| 16¹ | 16 | 16⁵ | 1,048,576 |
| 16² | 256 | 16⁶ | 16,777,216 |
| 16³ | 4,096 | 16⁷ | 268,435,456 |

### 10. Decimal → Hex
📝 範例：`422 → ?₁₆`

| 除法 | 商 | 餘 |
|------|----|----|
| 422 / 16 | 26 | **6** |
| 26 / 16 | 1 | **A** |
| 1 / 16 | 0 | **1** |

→ **decimal 422 = 1A6₁₆**

### 11. Hexadecimal Addition
規則：兩個位數相加後除以 base（16），商為進位，餘為該位數的值。

📝 範例：`6A + 4B`
```
   1            ← carry
   6A
 + 4B
 ────
   B5      （A + B = 21；21/16 = 1 餘 5；商 1 進位）
```

### 12. Hexadecimal Subtraction
規則：需要借位時，向左邊借 1 → 當前位數 +16。

📝 範例：`75 − 47 = ?`
```
   -1↓          ← 借位
   75
 - 47
 ────
   2E          （5 不夠減 7，借位：16+5=21；21-7=14=E；7-1-4=2）
```

📝 **練習**：var1 位址是 00400020，下一個變數的位址是 0040006A，var1 佔幾個 bytes？
→ 0040006A − 00400020 = **4A₁₆ = 74 bytes**

### 13. Signed Integers（有號整數）
- **最高位（MSB）為 sign bit**：1 = 負、0 = 正
- 16 進位最高位 > 7 即為負數，例如：**8A, C5, A2, 9D**

### 14. Two's Complement（二補數）
- 負數以二補數表示
- 步驟：① 反轉所有 bits → ② 加 1

📝 範例：`+1 → −1`

| 步驟 | 值 |
|------|-----|
| 起始值（+1） | 00000001 |
| Step 1：反轉 | 11111110 |
| Step 2：+1 | 11111111（即 −1） |

驗證：`00000001 + 11111111 = 00000000` ✅

### 15. Binary Subtraction
規則：`A − B` → 把 B 轉成二補數，然後做 `A + (−B)`

📝 範例：`00001100 − 00000011`
```
  00001100              00001100
- 00000011    ──▶     + 11111101
                       ──────────
                        00001001
```

📝 **練習**：`1001 − 0101 = ?` → **0100**

### 16. Signed Integer 範圍

| 型別 | 範圍 | 2 的次方 |
|------|------|---------|
| Signed byte | −128 ~ +127 | −2⁷ ~ (2⁷−1) |
| Signed word | −32,768 ~ +32,767 | −2¹⁵ ~ (2¹⁵−1) |
| Signed doubleword | −2,147,483,648 ~ +2,147,483,647 | −2³¹ ~ (2³¹−1) |
| Signed quadword | ±9.2×10¹⁸ | −2⁶³ ~ (2⁶³−1) |

📝 **練習**：20 bits 能儲存的最大正數？
→ **2¹⁹ − 1 = 524,287**

### 17. Character Storage（字元儲存）
- **Standard ASCII**（0–127）：基本英文字母、數字、控制字符
- **Extended ASCII**（0–255）：加入特殊字符與國際符號
- **ANSI**（0–255）：與 Extended ASCII 類似，常用於 Windows-1252
- **Unicode**（0–1,114,111）：包含全球語言字符、表情符號

📝 範例：
```c
char c = 'A';
printf("%d", c);   // 輸出 65
```

📝 **趣味練習**：`54 68 65 20 45 6E 64` 代表什麼？
→ 用 ASCII 轉換：**"The End"**

---

## 三、布林運算 Boolean Operations

### 1. Boolean Algebra
- 由 **George Boole** 設計的符號邏輯系統
- 三大基本運算子：**NOT、AND、OR**

| 表示式 | 意義 |
|--------|------|
| ¬X | NOT X |
| X ∧ Y | X AND Y |
| X ∨ Y | X OR Y |
| ¬X ∨ Y | (NOT X) OR Y |
| ¬(X ∧ Y) | NOT (X AND Y) |
| X ∧ ¬Y | X AND (NOT Y) |

### 2. NOT（反相）
反轉 boolean 值。

| X | ¬X |
|---|----|
| F | T |
| T | F |

Digital gate：`──▷○──`（三角形＋圓圈）

### 3. AND（且）
| X | Y | X ∧ Y |
|---|---|-------|
| F | F | F |
| F | T | F |
| T | F | F |
| T | T | T |

Digital gate：`──D──`（D 形）

### 4. OR（或）
| X | Y | X ∨ Y |
|---|---|-------|
| F | F | F |
| F | T | T |
| T | F | T |
| T | T | T |

Digital gate：`──⊃──`（凹弧形）

### 5. Operator Precedence（運算子優先序）

| 表示式 | 運算順序 |
|--------|---------|
| ¬X ∨ Y | NOT，再 OR |
| ¬(X ∨ Y) | OR，再 NOT |
| X ∨ (Y ∧ Z) | AND，再 OR |

🔑 **背口訣**：**NOT > AND > OR**（括號最大）

### 6. Truth Table 範例

📝 範例 1：`¬X ∨ Y`

| X | ¬X | Y | ¬X ∨ Y |
|---|----|----|--------|
| F | T | F | T |
| F | T | T | T |
| T | F | F | F |
| T | F | T | T |

📝 範例 2：`X ∧ ¬Y`

| X | Y | ¬Y | X ∧ ¬Y |
|---|---|----|--------|
| F | F | T | F |
| F | T | F | F |
| T | F | T | **T** |
| T | T | F | F |

📝 範例 3（Two-input Multiplexer 多工器）：`(Y ∧ S) ∨ (X ∧ ¬S)`

| X | Y | S | Y∧S | ¬S | X∧¬S | (Y∧S)∨(X∧¬S) |
|---|---|---|-----|----|----|-------------|
| F | F | F | F | T | F | F |
| F | T | F | F | T | F | F |
| T | F | F | F | T | T | T |
| T | T | F | F | T | T | T |
| F | F | T | F | F | F | F |
| F | T | T | T | F | F | T |
| T | F | T | F | F | F | F |
| T | T | T | T | F | F | T |

說明：當 S=0，輸出 = X；當 S=1，輸出 = Y。

---

## 四、x86 處理器架構

### 1. 什麼是 x86 Processor？
- 由 **Intel 於 1978 年** 推出的處理器架構
- 命名來自早期處理器：**8086, 80286, 80386, 80486** → "86" 成為共同識別

### 2. x86 架構特性
| 特性 | 說明 |
|------|------|
| **Backward compatibility** | 新處理器可執行為舊處理器寫的程式 |
| **Widely supported** | Windows、Linux、macOS 都支援 |
| **Extensive hardware/software support** | 龐大的軟硬體生態系 |
| **Complex instruction set (CISC)** | 指令集龐大，可用單一指令完成複雜任務 |

### 3. 電腦系統基本架構
```
┌─────────────────── data bus ──────────────────────────┐
│                                                         │
│ ┌──────────────┐  ┌─────────┐  ┌──────┐  ┌──────┐    │
│ │  registers   │  │ Memory  │  │ I/O  │  │ I/O  │    │
│ │  ┌────────┐  │  │ Storage │  │ Dev1 │  │ Dev2 │    │
│ │  │  CPU   │  │  │  Unit   │  │      │  │      │    │
│ │  │ALU CU 鐘│ │  │         │  │      │  │      │    │
│ │  └────────┘  │  │         │  │      │  │      │    │
│ └──────────────┘  └─────────┘  └──────┘  └──────┘    │
│                                                         │
└─── control bus ──── address bus ──────────────────────┘
```

### 4. Registers（暫存器）
- **特性**：CPU 內部的 **高速** 儲存位置
- **Intel basic program execution registers**：
  - 8 個 general-purpose registers
  - 6 個 segment registers
  - **EFLAGS**：處理器狀態旗標
  - **EIP**：指令指標

```
32-bit General-Purpose                16-bit Segment
┌─────┐  ┌─────┐                      ┌────┐  ┌────┐
│ EAX │  │ EBP │                      │ CS │  │ ES │
│ EBX │  │ ESP │                      │ SS │  │ FS │
│ ECX │  │ ESI │                      │ DS │  │ GS │
│ EDX │  │ EDI │                      └────┘  └────┘
└─────┘  └─────┘
┌────────┐
│ EFLAGS │
│   EIP  │
└────────┘
```

### 5. Memory Hierarchy（記憶體階層）
| 層級 | 大小 | 存取時間 |
|------|------|---------|
| Registers | 64 ~ 256 words | < 1 cycle |
| Cache 1 (L1) | 8 K words | 1–2 cycles |
| Cache 2 (L2) | 256 K words | 5–15 cycles |
| Memory (RAM) | 4 G words | 40–100 cycles |

🔑 **越靠近 CPU 越快越小**

### 6. General-Purpose Registers（拆分使用）
EAX、EBX、ECX、EDX 可拆成不同大小存取：

| 32-bit | 16-bit | 8-bit (高位) | 8-bit (低位) |
|--------|--------|------------|-------------|
| EAX | AX | AH | AL |
| EBX | BX | BH | BL |
| ECX | CX | CH | CL |
| EDX | DX | DH | DL |

```
   8     8
┌─────┬─────┐
│ AH  │ AL  │   ← 各 8 bits
└─────┴─────┘
┌───────────┐
│    AX     │   ← 16 bits
└───────────┘
┌───────────────────┐
│       EAX         │   ← 32 bits
└───────────────────┘
```

### 7. Index 與 Base Registers（只有 16-bit 名稱）
| 32-bit | 16-bit |
|--------|--------|
| ESI | SI |
| EDI | DI |
| EBP | BP |
| ESP | SP |

### 8. Specialized Register Uses（特殊用途）
| 暫存器 | 用途 | 助記 |
|--------|------|------|
| **EAX** | extended **A**ccumulator | 乘除法使用 |
| **ECX** | loop **C**ounter | 迴圈計數 |
| **ESP** | extended **S**tack **P**ointer | 堆疊指標 |
| **ESI, EDI** | **I**ndex registers | source/destination index |
| **EBP** | extended **B**ase pointer | 參考函式參數與區域變數 |

🔑 **背口訣**：A=Accumulator、C=Counter、S=Stack、I=Index、B=Base

### 9. Segment Registers（段暫存器）
| 暫存器 | 用途 |
|--------|------|
| **CS** (Code Segment) | 存放指令 |
| **DS** (Data Segment) | 存放全域變數 |
| **SS** (Stack Segment) | 存放區域變數與函式參數 |
| **ES, FS, GS** | 額外段 |

### 10. EIP（Instruction Pointer 指令指標）
- 儲存 **下一條要執行的指令位址**

📝 圖解：當 `EIP = 1009` 時，剛剛執行完 1006，下一條會執行 1009 (`imul ebx`)
```
EIP = 1009
        1000  mov eax, Y;
        1003  add eax, 4;
        1006  mov ebx 3;    ← 剛執行完
        1009  imul ebx;     ← 下一條要執行
        100C  mov X, eax;
```

### 11. EFLAGS Register（旗標暫存器）
- 每個 flag 是一個 binary bit
- 分成兩類：**Status Flag**（狀態）與 **Control Flag**（控制）

#### Status Flag（狀態旗標）— ⭐ 考試重點
| Flag | 名稱 | 觸發條件 |
|------|------|---------|
| **CF** | Carry Flag | unsigned 運算 **超出範圍** |
| **OF** | Overflow Flag | signed 運算 **超出範圍** |
| **SF** | Sign Flag | 結果為 **負數** |
| **ZF** | Zero Flag | 結果為 **0** |
| **AF** | Auxiliary Carry | 從 bit 3 → bit 4 進位 |
| **PF** | Parity Flag | 結果中 1 的個數為 **偶數** |

#### Control Flag（控制旗標）
- 控制 CPU 操作，例如 **DF (Direction Flag)**

### 12. System Registers（略讀）
- 僅最高權限（level 0，如 Windows XP 核心）可存取
- 包含：IDTR、GDTR、LDTR、Task Registers、CR0/CR2/CR3/CR4、MSR

### 13. 多媒體與浮點暫存器（略讀）
- **MMX**：8 個 64-bit 暫存器
- **XMM**：8 個 128-bit 暫存器（SIMD 用）
- **FPU**：8 個 80-bit 浮點暫存器 ST(0)–ST(7)，以 stack 形式排列

### 14. 64-bit x86-64 Processors

#### 操作模式
- **Compatibility mode**：可執行 16-bit 與 32-bit app（Windows 此模式只支援 32-bit）
- **64-bit mode**：Windows 64 使用此模式

#### 基本執行環境
- 位址可達 **64 bits**（實際使用 48 bits）
- **16 個** 64-bit 通用暫存器
- 指令指標稱為 **RIP**（64-bit）

#### 64-bit 通用暫存器命名
| 32-bit 名稱 | 64-bit 名稱 |
|------------|------------|
| EAX, EBX, ECX, EDX | RAX, RBX, RCX, RDX |
| EDI, ESI, EBP, ESP | RDI, RSI, RBP, RSP |
| R8D ~ R15D | R8 ~ R15 |

🔑 **規則**：`E` 開頭（32-bit）→ `R` 開頭（64-bit）；新加 R8~R15。

---

## 五、章節重點
1. 組合語言幫助你理解軟體的 **最底層（lowest level）** 是如何構造的。
2. 組合語言與機器語言是 **one-to-one** 的關係。
3. Boolean expressions 是設計電腦軟硬體的基礎。

---

# 🔥 速背重點整理（考前衝刺）

## ⭐ 核心定義
| 名詞 | 一句話定義 |
|------|----------|
| **Assembler** | AL → Machine Language 的轉換程式 |
| **Linker** | 結合多個 object files 成 executable |
| **Debugger** | 追蹤執行＋檢查記憶體的工具 |
| **AL ↔ Machine** | One-to-one |
| **C++/Java ↔ AL** | One-to-many |
| **AL 可移植？** | ❌ 不可移植，綁定處理器家族 |

## ⭐ 數字轉換口訣
- **Bin → Dec**：每位 ×2ⁿ，加起來
- **Dec → Bin**：重複除以 2，餘數**由下往上**讀
- **Hex → Dec**：每位 ×16ⁿ
- **Dec → Hex**：重複除以 16，餘數**由下往上**讀
- **Bin ↔ Hex**：每 **4 bits = 1 hex digit**

## ⭐ 二補數三步驟
1. 反轉所有 bits（NOT）
2. +1
3. 結果就是 −原數

> A − B = A + (−B) = A + 二補數(B)

## ⭐ 儲存大小範圍（必背）
| Size | Unsigned 最大 | Signed 範圍 |
|------|------------|------------|
| byte (8) | 255 | −128 ~ +127 |
| word (16) | 65,535 | −32,768 ~ +32,767 |
| dword (32) | 4,294,967,295 | ±2,147,483,647 |
| qword (64) | 2⁶⁴−1 | ±2⁶³−1 |

> **公式記**：Unsigned = 2ⁿ−1；Signed = ±2ⁿ⁻¹（負數多 1）

## ⭐ Hex 判斷正負
- **最高 hex digit > 7 即為負數**
- 例：**8A, C5, A2, 9D** 都是負數

## ⭐ 布林優先序
**NOT > AND > OR**（括號最大）

## ⭐ 暫存器助記（必背）
| 暫存器 | 助記 |
|--------|------|
| EAX | **A**ccumulator（乘除法） |
| ECX | **C**ounter（迴圈） |
| ESP | **S**tack **P**ointer |
| EBP | **B**ase Pointer |
| ESI/EDI | **I**ndex（source/dest） |
| EIP | **I**nstruction **P**ointer（下一條指令位址） |

## ⭐ Segment Registers
| 暫存器 | 內容 |
|--------|------|
| **CS** | Code（指令） |
| **DS** | Data（全域變數） |
| **SS** | Stack（區域變數、參數） |

## ⭐ EFLAGS Status Flags（必背）
| Flag | 觸發 |
|------|------|
| **CF** | **U**nsigned 溢位 |
| **OF** | **S**igned 溢位 |
| **SF** | 結果為負 |
| **ZF** | 結果為零 |
| **AF** | bit3→bit4 進位 |
| **PF** | 1 的個數為偶數 |

> 助記：**C**arry=u**n**signed、**O**verflow=**S**igned

## ⭐ Memory Hierarchy（由快到慢）
```
Registers > L1 Cache > L2 Cache > Memory (RAM)
 <1 cycle    1-2       5-15       40-100
```

## ⭐ 32-bit ↔ 64-bit 暫存器
- `E` 前綴 → `R` 前綴
- EAX → RAX、EIP → RIP
- 新增：**R8 ~ R15**（共 16 個通用暫存器）
- 位址：64-bit（實際 48-bit）

## ⭐ Hex 加法快速心法
- 兩位相加 → 除以 16 → 商=進位、餘=當位
- 字母對應：A=10, B=11, C=12, D=13, E=14, F=15

## ⭐ 重要 2 的次方（必背）
- 2¹⁰ = 1024（≈ 1 K）
- 2²⁰ = 1,048,576（≈ 1 M）
- 2¹⁶ = 65,536
- 2³² = 4,294,967,296（≈ 4 G）
