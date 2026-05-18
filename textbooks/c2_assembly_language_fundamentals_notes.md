# Chapter 2：Assembly Language Fundamentals 完整筆記

> 課程主軸：MASM 組合語言的基本元素、程式結構、組譯流程、資料定義、符號常數

---

## 📚 目錄
1. [Basic Elements of Assembly Language](#一basic-elements-of-assembly-language)
2. [Example: Adding and Subtracting Integers](#二example-adding-and-subtracting-integers)
3. [Assembling, Linking, and Running Programs](#三assembling-linking-and-running-programs)
4. [Defining Data](#四defining-data)
5. [Symbolic Constants](#五symbolic-constants)
6. [速背重點整理](#速背重點整理)

---

## 一、Basic Elements of Assembly Language

### 1. Integer Literals（整數字面值/常數）
- **Literals** 又稱 **constant**（常數）
- **語法**：`[{+|-}] digits [radix]`
  - 可選的 `+` 或 `−` 符號
  - digits 可以是 binary、decimal、hexadecimal、octal 數字

| Radix 字尾 | 意義 |
|------------|------|
| **h** | hexadecimal（16 進位）|
| **d** | decimal（10 進位）|
| **b** | binary（2 進位）|
| **r** | encoded real（編碼浮點數）|

📝 範例：`30d`, `6Ah`, `42`, `1101b`

⚠️ **重要規則**：
- 以**字母開頭** 的 hex 數字 **必須加前綴 0**
- 例：`A5h` ❌ → 正確寫法為 `0A5h` ✅
- 原因：避免 assembler 把它誤認為 **identifier**（識別字 / 變數 / 標籤）

### 2. Integer Expressions（整數運算式）

| 運算子 | 名稱 | 優先序 |
|--------|------|--------|
| `( )` | 括號 | 1 |
| `+`, `−` | 一元（unary）+/− | 2 |
| `*`, `/` | 乘 / 除 | 3 |
| `MOD` | 取餘數 | 3 |
| `+`, `−` | 加 / 減 | 4 |

📝 範例：

| 運算式 | 結果 |
|--------|------|
| `16 / 5` | 3 |
| `-(3 + 4) * (6 - 1)` | −35 |
| `-3 + 4 * 6 - 1` | 20 |
| `25 mod 3` | 1 |

### 3. Real Number Literals（跳過）
- 格式：`[sign] integer.[integer [exponent]]`
- 範例：`2.`、`+3.0`、`-44.2E+05`、`26.E5`
- 注意：**必須有數字＋小數點**

### 4. Character & String Literals
- **Character literal**（字元）：用單引號或雙引號包住 → 例 `'A'`、`"x"`，1 ASCII 字元 = 1 byte
- **String literal**（字串）：例 `"ABC"`、`'xyz'`，每個字元 1 byte
- **內嵌引號**：`'Say "Goodnight," Gracie'`

### 5. Reserved Words（保留字）
**不能** 作為 identifier 使用，包含：
- Instruction mnemonics（指令助記符）：ADD、MOV…
- **Directives**（指示詞）：告訴 MASM 如何組譯
- **Type attributes**（型別屬性）：BYTE、WORD…
- **Operators**（運算子）：+、−…
- **Predefined symbols**（預定義符號）：@data…

### 6. Identifiers（識別字）
- **定義**：使用者自訂的名稱（記憶體位置或函式）
- **規則**：
  - 1–247 個字元，包含數字
  - **Case insensitive**（不分大小寫，依組譯器而定）
  - **首字** 必須是字母、`_`、`@` 或 `$`

### 7. Directives（指示詞）
- 由 **assembler** 識別並執行的命令
- **不是** Intel 指令集的一部分
- **執行期不會跑**（不是指令）
- Case insensitive
- 不同組譯器有不同 directives（NASM ≠ MASM）

📝 範例：
```asm
.stack 100h
.data
message db 'Hello, world!$'
.code
start:
    mov ah, 09h
    lea dx, message
    int 21h
    mov ax, 4C00h
    int 21h
```

### 8. Instructions（指令）
- 由 assembler 組譯為 **machine code**
- 由 **CPU** 在執行期執行
- **四個部分**：

```
Label: Mnemonic  Operand(s) ; Comment
```

| 部分 | 是否必須 |
|------|---------|
| Label（標籤）| optional |
| Instruction Mnemonic（指令助記符）| **required** |
| Operand（運算元）| 依指令而定 |
| Comment（註解）| optional |

### 9. Labels（標籤）
- 作為 **place markers**（位置標記）
- 標記程式碼或資料的 **address (offset)**
- 遵循 identifier 規則
- **兩種類型**：
  - **Data label**：例 `first BYTE 10`
  - **Code label**：jump / loop 指令的目標

📝 範例：
```asm
target: mov ax, bx
...
        jmp target
```

### 10. Mnemonics & Operands
- **Instruction Mnemonics**：簡短表示指令的英文字（MOV、ADD、SUB、MUL、INC、DEC）
- **Operands**：指令可以有 **0~3 個** 運算元

| 類型 | 範例 |
|------|------|
| **constant**（立即值）| `mov ax, 96` |
| **constant expression**（立即值）| `mov ax, 5+4` |
| **register**（暫存器）| `mov ax, bx` |
| **memory**（資料標籤）| `mov count, bx` |

> Constants 與 constant expressions 統稱為 **immediate values**（立即值）

### 11. Comments（註解）
- **單行註解**：以分號 `;` 開頭
- **多行註解**：以 `COMMENT` directive 開頭，搭配程式員選定的字元（前後相同）

```asm
COMMENT !
   .....
   .....
!
```

### 12. Instruction Format Examples（指令格式範例）

| 運算元數量 | 範例 |
|-----------|------|
| **無 operand** | `stc` ; set Carry flag |
| **一個 operand** | `inc eax` ; register<br>`inc myByte` ; memory |
| **兩個 operands** | `add ebx, ecx` ; reg, reg<br>`sub myByte, 25` ; mem, const<br>`add eax, 36 * 25` ; reg, expression |

### 13. NOP（No Operation）指令
- 佔 **1 byte**，**不做任何事**
- 用途：把程式碼 **align（對齊）** 到 **even-address boundaries**（偶位址邊界）

📝 範例：
```
00000000  mov ax, bx
00000003  nop
00000004  mov edx, ecx   ; 對齊到 doubleword 邊界
```

> IA-32 從 **even doubleword address** 載入程式碼/資料較快。

### 14. Aligned vs Unaligned 範例對比

**Case 1：Aligned（對齊）**
```
00000000  mov ax, bx    ; 3 bytes
00000003  nop           ; 1 byte
00000004  mov edx, ecx  ; 3 bytes
```

**Case 2：Unaligned（未對齊）**
```
00000000  mov ax, bx    ; 3 bytes
00000003  mov edx, ecx  ; 3 bytes
```

> 兩種情況都要 2 次 memory access，但 Case 1 **更有效率**。

### 15. 為什麼 Alignment 提升效能？

🔑 **兩大原因**：
1. **未對齊存取會打亂 instruction prefetching 與 pipeline execution**
   - 若指令橫跨 64-byte cache line 邊界（如從 0x003E 開始），CPU 必須抓兩條 cache line 才能解碼一條指令 → **pipeline stall**
2. **對齊指令減少 CPU memory access cycles，提升 cache 效率**
   - 從對齊位址（0x0004）讀 4-byte int：只需 1 cycle
   - 從未對齊位址（0x0003）讀：要 2 cycles 還要 merge

📝 **練習解答**（Exercise）：
- **−35** 的 MASM 寫法：
  - decimal：`-35`
  - hexadecimal：`-23h` 或 `-0x23`
  - binary：`-00100011b` 或 `-0b100011`
- `A5h` 是有效的 hex literal 嗎？→ **否**（須寫成 `0A5h`）
- 💡 程式碼中的字面值通常以人類可讀方式表示數值，不直接用補數表示。

---

## 二、Example: Adding and Subtracting Integers

### 1. 完整範例程式碼
```asm
; AddTwo.asm - adds two 32-bit integers

.386
.model flat, stdcall
.stack 4096
ExitProcess PROTO, dwExitCode:DWORD
.code
main PROC
    mov eax, 5      ; move 5 to the EAX register
    add eax, 6      ; add 6 to the EAX register

    INVOKE ExitProcess, 0
main ENDP
END main
```

### 2. 各 directive 詳解

| Directive | 意義 |
|-----------|------|
| **`.386`** | 80386 處理器，表示這是 **32-bit program** |
| **`.model flat, stdcall`** | 設定記憶體模型為 **flat**；procedure 呼叫慣例為 **stdcall** |
| **`.stack 4096`** | 設定 stack segment 大小為 4096 bytes |
| **`ExitProcess PROTO, dwExitCode:DWORD`** | 宣告 Windows 函式 ExitProcess 的 **function prototype**，接受 DWORD 作為 exit code |
| **`.code`** | 標記 **code segment** 起點 |
| **`PROC`** | 標記 procedure 起點 |
| **`mov eax, 10000h`** | 第一個 operand = destination；第二個 = source |
| **`INVOKE ExitProcess, 0`** | 呼叫 Windows 函式並回傳 0 |
| **`ENDP`** | 標記 main procedure 結尾 |
| **`END main`** | 標記程式結尾，**並指定 startup procedure 名稱** |

### 3. Startup Procedure 命名可自訂

📝 範例（startup 改為 xyz）：
```asm
.code
xyz PROC
    mov eax, 5
    add eax, 6
    INVOKE ExitProcess, 0
xyz ENDP
END xyz       ; ← 改成 xyz
```

### 4. Code / Data / Stack Segment

| Segment | 內容 | 對應暫存器 |
|---------|------|-----------|
| **Code segment** | 執行的程式碼 | **CS** |
| **Data segment** | 全域變數 | **DS** |
| **Stack segment** | 區域變數、參數、回傳位址 | **SS** |

### 5. Virtual Memory 圖解
```
        Memory（虛擬記憶體）
       ┌────────────────────┐
SS:BP  │   Used Stack       │
SS:SP  │   Unused Stack     │  ← Stack（SS）
       │         ↓          │
       │                    │
       │         ↑          │
CS:IP  │   Code (Program)   │  ← CS
DS:DI  │   Data (Variables) │  ← DS
DS:SI  │                    │
       └────────────────────┘
```

### 6. 執行中程式的記憶體佈局
```
max ┌──────────┐
    │  stack   │   ← 向下成長
    │    ↓     │
    │          │
    │    ↑     │
    │   heap   │   ← 向上成長
    │   data   │
    │   text   │
  0 └──────────┘
```

### 7. Program Template（程式模板）
```asm
; Program template (Template.asm)

.386
.model flat, stdcall
.stack 4096
ExitProcess PROTO, dwExitCode:DWORD
.code
main PROC
    ; write your code here

    INVOKE ExitProcess, 0
main ENDP
END main
```

---

## 三、Assembling, Linking, and Running Programs

### Assemble-Link Execute Cycle（組譯-連結-執行流程）

```
                         ┌───────────┐
                         │   Link    │
                         │  Library  │
                         └─────┬─────┘
                               │
┌────────┐  Step 2:    ┌──────▼──────┐  Step 3:  ┌───────────┐  Step 4:  ┌────────┐
│ Source │ ──assembler→│  Object     │ ──linker→ │Executable │ OS loader→│ Output │
│  File  │             │  File       │           │   File    │           │        │
└───▲────┘             ├─────────────┤           ├───────────┤           └────────┘
    │                  │ Listing File│           │  Map File │
Step 1: text editor    └─────────────┘           └───────────┘
```

> ⚠️ **若 source code 被修改，Step 2 ~ 4 必須重複執行**。

### Listing File（略讀）
- 用於查看程式如何被組譯
- 包含：source code、addresses、object code（機器碼）、segment names、symbols

### Map File（略讀）
- 描述每個 segment：starting address、ending address、size、type

---

## 四、Defining Data

### 1. Intrinsic Data Types（內建資料型別）

| 類型 | 大小 | 用途 |
|------|------|------|
| **BYTE** | 8-bit | unsigned 整數 |
| **SBYTE** | 8-bit | **signed** 整數 |
| **WORD** | 16-bit | unsigned |
| **SWORD** | 16-bit | **signed** |
| **DWORD** | 32-bit | unsigned |
| **SDWORD** | 32-bit | **signed** |
| **QWORD** | 64-bit | 整數 |
| **TBYTE** | 80-bit | 整數 |
| **REAL4** | 4 bytes | IEEE short real |
| **REAL8** | 8 bytes | IEEE long real |
| **REAL10** | 10 bytes | IEEE extended real |

🔑 **背口訣**：S 前綴 = **Signed**（有號）；無 S = unsigned。

### 2. Data Definition Statement（資料定義語法）
- 用途：為變數 **配置記憶體**
- 語法：`[name] directive initializer [,initializer]...`

📝 範例：`value1 BYTE 10` → 配置 1 byte，初值為 10 → 二進位 `00001010`

### 3. 範例：AddVariables.asm
```asm
.386
.model flat, stdcall
.stack 4096
ExitProcess PROTO, dwExitCode:DWORD
.data
sum     DWORD 0           ; 在 data segment 宣告變數
.code
main PROC
    mov eax, 5
    add eax, 6
    mov sum, eax          ; 把 eax 寫入 sum
    INVOKE ExitProcess, 0
main ENDP
END main
```

### 4. 定義 BYTE / SBYTE
```asm
value1 BYTE  'A'      ; character constant
value2 BYTE  0        ; smallest unsigned byte
value3 BYTE  255      ; largest unsigned byte
value4 SBYTE -128     ; smallest signed byte
value5 SBYTE +127     ; largest signed byte
value6 BYTE  ?        ; uninitialized byte（?表示未初始化）
```

### 5. 變數名稱 = label（offset）
- 變數名稱是 label，標記變數在 segment 內的 **offset**

📝 範例：
```asm
.data
value1 BYTE 10h   ; offset 0（低位址）
value2 BYTE 20h   ; offset 1（高位址）
```

```
        Data Segment
        ┌─────────┐
低位址  │   10h   │  ← value1
        ├─────────┤
高位址  │   20h   │  ← value2
        └─────────┘
```

### 6. Defining Byte Arrays（位元組陣列）
```asm
list1 BYTE 10,20,30,40
list2 BYTE 10,20,30,40
      BYTE 50,60,70,80
      BYTE 81,82,83,84
list3 BYTE ?,32,41h,00100010b
list4 BYTE 0Ah,20h,'A',22h
```

📐 圖解（list1 的記憶體配置）：
```
        Offset    Value
低位址   0000      10    ← list1 指向這裡
         0001      20
         0002      30
高位址   0003      40
```

> 📌 **重點**：label 指向陣列的 **第一個 byte 的 offset**。

### 7. Defining Strings（字串定義）
- 字串實作為 **字元陣列**
- 通常用引號包起來
- 常以 **null-terminated**（`,0` 結尾），讓 `printf`、`strlen` 等知道字串結尾

```asm
str1 BYTE "Enter your name",0
str2 BYTE 'Error: halting program',0
str3 BYTE 'A','E','I','O','U'
greeting1 BYTE "Welcome to the Encryption Demo program "
          BYTE "created by Kip Irvine.",0
greeting2 \
    BYTE "Welcome to the Encryption Demo program "
    BYTE "created by Kip Irvine.",0
```

#### End-of-line 字元（重要）
| Hex | 意義 |
|-----|------|
| **0Dh** | **carriage return**（回車） |
| **0Ah** | **line feed**（換行） |

```asm
str1 BYTE "Enter your name:    ",0Dh,0Ah
     BYTE "Enter your address: ",0
newLine BYTE 0Dh,0Ah,0
```

### 8. DUP Operator（重複配置）
- 語法：`counter DUP (argument)`
- **counter 與 argument 必須為常數或常數運算式**

```asm
var1 BYTE 20 DUP(0)         ; 20 bytes，全為 0
var2 BYTE 20 DUP(?)         ; 20 bytes，未初始化
var3 BYTE 4 DUP("STACK")    ; 20 bytes："STACKSTACKSTACKSTACK"
var4 BYTE 10, 3 DUP(0), 20  ; 5 bytes
```

### 9. WORD / SWORD（16-bit）
```asm
word1 WORD  65535         ; largest unsigned
word2 SWORD -32768        ; smallest signed
word3 WORD  ?             ; uninitialized
word4 WORD  "AB"          ; double characters
myList WORD 1,2,3,4,5     ; array of words
array WORD  5 DUP(?)
```

📐 myList 的 offset（每個元素 **2 bytes**）：
```
Offset    Data
0000      1     ← myList
0002      2
0004      3
0006      4
0008      5
```

### 10. DWORD / SDWORD（32-bit）
```asm
val1 DWORD 12345678h          ; unsigned
val2 SDWORD -2147483648       ; signed
val3 DWORD 20 DUP(?)          ; unsigned array
val4 SDWORD -3,-2,-1,0,1      ; signed array

myList DWORD 1, 2, 3, 4, 5
```

📐 myList 的 offset（每個元素 **4 bytes**）：
```
Offset    Data
0000      1
0004      2
0008      3
000C      4
0010      5
```

### 11. QWORD / TBYTE / REAL
```asm
quad1 QWORD  1234567812345678h
val1  TBYTE  1000000000123456789Ah
rVal1 REAL4  -2.1
rVal2 REAL8  3.2E-260
rVal3 REAL10 4.6E+4096
ShortArray REAL4 20 DUP(0.0)
```

### 12. Little Endian vs Big Endian ⭐
**問題**：超過一個 byte 的資料，要怎麼擺進記憶體？

📝 範例：`val1 DWORD 12345678h`

#### Little Endian（x86 採用）⭐
- **最低有效 byte 放在最低位址**

```
Offset    Value
0000      78    ← LSB
0001      56
0002      34
0003      12    ← MSB
```

#### Big Endian
- **最高有效 byte 放在最低位址**

```
Offset    Value
0000      12    ← MSB
0001      34
0002      56
0003      78    ← LSB
```

🔑 **記憶口訣**：
- **Little Endian** → **L**SB at **L**owest address（x86 用）
- **Big Endian** → MSB at lowest address

### 13. .data?（未初始化資料）
- 用 **`.data?` directive** 宣告未初始化資料段

```asm
.data
smallArray DWORD 10 DUP(5)    ; 40 bytes（初始化）

.data?
bigArray   DWORD 5000 DUP(?)  ; 20000 bytes（未初始化）
```

#### 為什麼要分開？
- 若全部寫在 `.data`，編譯出的 **.exe 檔多 20000 bytes**
- 用 `.data?` 在 EXE 中只是 **reserve**，**檔案不變大**
- ⚠️ 但載入記憶體後兩種寫法佔用大小相同（20040 bytes）

📐 **圖解**：
```
(a) 不用 .data?              (b) 用 .data?
┌────────────┐               ┌────────────┐
│ 5          │ ┐             │ 5          │ ┐
│ 5          │ │ 40 bytes    │ 5          │ │ 40 bytes
│ ...        │ │             │ ...        │ │
│ 5          │ ┘             │ 5          │ ┘
│ ?          │ ┐             │ Reserve    │ x bytes
│ ?          │ │ 20000 bytes │ 20000 bytes│   (小檔)
│ ...        │ │             └────────────┘
│ ?          │ ┘
└────────────┘
   EXE 大            EXE 小
```

🔑 **優勢**：`.data?` **降低 EXE 檔案大小**。

📝 **練習解答**：
- 16-bit signed integer 未初始化：`mySigned16 SWORD ?`
- 8-bit unsigned integer 未初始化：`myUnsigned8 BYTE ?`
- 8-bit signed integer 未初始化：`mySigned8 SBYTE ?`
- 哪個型別可儲存 32-bit signed integer？→ **SDWORD**

---

## 五、Symbolic Constants（符號常數）

### 1. 定義
- 將 **identifier（symbol）** 與 **整數運算式或文字** 連結
- **不佔記憶體**
- 只在 **assembly 階段** 使用
- **不能在 runtime 改變**

### 2. 三種方法
| Directive | 用途 |
|-----------|------|
| **`=`** Equal-Sign | 表示 **integer** 常數 |
| **`EQU`** | 整數 / 文字（**不可** 重新定義） |
| **`TEXTEQU`** | 文字 macro（**可** 重新定義） |

### 3. Equal-Sign Directive
- 語法：`name = expression`
- expression 為 32-bit 整數
- name 稱為 **symbolic constant**
- 組譯時，所有 name 都會被替換為 expression

📝 範例：
```asm
COUNT = 500
...
mov ax, COUNT       →  mov ax, 500
```

#### 為何使用 symbols？
- 程式更易讀、易維護
- 良好程式風格

#### Redefinition（可重新定義）
```asm
COUNT = 5
mov al, count       ; al = 5
COUNT = 10
mov al, count       ; al = 10
```

### 4. Current Location Counter `$` ⭐
- **重要符號**：`$`
- 回傳 **當前程式敘述的 offset**

📝 範例：
```asm
selfPtr DWORD $    ; 宣告 selfPtr 並將其 offset 寫入自己
```

### 5. 計算陣列大小（必考）

#### 計算 Byte Array
```asm
List BYTE 10, 20, 30, 40
ListSize = ($ - list)        ; 結果 = 4 bytes
```
🔑 **ListSize 必須緊跟在 List 後面**！

#### 計算 Word Array（÷ 2）
```asm
list WORD 1000h,2000h,3000h,4000h
ListSize = ($ - list) / 2     ; 結果 = 4 個 word
```
> 因為 `$` 回傳 **bytes** 為單位，但每個 WORD 元素是 2 bytes。

#### 計算 Doubleword Array（÷ 4）
```asm
list DWORD 1, 2, 3, 4
ListSize = ($ - list) / 4     ; 結果 = 4 個 dword
```

### 6. EQU Directive
- 定義符號為 **整數** 或 **文字運算式**
- **不能重新定義**

語法：
```asm
name  EQU  expression
name  EQU  symbol        ; 該 symbol 必須先用 = 或 EQU 定義
name  EQU  <text>        ; 文字以 < > 包起來
```

📝 範例：
```asm
PI       EQU <3.1416>
pressKey EQU <"Press any key to continue...",0>
.data
prompt   BYTE pressKey
```

#### EQU：整數 vs 文字
```asm
matrix1 EQU 10*10        ; assembler 會計算為 100
matrix2 EQU <10*10>      ; 直接複製文字 10*10
.data
M1 WORD matrix1          ; → M1 WORD 100
M2 WORD matrix2          ; → M2 WORD 10*10
```

| 寫法 | 結果 |
|------|------|
| `EQU 10*10`（無 `<>`）| 整數運算式被 assembler **計算** |
| `EQU <10*10>`（有 `<>`）| 文字 **直接複製** |

### 7. TEXTEQU Directive
- 定義 symbol 為 **text macro**（文字巨集）
- 使用時被 **替換為定義的文字**
- ⭐ **可以重新定義**

語法：
```asm
name TEXTEQU <text>
name TEXTEQU textmacro    ; 之前 TEXTEQU 定義的符號
name TEXTEQU %constExpr   ; % 表示計算運算式
```

📝 範例 1：
```asm
continueMsg TEXTEQU <"Do you wish to continue (Y/N)?">
.data
prompt1 BYTE continueMsg
```

📝 範例 2：
```asm
rowSize = 5
count   TEXTEQU %(rowSize * 2)    ; 用 % 計算運算式
move    TEXTEQU <mov>
setupAL TEXTEQU <move al, count>

; 等同於
setupAL TEXTEQU <mov al, 10>
```

---

# 🔥 速背重點整理（考前衝刺）

## ⭐ Integer Literal 規則
- **Radix 字尾**：`h` Hex、`d` Dec、`b` Bin、`r` Real
- **字母開頭 hex 必須加 0**：`A5h` ❌ → `0A5h` ✅
- −35 三種寫法：`-35`、`-23h`、`-00100011b`

## ⭐ MASM 運算子優先序（由高到低）
1. `( )`
2. unary `+/-`
3. `* /` MOD
4. `+ -`（加減）

## ⭐ 指令完整格式
```
Label: Mnemonic  Operand(s) ; Comment
       ↑必須
```

## ⭐ Identifier 規則
- 1–247 字元
- 首字：字母、`_`、`@`、`$`
- 不分大小寫

## ⭐ 三大 Segment ↔ 暫存器
| Segment | 內容 | 暫存器 |
|---------|------|--------|
| Code | 程式碼 | **CS** |
| Data | 全域變數 | **DS** |
| Stack | 區域變數、參數、return addr | **SS** |

## ⭐ Program Template（必背）
```asm
.386
.model flat, stdcall
.stack 4096
ExitProcess PROTO, dwExitCode:DWORD
.code
main PROC
    ; code here
    INVOKE ExitProcess, 0
main ENDP
END main
```

| Directive | 一句話 |
|-----------|--------|
| `.386` | 32-bit 程式 |
| `.model flat, stdcall` | 記憶體模型 + 呼叫慣例 |
| `.stack 4096` | stack 大小 |
| `.code` | 程式碼段起點 |
| `PROC` / `ENDP` | procedure 起 / 終 |
| `END xxx` | 程式結尾 + 指定 startup procedure |
| `INVOKE` | 呼叫 Windows 函式 |

## ⭐ Assemble-Link 流程（必背）
```
Source → (assembler) → Object → (linker) → Executable → (OS loader) → Output
                          ↓                     ↓
                      Listing               Map File
```
> 修改 source code 後，**Step 2~4 都要重做**。

## ⭐ Intrinsic Data Types
| Type | 大小 | Type | 大小 |
|------|------|------|------|
| BYTE / SBYTE | 8 bits | QWORD | 64 bits |
| WORD / SWORD | 16 bits | TBYTE | 80 bits |
| DWORD / SDWORD | 32 bits | REAL4/8/10 | 4/8/10 bytes |

🔑 **S 前綴 = Signed**

## ⭐ String 結尾字元
- `0Dh` = **C**arriage **R**eturn（回車）
- `0Ah` = **L**ine **F**eed（換行）
- 字串通常 **null-terminated**（`,0` 結尾）

## ⭐ DUP 語法
```asm
20 DUP(0)        ; 20 個 0
20 DUP(?)        ; 20 個未初始化
4 DUP("STACK")   ; "STACKSTACKSTACKSTACK"
```

## ⭐ Little Endian vs Big Endian
- **x86 = Little Endian**
- 口訣：**L**SB at **L**owest address
- `12345678h` 在 Little Endian：`78 56 34 12`

## ⭐ `.data` vs `.data?`
| Directive | 用途 | EXE 大小 |
|-----------|------|---------|
| `.data` | 初始化資料 | 包含資料佔位 |
| `.data?` | 未初始化資料 | **只 reserve，檔案小** |

> 載入記憶體後兩者一樣大。

## ⭐ Symbolic Constant 三方法
| Directive | 整數 | 文字 | 可重新定義 |
|-----------|------|------|----------|
| **`=`** | ✅ | ❌ | ✅ |
| **`EQU`** | ✅ | ✅ | ❌ |
| **`TEXTEQU`** | ✅(用 `%`) | ✅ | ✅ |

## ⭐ Current Location Counter `$`（必考）
- `$` 回傳 **當前 statement 的 offset**（byte 為單位）
- 計算陣列元素數：
  - BYTE：`($ - list)`
  - WORD：`($ - list) / 2`
  - DWORD：`($ - list) / 4`

## ⭐ EQU 的 `<>` 差異
- `EQU 10*10` → 計算為 100
- `EQU <10*10>` → 直接複製文字 `10*10`

## ⭐ NOP 用途
- 1 byte，不做事
- 把程式碼對齊到 **even doubleword address**，提升載入效率
- 避免 cache line 邊界造成的 pipeline stall

## ⭐ Operand 四種類型
1. **constant**（立即值）：`mov ax, 96`
2. **constant expression**：`mov ax, 5+4`
3. **register**：`mov ax, bx`
4. **memory**（資料標籤）：`mov count, bx`

## ⭐ Reserved Words 五大類
1. Instruction Mnemonics（ADD、MOV…）
2. Directives（.data、.code…）
3. Type attributes（BYTE、WORD…）
4. Operators（+、-…）
5. Predefined symbols（@data…）
