# Chapter 4：Procedures 完整筆記

> 課程主軸：Runtime Stack、PUSH/POP、定義與呼叫 Procedure、外部 Library（Irvine32）

---

## 📚 目錄
1. [Stack Operations](#一stack-operations)
2. [Defining and Using Procedures](#二defining-and-using-procedures)
3. [Linking to an External Library](#三linking-to-an-external-library)
4. [The Irvine32 Link Library](#四the-irvine32-link-library)
5. [速背重點整理](#速背重點整理)

---

## 一、Stack Operations

### 1. Runtime Stack（執行期堆疊）
- 由兩個暫存器管理的 **記憶體區域**：
  - **SS**（Stack Segment）：存放 segment descriptor
  - **ESP**（Stack Pointer）：指向 stack 的 32-bit offset

```
Offset
00001000  [ 00000006 ]  ← ESP
00000FFC
00000FF8
00000FF4
00000FF0
```

🔑 **重點**：**SS 與 ESP 不直接修改**，而是透過 **push / pop / call / ret** 間接修改。

### 2. Stack 的常見用途
1. 暫存暫存器（**Temporary save area**）
2. 呼叫 procedure 時 CPU 把 **return address** 存到 stack
3. 呼叫 procedure 時的 **arguments** 推入 stack
4. Procedure 內的 **local variables** 在 stack 建立、結束時釋放

### 3. PUSH 操作
**兩步驟**：
1. **ESP −= 4 或 2**（向下成長）
2. 把值複製到 ESP 指到的位置

📝 圖解 `push eax`（假設 EAX=0xA5）：
```
   BEFORE                          AFTER
   ┌──────────┐                   ┌──────────┐
   │ 00000006 │ ← ESP             │ 00000006 │
   ├──────────┤                   ├──────────┤
   │          │                   │ 000000A5 │ ← ESP
   ├──────────┤                   ├──────────┤
   │          │                   │          │
   └──────────┘                   └──────────┘
```

📝 連續 push 三個值的結果：
```
00001000  00000006
00000FFC  000000A5
00000FF8  00000001
00000FF4  00000002  ← ESP
```

🔑 **Stack 往下成長**（往較低位址）。ESP 下方的區域是 stack 的有效範圍。

### 4. POP 操作
**兩步驟**：
1. 把 stack[ESP] 內容複製到 register 或 variable
2. **ESP += 4 或 2**（依 operand 大小）

📝 圖解 `pop eax`：
```
   BEFORE                          AFTER
   00001000 00000006              00001000 00000006
   00000FFC 000000A5              00000FFC 000000A5
   00000FF8 00000001              00000FF8 00000001  ← ESP
   00000FF4 00000002 ← ESP        00000FF4
```

### 5. PUSH / POP 語法
| 語法 | 說明 |
|------|------|
| `PUSH r/m16` | 16-bit reg/mem |
| `PUSH r/m32` | 32-bit reg/mem |
| `PUSH imm32` | 32-bit 立即值 |
| `POP r/m16` | 16-bit reg/mem |
| `POP r/m32` | 32-bit reg/mem |

> ⚠️ **POP 沒有 immediate 版本**！

### 6. 用 PUSH/POP 暫存暫存器（順序相反）⭐
```asm
push esi
push ecx
push ebx

mov esi, OFFSET dwordVal
mov ecx, LENGTHOF dwordVal
mov ebx, TYPE dwordVal
call DumpMem

pop ebx        ; 注意：POP 順序與 PUSH 相反！
pop ecx
pop esi
```

🔑 **LIFO**：先進後出。

### 7. 範例：反轉字串（Reversing a String）
**思路**：
1. 用 loop + indexed addressing
2. **每個字元 push 到 stack**
3. 從字串開頭，**反序 pop**，逐字寫回原位置

```asm
.data
aName    BYTE "Alraham Lincoln", 0
nameSize = ($ - aName) - 1
.code
main PROC
    ; 把字串推入 stack
    mov ecx, nameSize
    mov esi, 0
L1: movzx eax, aName[esi]
    push eax              ; 推一個字元
    inc esi
    loop L1

    ; 反序 pop 並回寫
    mov ecx, nameSize
    mov esi, 0
L2: pop eax               ; 取出（反序）
    mov aName[esi], al
    inc esi
    loop L2

    ; 顯示
    mov edx, OFFSET aName
    call WriteString
    call Crlf
    exit
```

### 8. Related Instructions（相關指令）
| 指令 | 功能 |
|------|------|
| **PUSHFD** | 把 **EFLAGS** push 到 stack（32-bit）|
| **POPFD** | 從 stack pop 到 **EFLAGS** |
| **PUSHAD** | 把 8 個 32-bit 通用暫存器全部 push |
| **POPAD** | 反序 pop 出 |
| **PUSHA / POPA** | 同上，但對 16-bit 暫存器 |

**PUSHAD 的順序**：EAX, ECX, EDX, EBX, ESP, EBP, ESI, EDI

📝 範例：保存 flags 到變數
```asm
.data
saveFlags DWORD ?
.code
pushfd                ; push flags to stack
pop  saveFlags        ; copy to variable

push saveFlags        ; push variable
popfd                 ; copy back to flags
```

---

## 二、Defining and Using Procedures

### 1. 建立 Procedure
- Procedure 就像 Java/C++ 的 function
- **main 也是一種 procedure**，但用 **exit** 結尾

```asm
sample PROC
    .
    .
    ret
sample ENDP
```

### 2. Labels in Procedures（標籤的可見性）
| 類型 | 寫法 | 可見範圍 |
|------|------|---------|
| **Local label** | `Label_name:` | 同一個 procedure |
| **Global label** | `Global_name::`（兩個冒號）| 任何地方都可見 |

📝 範例：
```asm
main PROC
    jmp L2          ; ❌ error！L2 是別的 proc 的 local label
L1::                ; global label（兩個冒號）
    exit
main ENDP

sub2 PROC
L2:                 ; local label
    jmp L1          ; ✅ OK（L1 是 global）
    ret
sub2 ENDP
```

### 3. Documenting Procedures（建議的註解格式）
- **Description**：所做任務的描述
- **Receives**：輸入參數列表
- **Returns**：回傳值
- **Requires**：（可選）前置條件 preconditions

📝 範例：SumOf
```asm
;-----------------------------------------------
SumOf PROC
;
; Calculates and returns the sum of three 32-bit integers.
; Receives: EAX, EBX, ECX, the three integers.
;           May be signed or unsigned.
; Returns:  EAX = sum, and the status flags (Carry, Overflow, etc.)
;           are changed.
; Requires: nothing
;-----------------------------------------------
    add eax, ebx
    add eax, ecx
    ret
SumOf ENDP
```

### 4. CALL 與 RET 指令 ⭐⭐⭐

#### CALL（呼叫 procedure）
1. **把下一條指令的 offset（return address）push 到 stack**
2. **把 called procedure 的位址載入 EIP**

#### RET（從 procedure 返回）
- **從 stack pop 到 EIP**

### 5. CALL-RET 範例

```asm
abcd PROC
    00000020 call MySub         ; CALL：push 25 to stack, EIP=40
    00000025 mov eax, ebx       ; ← return address = 25
    .
abcd ENDP

MySub PROC
    00000040 mov eax, edx       ; 第一條指令
    .
    00000080 ret                ; RET：pop 25 from stack, EIP=25
MySub ENDP
```

📐 圖解：
```
CALL MySub 後：               RET 後：
┌──────────┐                  ┌──────────┐
│ 00000025 │ ← ESP            │ 00000025 │ ← ESP
└──────────┘                  └──────────┘
EIP = 00000040                EIP = 00000025
```

### 6. Nested Procedure Calls（巢狀呼叫）

```asm
main PROC
    call Sub1
    exit
main ENDP

Sub1 PROC
    call Sub2
    ret
Sub1 ENDP

Sub2 PROC
    call Sub3
    ret
Sub2 ENDP

Sub3 PROC
    ret
Sub3 ENDP
```

📐 **Stack 變化**：

**Sub1 被呼叫時**：
```
[(ret to main)]  ← ESP
```

**Sub2 被呼叫時**：
```
[(ret to main)]
[(ret to Sub1)]  ← ESP
```

**Sub3 被呼叫時**：
```
[(ret to main)]
[(ret to Sub1)]
[(ret to Sub2)]  ← ESP
```

**Sub3 ret 後**：
```
[(ret to main)]
[(ret to Sub1)]  ← ESP
```

**Sub2 ret 後**：
```
[(ret to main)]  ← ESP
```

**Sub1 ret 後**：
```
[]  ← ESP
```

🔑 **記憶要點**：每進入一層 procedure 多一個 return address；每 ret 一次取走一個。

### 7. Passing Register Arguments（用 register 傳參）
**最簡單的傳參方式**：用 general-purpose registers

```asm
.data
theSum DWORD ?
.code
main PROC
    mov eax, 10000h       ; argument 1
    mov ebx, 20000h       ; argument 2
    mov ecx, 30000h       ; argument 3
    call SumOf            ; EAX = EAX + EBX + ECX
    mov theSum, eax       ; 存結果
```

### 8. 範例：Summing an Integer Array

#### 版本一：不夠彈性（綁死變數名稱）
```asm
ArraySum PROC
    mov esi, 0
    mov eax, 0
L1: add eax, myArray[esi]     ; 綁死 myArray
    add esi, 4
    loop L1
    mov theSum, eax           ; 綁死 theSum
    ret
ArraySum ENDP
```
> ❌ 缺點：只能加 myArray，無法重用

#### 版本二：彈性版本（用 register 傳參）
```asm
ArraySum PROC
; Receives: ESI 指向 doubleword array
;           ECX = 陣列元素個數
; Returns:  EAX = sum
;-------------------------------------------------
    mov eax, 0
L1: add eax, [esi]
    add esi, 4
    loop L1
    ret
ArraySum ENDP
```

#### 測試程式
```asm
.386
.model flat, stdcall
.stack 4096
ExitProcess PROTO dwExitCode:DWORD

.data
array  DWORD 10000h, 20000h, 30000h, 40000h, 50000h
theSum DWORD ?

.code
Main PROC
    mov esi, OFFSET array       ; ESI 指向 array
    mov ecx, LENGTHOF array     ; ECX = 元素個數
    call ArraySum
    mov theSum, eax
    INVOKE ExitProcess, 0
Main ENDP
```

### 9. Saving and Restoring Registers ⭐
**procedure 應在開頭 push 用到的暫存器、結尾 pop 還原**：
```asm
ArraySum PROC
    push esi
    push ecx
    mov eax, 0
L1: add eax, [esi]
    add esi, 4
    loop L1
    pop ecx
    pop esi
    ret
ArraySum ENDP
```

> 注意 **POP 順序與 PUSH 相反**

### 10. USES Operator（讓 MASM 自動 push/pop）
- 與 PROC 一起使用，列出要保存的暫存器
- MASM 自動生成：
  - 開頭：PUSH 指令
  - 結尾：POP 指令

📝 範例：
```asm
ArraySum PROC USES esi ecx
    mov eax, 0
    ...
    ret
ArraySum ENDP
```

**等同於 MASM 產生**：
```asm
ArraySum PROC
    push esi
    push ecx
    mov eax, 0
    ...
    pop ecx
    pop esi
    ret
ArraySum ENDP
```

---

## 三、Linking to an External Library

### 1. Library 概念
- **Library**：包含已編譯成機器碼的 procedures 的檔案
- 由一個或多個 OBJ 檔組成

### 2. 建立 Library 的流程
1. 從一個或多個 ASM source files 開始
2. 各自組譯成 OBJ files
3. 建立空的 `.LIB` 檔
4. 用 Microsoft **LIB utility** 把 OBJ 加入 library

### 3. Linker 與 Linking 流程

```
   Source Files       ↓
   (.c/.cpp/.s)     Preprocessor
                       ↓
                   Compiler / Assembler
                       ↓
                   Object Files (.o)
                       ↓
   Library Files ──→ Linker ← Linker Command File
   (.a / .lib)         ↓
                  Executable (.exe / .elf / .out)
```

### 4. 函數原型（Function Prototypes）補充
- **是 function 的宣告**：指定名稱、參數、回傳型別
- **不含函式本體**
- 告訴 compiler 該 function 之後會被使用

**語法（C）**：
```c
returnType functionName(type1 arg1, type2 arg2, ...);
```

📝 範例：`int addNumbers(int a, int b);`
- 函式名：`addNumbers`
- 回傳型別：`int`
- 參數：兩個 `int`

> 💡 若 function 定義在 `main()` **之前**，不需要 prototype。

**錯誤示範**：function 寫在 main() 之後，又沒有 prototype
```c
#include <stdio.h>
int main(void) {
    int v = Max(7, 18);       // ⚠️ implicit declaration warning
    printf("Max: %d\n", v);
    return 0;
}
int Max(int n1, int n2) { ... }
```

> 此為舊版 C 行為，**現代 C 編譯器會直接報錯**。

**修正方法**：
1. 在 main() 前加 prototype：`int Max(int n1, int n2);`
2. 或把 Max() 定義移到 main() 之前

### 5. Calling a Library Procedure
- 用 `CALL` 指令
- 語法：`call procedure_name`
- **INCLUDE directive 匯入 function prototypes**

```asm
INCLUDE Irvine32.inc
.code
    mov eax, 1234h        ; input argument
    call WriteHex         ; show hex number
    call Crlf             ; end of line
```

### 6. Linking 指令
```bash
link hello.obj irvine32.lib kernel32.lib
```
- `irvine32.lib`：書本作者提供
- `kernel32.lib`：Microsoft Win32 SDK 的一部分

```
Your program → irvine32.lib → kernel32.lib → kernel32.dll（執行）
```

> 💡 `kernel32.dll` 提供底層功能（CreateFile、VirtualAlloc、CreateProcess、Sleep…）

---

## 四、The Irvine32 Link Library

### 1. 呼叫範例
```asm
INCLUDE Irvine32.inc
.code
    mov eax, 1234h
    call WriteHex         ; 顯示 16 進位
    call Crlf             ; 換行
```

### 2. 完整 procedure 列表

#### Console / 螢幕控制
| Procedure | 功能 |
|-----------|------|
| **Clrscr** | 清螢幕、游標到左上 |
| **Crlf** | 寫出換行符（0Dh + 0Ah）|
| **Gotoxy** | 游標移到指定 row/col |
| **GetMaxXY** | 取得 console window 的行數/列數 |
| **SetTextColor** | 設定前景/背景顏色 |
| **GetTextColor** | 取得目前前景/背景顏色 |

#### 計時與延遲
| Procedure | 功能 |
|-----------|------|
| **Delay** | 暫停執行 n 毫秒（EAX 指定）|
| **GetMseconds** | 取得從午夜起經過的毫秒數 |
| **GetDateTime** | 取得目前日期時間 |

#### 隨機數
| Procedure | 功能 |
|-----------|------|
| **Random32** | 產生 0~FFFFFFFFh 的 32-bit 偽隨機整數 |
| **Randomize** | 設定隨機種子 |
| **RandomRange** | 在指定範圍產生隨機整數 |

#### 輸入（從鍵盤讀取）
| Procedure | 功能 |
|-----------|------|
| **ReadChar** | 讀一個字元 |
| **ReadDec** | 讀 32-bit unsigned 十進位 |
| **ReadHex** | 讀 32-bit 16 進位 |
| **ReadInt** | 讀 32-bit signed 十進位 |
| **ReadKey** | 讀鍵盤緩衝區字元 |
| **ReadString** | 讀字串（Enter 結束）|

#### 輸出（寫到 console）
| Procedure | 功能 |
|-----------|------|
| **WriteBin** | 寫 unsigned 32-bit ASCII binary |
| **WriteBinB** | 寫 byte/word/dword binary |
| **WriteChar** | 寫單一字元 |
| **WriteDec** | 寫 unsigned 32-bit 十進位 |
| **WriteHex** | 寫 unsigned 32-bit 16 進位 |
| **WriteHexB** | 寫 byte/word/dword 16 進位 |
| **WriteInt** | 寫 signed 32-bit 十進位 |
| **WriteString** | 寫 null-terminated 字串 |

#### 字串
| Procedure | 功能 |
|-----------|------|
| **Str_compare** | 比較兩字串 |
| **Str_copy** | 複製字串 |
| **Str_length** | 回傳長度（EAX）|
| **Str_trim** | 移除不需要的字元 |
| **Str_ucase** | 轉為大寫 |

#### 檔案 I/O
| Procedure | 功能 |
|-----------|------|
| **OpenInputFile** | 開啟既存檔案用於讀取 |
| **CreateOutputFile** | 建立新檔案用於寫入 |
| **ReadFromFile** | 讀檔到 buffer |
| **WriteToFile** | 寫 buffer 到檔案 |
| **CloseFile** | 關檔 |

#### 記憶體與除錯
| Procedure | 功能 |
|-----------|------|
| **DumpMem** | 以 16 進位顯示記憶體區塊 |
| **DumpRegs** | 顯示 EAX/EBX/.../EIP/EFLAGS + CF/SF/ZF/OF（除錯用）|
| **WriteStackFrame** | 顯示當前 procedure 的 stack frame |
| **WriteStackFrameName** | 顯示名稱 + stack frame |

#### 雜項
| Procedure | 功能 |
|-----------|------|
| **GetCommandtail** | 取得程式啟動的命令列參數（command tail）|
| **IsDigit** | 若 AL 是 ASCII '0'~'9'，設 ZF |
| **MsgBox / MsgBoxAsk** | 彈出訊息視窗 |
| **ParseDecimal32** | unsigned 整數字串 → binary |
| **ParseInteger32** | signed 整數字串 → binary |
| **WaitMsg** | 顯示訊息並等待 Enter |
| **WriteWindowsMsg** | 顯示 Windows 最近錯誤訊息 |

### 3. 細節範例

#### Delay（暫停 1 秒）
```asm
mov eax, 1000        ; 1 秒
call Delay
```

#### DumpMem（傾印記憶體）
```asm
.data
array DWORD 1,2,3,4,5,6,7,8,9,0Ah,0Bh
.code
mov esi, OFFSET array       ; 起始位址
mov ecx, LENGTHOF array     ; 單位個數
mov ebx, TYPE array         ; 單位大小
call DumpMem
; 輸出：00000001 00000002 ... 0000000B
```

#### GetCommandtail
```bash
d:> copy file1.txt file2.txt    # ← command tail: "file1.txt file2.txt"
```
```asm
.data
cmdTail BYTE 129 DUP(0)
.code
mov edx, OFFSET cmdTail
call GetCommandtail
```

#### Gotoxy
```asm
mov dh, 10        ; row 10
mov dl, 20        ; column 20
call Gotoxy
```

#### Random32 / Randomize
```asm
.data
randVal DWORD ?
.code
call Random32
mov randVal, eax

; 加入隨機種子
call Randomize
mov ecx, 10
L1: call Random32
    ; ... 使用或顯示
    loop L1
```

#### RandomRange
```asm
.data
randVal DWORD ?
.code
mov eax, 5000        ; 範圍上限
call RandomRange     ; 產生 0~4999
mov randVal, eax
```

#### ReadChar / ReadDec / ReadHex / ReadInt
```asm
.data
char    BYTE ?
hexVal  DWORD ?
intVal  SWORD ?
.code
call ReadChar
mov char, al        ; AL = 字元

call ReadHex
mov hexVal, eax     ; EAX = 16 進位

call ReadInt
mov intVal, eax     ; EAX = signed
```

#### ReadString
```asm
.data
buffer    BYTE 50 DUP(0)
byteCount DWORD ?
.code
mov edx, OFFSET buffer      ; 緩衝區位址
mov ecx, SIZEOF buffer      ; 最大字元數
call ReadString
mov byteCount, eax          ; 實際輸入字元數
```

#### SetTextColor 顏色表
| 0 black | 4 red | 8 gray | 12 lightRed |
| 1 blue | 5 magenta | 9 lightBlue | 13 lightMagenta |
| 2 green | 6 brown | 10 lightGreen | 14 yellow |
| 3 cyan | 7 lightGray | 11 lightCyan | 15 white |

📝 黃字藍底：
```asm
.data
str1 BYTE "Color output is easy!", 0
.code
mov eax, yellow + (blue * 16)   ; 背景 × 16 + 前景
call SetTextColor
mov edx, OFFSET str1
call WriteString
call Crlf
```

🔑 **背景顏色要 × 16 才能加上前景**！

### 4. 綜合範例

#### Example 1：清螢幕、延遲 500ms、傾印暫存器
```asm
.code
call Clrscr
mov eax, 500
call Delay
call DumpRegs
```
**範例輸出**：
```
EAX=00000613 EBX=00000000 ECX=000000FF EDX=00000000
ESI=00000000 EDI=00000100 EBP=0000091E ESP=000000F6
EIP=00401026 EFL=00000286 CF=0 SF=1 ZF=0 OF=0
```

#### Example 2：印字串並換行
```asm
.data
str1 BYTE "Assembly language is easy!", 0

.code
mov edx, OFFSET str1
call WriteString
call Crlf
```

#### Example 3：以 binary / decimal / hex 顯示同一個數
```asm
IntVal = 35
.code
mov eax, IntVal
call WriteBin
call Crlf
call WriteDec
call Crlf
call WriteHex
call Crlf
```
**輸出**：
```
0000 0000 0000 0000 0000 0000 0010 0011
35
23
```

#### Example 4：從鍵盤讀字串
```asm
.data
fileName BYTE 80 DUP(0)

.code
mov edx, OFFSET fileName
mov ecx, SIZEOF fileName - 1   ; 留 1 byte 給 null
call ReadString
```

#### Example 5：產生並顯示 10 個 0~99 隨機數
```asm
.code
mov ecx, 10                ; 迴圈計數
L1: mov eax, 100            ; 範圍上限
    call RandomRange       ; 產生 0~99
    call WriteInt          ; 印出 signed
    call Crlf
    loop L1
```

#### Example 6：黃字藍底
```asm
.data
str1 BYTE "Color output is easy!", 0

.code
mov eax, yellow + (blue * 16)
call SetTextColor
mov edx, OFFSET str1
call WriteString
call Crlf
```

---

# 🔥 速背重點整理（考前衝刺）

## ⭐ Stack 基本概念
- 由 **SS + ESP** 管理
- 向下成長（往 **較低位址**）
- **不要直接動 SS/ESP**，用 push/pop/call/ret

## ⭐ PUSH / POP 步驟
| 操作 | Step 1 | Step 2 |
|------|--------|--------|
| **PUSH** | ESP −= 4/2 | 寫入 [ESP] |
| **POP** | 讀取 [ESP] | ESP += 4/2 |

🔑 順序：PUSH 先減後寫，POP 先讀後加。

## ⭐ PUSH/POP 規則
- POP 沒有 immediate 版本
- PUSH 與對應 POP 順序 **相反**（LIFO）

## ⭐ 特殊 PUSH/POP 指令
| 指令 | 功能 |
|------|------|
| **PUSHFD/POPFD** | EFLAGS（32-bit）|
| **PUSHAD/POPAD** | 8 個 32-bit 通用暫存器 |
| **PUSHA/POPA** | 16-bit 版本 |

> PUSHAD 順序：**EAX, ECX, EDX, EBX, ESP, EBP, ESI, EDI**

## ⭐ CALL 與 RET（必背）
| 指令 | 動作 |
|------|------|
| **CALL** | ① push 下條指令 offset（return addr）<br>② EIP ← 被呼叫程序位址 |
| **RET** | pop top of stack → EIP |

## ⭐ Labels 可見性
| 類型 | 寫法 | 範圍 |
|------|------|------|
| Local | `Label:` | 同 procedure |
| Global | `Label::`（雙冒號）| 全程式 |

## ⭐ Procedure 範本
```asm
SumOf PROC
;-----------------------------------------------
; Description: ...
; Receives: EAX, EBX, ECX
; Returns:  EAX = sum
; Requires: nothing
;-----------------------------------------------
    add eax, ebx
    add eax, ecx
    ret
SumOf ENDP
```

## ⭐ USES Operator
```asm
ArraySum PROC USES esi ecx
    ...
    ret
ArraySum ENDP
```
> MASM 自動在開頭 PUSH、結尾 POP（順序相反）

## ⭐ 傳參數方式
- 用 **general-purpose registers** 傳 arguments
- 用 **EAX 回傳** 結果（慣例）
- procedure 中用到的 register 應 **push 保存 → pop 還原**

## ⭐ Nested Procedure Calls 規則
- 每進入一層 → stack 多一個 return address
- 每 RET 一次 → 取走一個
- ESP 永遠指向最上層的 return address

## ⭐ Library 流程
```
ASM source → OBJ → LIB → 連結 → EXE
```
連結指令：`link hello.obj irvine32.lib kernel32.lib`

## ⭐ 使用 Irvine32 必需
1. `INCLUDE Irvine32.inc` 取得 prototypes
2. `call XXX` 呼叫函式
3. 連結時加上 `irvine32.lib` 與 `kernel32.lib`

## ⭐ 常用 Irvine32 函式（必背）
| 函式 | 輸入 | 輸出 |
|------|------|------|
| **Clrscr** | – | 清螢幕 |
| **Crlf** | – | 換行 |
| **Delay** | EAX = 毫秒 | – |
| **DumpRegs** | – | 顯示暫存器 |
| **DumpMem** | ESI=位址、ECX=個數、EBX=size | – |
| **Gotoxy** | DH=row, DL=col | 移動游標 |
| **ReadChar** | – | AL |
| **ReadDec** | – | EAX (unsigned) |
| **ReadHex** | – | EAX (hex) |
| **ReadInt** | – | EAX (signed) |
| **ReadString** | EDX=buf, ECX=max | EAX=count |
| **WriteChar** | AL | – |
| **WriteDec** | EAX | – |
| **WriteHex** | EAX | – |
| **WriteInt** | EAX | – |
| **WriteString** | EDX=null-term str | – |
| **WriteBin** | EAX | – |
| **Random32** | – | EAX |
| **RandomRange** | EAX=上限 | EAX (0~上限-1) |
| **Randomize** | – | 設定種子 |
| **SetTextColor** | EAX=前景+背景×16 | – |

## ⭐ SetTextColor 公式
`EAX = foreground + (background × 16)`

## ⭐ Function Prototype（補充）
- 宣告 function（不含 body）
- 告訴 compiler「之後會用」
- 若 function 定義在 main 之前 → **不需要 prototype**

## ⭐ Reversing String 經典範例
```
push every character → loop ECX times
pop every character (反序) → 覆寫回字串
```

## ⭐ Stack 應用情境（4 種）
1. 暫存暫存器
2. 儲存 return address（call/ret 自動處理）
3. 傳遞 arguments
4. 配置 local variables

## ⭐ 兩種 .data 的差別（複習）
- `.data`：初始化資料、EXE 變大
- `.data?`：未初始化、EXE 不變大

## ⭐ 容易忘記的小細節
- `END main` 中 `main` 是 startup procedure（可改名）
- `main` 用 `exit` 結尾、其他 procedure 用 `ret`
- INCLUDE Irvine32.inc → 引入 prototypes
- Irvine32.lib → 提供實作
- kernel32.lib → 由 Microsoft Win32 SDK 提供，底層作業系統呼叫
