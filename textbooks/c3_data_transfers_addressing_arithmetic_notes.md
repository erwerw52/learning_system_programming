# Chapter 3：Data Transfers, Addressing, and Arithmetic 完整筆記

> 課程主軸：資料傳送指令、加減運算、EFLAGS 旗標、運算子與 directives、間接定址、JMP/LOOP

---

## 📚 目錄
1. [Data Transfer Instructions](#一data-transfer-instructions)
2. [Addition and Subtraction](#二addition-and-subtraction)
3. [Data-Related Operators and Directives](#三data-related-operators-and-directives)
4. [Indirect Addressing](#四indirect-addressing)
5. [JMP and LOOP Instructions](#五jmp-and-loop-instructions)
6. [速背重點整理](#速背重點整理)

---

## 一、Data Transfer Instructions

### 1. Operand Types（運算元三大類）
| 類型 | 說明 |
|------|------|
| **Immediate** | 常數整數（8/16/32 bits） |
| **Register** | CPU 內具名暫存器 |
| **Memory** | 參照記憶體位置 |

### 2. Instruction Operand Notation（縮寫對照）
| Operand | 描述 |
|---------|------|
| **r8** | 8-bit 通用暫存器：AH/AL/BH/BL/CH/CL/DH/DL |
| **r16** | 16-bit：AX/BX/CX/DX/SI/DI/SP/BP |
| **r32** | 32-bit：EAX/EBX/ECX/EDX/ESI/EDI/ESP/EBP |
| **reg** | 任何通用暫存器 |
| **sreg** | 16-bit 段暫存器：CS/DS/SS/ES/FS/GS |
| **imm** | 8/16/32-bit 立即值 |
| **imm8/16/32** | 對應大小的立即值 |
| **r/m8/16/32** | 暫存器或記憶體 |
| **mem** | 8/16/32-bit memory operand |

### 3. Direct Memory Operands
- 對 memory 的具名參照（label）
- 在 MASM 中 label **自動 dereference**

```asm
.data
var1 BYTE 10h
.code
mov al, var1        ; AL = 10h
mov al, [var1]      ; AL = 10h（替代寫法，用於 expression）
```

### 4. MOV Instruction ⭐
- 語法：`MOV destination, source`
- **三大規則**：
  1. **兩個 operand 大小必須相同**
  2. **最多一個 memory operand**（不可 mem-to-mem）
  3. **EIP（與 IP）不可為 destination**
- MOV **不影響任何 flag**

📝 **錯誤範例分析**：
```asm
.data
bVal  BYTE  100
bVal2 BYTE  ?
wVal  WORD  2
dVal  DWORD 5

.code
mov esi, wVal       ; ❌ size mismatch（ESI 是 32-bit、wVal 是 16-bit）
mov eip, dVal       ; ❌ EIP 不能當 destination
mov 25, bVal        ; ❌ 立即值不能當 destination
mov bVal2, bVal     ; ❌ 不可 memory-to-memory
```

### 5. Overlapping Values（不同大小寫入同一暫存器）
```asm
.data
oneByte  BYTE  78h
oneWord  WORD  1234h
oneDword DWORD 12345678h

.code
mov eax, 0          ; EAX = 00000000h
mov al, oneByte     ; EAX = 00000078h
mov ax, oneWord     ; EAX = 00001234h
mov eax, oneDword   ; EAX = 12345678h
mov ax, 0           ; EAX = 12340000h
```

### 6. MOVZX（Zero Extension）⭐
- 把小的值複製到大的 destination，**高位填 0**
- destination 必須是 **register**
- 用於 **unsigned integer**

```
   Source（8-bit）     →    Destination（16-bit）
   10001111                  00000000 10001111
                            ↑ 高位填 0
```

```asm
mov bl, 10001111b
movzx ax, bl        ; AX = 0000000010001111b
```

⚠️ **錯誤範例**（為何不能用在 signed）：
```asm
.data
count SWORD -16
.code
movzx ecx, count    ; ecx = 65520（錯誤！原值 -16 變正數）
```
- 因為 `-16` 是 `FFF0h`；用 MOVZX 變成 `0000FFF0h = 65520`，不再是 −16
- **解決**：用 MOVSX（sign extension）

### 7. MOVSX（Sign Extension）⭐
- 把小的值複製到大的 destination，**高位填 sign bit**
- destination 必須是 **register**
- 用於 **signed integer**

```
   Source（8-bit）     →    Destination（16-bit）
   10001111                  11111111 10001111
   ↑sign bit=1               ↑ 全填 1
```

```asm
mov bl, 10001111b
movsx ax, bl        ; AX = 1111111110001111b
```

### 8. Extension Summary（記住！）
| 指令 | 用於 |
|------|------|
| **MOVZX** | **unsigned** integers |
| **MOVSX** | **signed** integers |

### 9. LAHF / SAHF（旗標暫存器存取）
- **LAHF**（Load AH from Flags）：把 EFLAGS 低位元組（status flags）載入 AH
- **SAHF**（Store AH to Flags）：把 AH 寫回 EFLAGS 低位元組

```asm
.data
saveflags BYTE ?
.code
lahf                ; load flags into AH
mov saveflags, ah   ; save them in a variable
; ...
mov ah, saveflags   ; load saved flags into AH
sahf                ; copy into Flags register
```

### 10. XCHG Instruction（交換）
- 語法：`XCHG op1, op2`
- 規則：
  - **至少一個 operand 必須是 register**
  - **不允許 immediate operand**
  - **不可 memory-to-memory**

```asm
xchg ax, bx         ; 16-bit 對換
xchg ah, al         ; 8-bit 對換
xchg var1, bx       ; mem ↔ reg
xchg eax, ebx       ; 32-bit 對換

xchg var1, var2     ; ❌ 兩個 memory
```

📝 **mem-to-mem 對換的技巧**：用 register 當橋樑
```asm
mov  ax, val1       ; ax = val1
xchg ax, val2       ; ax = val2, val1 = val2
mov  val1, ax       ; val1 = ax = val2
```

### 11. Direct-Offset Operands（直接偏移運算元）
- **Effective Address (EA)** = data label + constant
- 該位址被 dereference 以取得內容

```asm
.data
arrayB BYTE 10h, 20h, 30h, 40h
.code
mov al, arrayB+1        ; AL = 20h
mov al, [arrayB+1]      ; 替代寫法

.data
arrayW WORD  1000h, 2000h, 3000h
arrayD DWORD 1, 2, 3, 4
.code
mov ax, [arrayW+2]      ; AX = 2000h
mov ax, [arrayW+4]      ; AX = 3000h
mov eax, [arrayD+4]     ; EAX = 00000002h
```

📝 **練習解答**：把 `arrayD DWORD 1,2,3` 重排為 `3, 1, 2`
```asm
mov eax, arrayD         ; eax = 1
xchg eax, [arrayD+4]    ; arrayD: 2,1,3; eax = 2
xchg eax, [arrayD+8]    ; arrayD: 2,1,2; eax = 3
mov arrayD, eax         ; arrayD: 3,1,2
```

---

## 二、Addition and Subtraction

### 1. INC / DEC（自增 / 自減）
- 語法：`INC reg/mem`、`DEC reg/mem`
- 邏輯：`destination ← destination ± 1`

```asm
.data
myWord  WORD  1000h
myDword DWORD 10000000h
.code
inc myWord          ; 1001h
dec myWord          ; 1000h
inc myDword         ; 10000001h

mov ax, 00FFh
inc ax              ; AX = 0100h
mov ax, 00FFh
inc al              ; AX = 0000h（只改 AL）
```

### 2. ADD / SUB
- 語法：`ADD destination, source`、`SUB destination, source`
- 規則同 MOV

```asm
.data
var1 DWORD 10000h
var2 DWORD 20000h
.code                ;       --EAX--
mov eax, var1        ; 00010000h
add eax, var2        ; 00030000h
add ax, 0FFFFh       ; 0003FFFFh
add eax, 1           ; 00040000h
sub ax, 1            ; 0004FFFFh
```

### 3. NEG（Negate 取負）
- 用 **2's complement** 反轉 operand 符號
- operand 可以是 **register 或 memory**

```asm
.data
valB BYTE  -1
valW WORD  +32767
.code
mov al, valB        ; AL = -1
neg al              ; AL = +1
neg valW            ; valW = -32767
```

### 4. 實作算式（Implementing Arithmetic Expressions）
📝 範例：`Rval = -Xval + (Yval - Zval)`
```asm
Rval DWORD ?
Xval DWORD 26
Yval DWORD 30
Zval DWORD 40
.code
mov eax, Xval
neg eax             ; EAX = -26
mov ebx, Yval
sub ebx, Zval       ; EBX = -10
add eax, ebx
mov Rval, eax       ; Rval = -36
```

📝 **練習解答**：`Rval = Xval - (-Yval + Zval)`（不可修改 Xval/Yval/Zval）
```asm
mov ebx, Yval
neg ebx
add ebx, Zval
mov eax, Xval
sub eax, ebx
mov Rval, eax
```

### 5. Flags Affected by Arithmetic ⭐⭐⭐

ALU 的 **status flags** 反映運算結果（基於 **destination operand**），存在 **EFLAGS**。

🔑 **MOV 不影響任何 flag！**

#### Concept Map：
```
arithmetic & bitwise operations
        │ affect
        ▼
    status flags ──used by──→ conditional jumps
        │                          │ provide
        │ attached to             ▼
        ▼                    branching logic
       ALU
        │ part of
        ▼
       CPU
```

#### 重要 flags
| Flag | 觸發時機 |
|------|---------|
| **ZF** Zero | 結果為 0 |
| **SF** Sign | 結果為負（destination 的最高 bit = 1） |
| **CF** Carry | unsigned 超出範圍 |
| **OF** Overflow | signed 超出範圍 |
| **AF** Aux Carry | bit 3 → bit 4 進位 |
| **PF** Parity | 結果低位元組中 1 的個數為偶數 |

### 6. Zero Flag (ZF)
```asm
mov cx, 1
sub cx, 1           ; CX = 0, ZF = 1
mov ax, 0FFFFh
inc ax              ; AX = 0, ZF = 1
inc ax              ; AX = 1, ZF = 0
```

### 7. Sign Flag (SF)
- **規則**：SF = destination 的最高 bit（sign bit）

📝 範例：
```asm
mov bl, 1
sub bl, 2           ; BL = FFh (-1), SF = 1
```
過程：
```
   00000001 (1)
 + 11111110 (-2 的二補數)
 ──────────
   11111111 (FFh)  ← 最高位=1 → SF=1
```

更多例子：
```asm
mov cx, 0
sub cx, 1           ; CX = 1111111111111111b, SF = 1
add cx, 2           ; CX = 0000000000000001b, SF = 0
```

### 8. Carry Flag (CF)
- 當 unsigned 結果超出範圍時 CF=1

```asm
; Example 1：byte 最大 unsigned = 255
mov al, 0FFh
add al, 1           ; CF = 1, AL = 0

; Example 2：byte 最小 unsigned = 0
mov al, 1
sub al, 2           ; CF = 1, AL = 0FFh (=255)
```

#### Addition 與 CF
- **規則**：CF = MSB 進位出來的 bit

📝 範例：
```
   11111111 (FFh = 255)
 + 00000001
 ──────────
 [1]00000000   ← MSB 有進位 → CF=1
```

#### Subtraction 與 CF
- **規則**：CF = **invert**（MSB 進位出來的 bit）
- 因為 SUB 內部是 `dest + (−src 的二補數)`

📝 範例：`1 − 2`
```
   00000001 (1)
 + 11111110 (-2)
 ──────────
   11111111 (FFh)   ← MSB 進位=0 → invert → CF=1
```

### 9. Overflow Flag (OF)
- 當 **signed** 結果超出範圍（overflow / underflow）

```asm
; Example 1：byte 最大 signed = +127
mov al, +127
add al, 1           ; OF = 1

; Example 2：byte 最小 signed = -128
mov al, -128
sub al, 1           ; OF = 1
```

#### Rule of Thumb（加法 OF 判斷）⭐
**OF 只在以下兩種情況設為 1**：
1. **兩個正數相加，結果為負**（sign bit: 0+0 → 1）
2. **兩個負數相加，結果為正**（sign bit: 1+1 → 0）

📝 範例：
```asm
mov al, 80h         ; 80h = -128
add al, 92h         ; 92h = -110；OF = 1（兩負相加變正）

mov al, -2
add al, +127        ; OF = 0（不同號相加）
```

#### Hardware：OF 如何判斷
**OF = (進入 MSB 的 carry) XOR (從 MSB 進位出來的 carry)**

📝 範例：(−128) + (−2) = 10000000 + 11111110
```
進位入 MSB = 1，進位出 MSB = 1
OF = 1 XOR 1 = 0  ❌ 不對
```
等等，slide 上是另一種寫法：
```
   10000000 (-128)
 + 11111110 (-2)
 ──────────
   01111110     ← 兩個負相加變正 → OF=1
```

📝 範例：(−5) − (125) = 11111011 + 10000011（−125 的二補數）
```
   11111011 (-5)
 + 10000011 (-125)
 ──────────
   01111110     ← 兩負變正 → OF=1
```
> `-130` 超出 8-bit signed 範圍 → OF=1

### 10. Auxiliary Carry (AF)
- bit 3 → bit 4 進位（或借位）

📝 範例：`0Fh + 1`
```
   00001111 (0Fh)
 + 00000001
 ──────────
   00010000    ← bit 3 進位到 bit 4 → AC=1
```

### 11. Parity Flag (PF)
- 結果低位元組中 1 的個數為 **偶數** 時 PF=1

```asm
mov al, 10001100b
add al, 00000010b   ; AL = 10001110, PF = 1（四個 1）
sub al, 10000000b   ; AL = 00001110, PF = 0（三個 1）
```

### 12. NEG 與 OF
- 當結果無法正確儲存時，NEG 設 OF=1

```asm
mov al, -128
neg al              ; AL = 10000000b (=-128), OF = 1
                    ; 因為 +128 超出 byte signed 範圍

mov al, +127
neg al              ; AL = 10000001b (=-127), OF = 0
```

---

## 三、Data-Related Operators and Directives

### 1. OFFSET Operator ⭐
- 回傳 label **到 segment 起點的距離**（bytes）
- Protected mode: 32 bits；Real mode: 16 bits

📝 假設 bVal 位於 offset `00404000h`：
```asm
.data
bVal  BYTE  ?       ; offset 00404000h（1 byte）
wVal  WORD  ?       ; offset 00404001h（2 bytes）
dVal  DWORD ?       ; offset 00404003h（4 bytes）
dVal2 DWORD ?       ; offset 00404007h

.code
mov esi, OFFSET bVal    ; ESI = 00404000h
mov esi, OFFSET wVal    ; ESI = 00404001h
mov esi, OFFSET dVal    ; ESI = 00404003h
mov esi, OFFSET dVal2   ; ESI = 00404007h
```

📝 範例：取得陣列第 3 個元素的 offset
```asm
.data
myArray WORD 1,2,3,4,5
.code
mov esi, OFFSET myArray + 4  ; 指向第 3 個 WORD
```

### 2. ALIGN Directive
- 把變數對齊到 byte/word/doubleword 邊界
- 為什麼？**CPU 處理偶位址資料較快**

```asm
bVal  BYTE  ?       ; 00404000
ALIGN 2
wVal  WORD  ?       ; 00404002
bVal2 BYTE  ?       ; 00404004
ALIGN 4
dVal  DWORD ?       ; 00404008
dVal2 DWORD ?       ; 0040400C
```

### 3. PTR Operator ⭐
- **覆寫 label 預設的型別**
- 允許存取 variable 的一部分

```asm
.data
myDouble DWORD 12345678h

.code
mov ax, myDouble                ; ❌ error！size mismatch
mov ax, WORD PTR myDouble       ; AX = 5678h
mov WORD PTR myDouble, 4321h    ; 存入 4321h
```

> ⚠️ `myDouble` 在記憶體中為 **Little Endian**：
> ```
> byte  offset
>  78   0000   ← myDouble
>  56   0001
>  34   0002
>  12   0003
> ```

更多範例：
```asm
mov al, BYTE PTR myDouble       ; AL = 78h
mov al, BYTE PTR [myDouble+1]   ; AL = 56h
mov al, BYTE PTR [myDouble+2]   ; AL = 34h
mov ax, WORD PTR [myDouble]     ; AX = 5678h
mov ax, WORD PTR [myDouble+2]   ; AX = 1234h
```

### 4. PTR：把小型別組合成大型別
```asm
.data
myBytes BYTE 12h,34h,56h,78h    ; 12 34 56 78（low → high）

.code
mov ax, WORD PTR [myBytes]      ; AX = 3412h
mov ax, WORD PTR [myBytes+2]    ; AX = 7856h
mov eax, DWORD PTR myBytes      ; EAX = 78563412h

.data
wordList WORD 5678h, 1234h
.code
mov eax, DWORD PTR wordList     ; EAX = 12345678h
```

### 5. TYPE Operator
- 回傳 **單一元素** 的 **大小（bytes）**

```asm
.data
var1 BYTE  ?        ; TYPE = 1
var2 WORD  ?        ; TYPE = 2
var3 DWORD ?        ; TYPE = 4
var4 QWORD ?        ; TYPE = 8
```

### 6. LENGTHOF Operator
- 回傳資料宣告的 **元素個數**

```asm
.data                                  ; LENGTHOF
byte1   BYTE  10,20,30                  ; 3
array1  WORD  30 DUP(?), 0, 0           ; 32
array2  WORD  5 DUP(3 DUP(?))           ; 15
array3  DWORD 1,2,3,4                   ; 4
digitStr BYTE "12345678",0              ; 9
```

### 7. SIZEOF Operator
- **SIZEOF = LENGTHOF × TYPE**（總 byte 數）

```asm
.data                                  ; SIZEOF
byte1   BYTE  10,20,30                  ; 3
array1  WORD  30 DUP(?), 0, 0           ; 64
array2  WORD  5 DUP(3 DUP(?))           ; 30
array3  DWORD 1,2,3,4                   ; 16
digitStr BYTE "12345678",0              ; 9
```

🔑 **公式記憶**：
| Operator | 回傳 |
|----------|------|
| **TYPE** | 單個元素的 bytes |
| **LENGTHOF** | 元素個數 |
| **SIZEOF** | 總 bytes (= L × T) |

### 8. Spanning Multiple Lines（跨行宣告）
**情況 1**：每行以逗號結尾 → 都屬於同一宣告
```asm
.data
array WORD 10,20,
    30,40,
    50,60
.code
mov eax, LENGTHOF array    ; 6
mov ebx, SIZEOF array      ; 12
```

**情況 2**：分開三行宣告 → `array` 只指第一行
```asm
.data
array WORD 10,20
      WORD 30,40
      WORD 50,60
.code
mov eax, LENGTHOF array    ; 2
mov ebx, SIZEOF array      ; 4
```

### 9. LABEL Directive
- 給 **既有 storage location** 指派 **替代名稱 + 型別**
- **本身不配置儲存空間**
- 取代 PTR 的需求

```asm
.data
val16 LABEL WORD
val32 DWORD 12345678h
.code
mov ax, val16           ; AX = 5678h
mov dx, [val16+2]       ; DX = 1234h
```
> `val16` 是 `val32` 同位置的別名（type = WORD）

📝 範例：用兩個小整數組合成大整數
```asm
.data
LongValue LABEL DWORD
val1      WORD  5678h
val2      WORD  1234h

.code
mov eax, LongValue       ; EAX = 12345678h
```

---

## 四、Indirect Addressing（間接定址）

### 1. Indirect Operands `[reg32]`
- reg32：任何 32-bit 通用暫存器
- register 內存放 **資料的 offset**（當 pointer 用 → 稱為間接定址）

```asm
.data
val1 BYTE 10h, 20h, 30h
.code
mov esi, OFFSET val1
mov al, [esi]           ; AL = 10h（dereference ESI）
inc esi
mov al, [esi]           ; AL = 20h
inc esi
mov al, [esi]           ; AL = 30h
```

### 2. 重要警告
**⚠️ 使用前一定要初始化 register**！
```asm
.data
val1 BYTE 10h,20h,30h
.code
mov ax, [esi]       ; ❌ General protection fault（ESI 未初始化）
```

### 3. PTR 配合間接定址
**ambiguous size 時必須用 PTR**：
```asm
.data
myCount WORD 0
.code
mov esi, OFFSET myCount
inc [esi]               ; ❌ error: ambiguous
inc WORD PTR [esi]      ; ✅ OK
```
> assembler 不知道 ESI 指向 byte / word / dword

### 4. Array Sum 範例（WORD）
```asm
.data
arrayW WORD 1000h, 2000h, 3000h
.code
mov esi, OFFSET arrayW
mov ax, [esi]
add esi, 2              ; or: add esi, TYPE arrayW
add ax, [esi]
add esi, 2
add ax, [esi]           ; AX = sum
```

### 5. Array Sum（DWORD：要 +4）
```asm
.data
arrayD DWORD 10000h, 20000h, 30000h
.code
mov esi, OFFSET arrayD
mov ax, [esi]
add esi, 4              ; or: add esi, TYPE arrayD
add ax, [esi]
add esi, 4
add ax, [esi]
```

### 6. Indexed Operands（索引定址）
- 計算 EA = index register + base address
- 兩種寫法：**`[label + reg]`** 或 **`label[reg]`**

```asm
.data
arrayW WORD 1000h, 2000h, 3000h
.code
mov esi, 0
mov ax, [arrayW + esi]  ; AX = 1000h
mov ax, arrayW[esi]     ; 替代寫法
add esi, 2              ; or: add esi, TYPE arrayW
mov ax, [arrayW + esi]  ; AX = 2000h
```

### 7. Indexed Operand + scale factor
**乘以元素大小** 來索引：
```asm
.data
arrayD DWORD 1h, 2h, 3h, 4h
.code
mov esi, 3
mov eax, arrayD[esi * 4]            ; EAX = 4h（第 4 個元素）

; 或用 TYPE
mov eax, arrayD[esi * TYPE arrayD]  ; 更彈性
```

### 8. Pointers（指標變數）
- 包含另一變數 offset 的變數

```asm
.data
arrayW WORD 1000h, 2000h, 3000h
ptrW   DWORD arrayW
.code
mov esi, ptrW
mov ax, [esi]           ; AX = 1000h
```

更清楚的寫法（用 OFFSET）：
```asm
.data
arrayW WORD  1000h, 2000h, 3000h
ptrW   DWORD OFFSET arrayW
```

---

## 五、JMP and LOOP Instructions

### 1. 控制流分類
| 類型 | 說明 | 範例 |
|------|------|------|
| **Unconditional transfer** | 必定分支 | JMP |
| **Conditional transfer** | 條件成立才分支 | LOOP（看 ECX）、conditional jump（看 flags） |

> 預設 CPU 依序執行指令（EIP 自動遞增）

### 2. JMP Instruction
- 無條件 jump
- 語法：`JMP target`
- 邏輯：`EIP ← target's offset`

```asm
top:
    .
    .
    jmp top         ; 無條件跳回 top
```

### 3. LOOP Instruction ⭐
- 建立計數迴圈
- 語法：`LOOP target`
- 邏輯：
  1. `ECX ← ECX − 1`
  2. **如果 ECX > 0，jump to target**

📝 範例：累加 5 次
```asm
mov ax, 0
mov ecx, 5
L1:
    inc ax
    loop L1
; 最終 AX = 5, ECX = 0
```

### 4. LOOP 限制
- 跳轉距離必須在 **−128 ~ +127 bytes** 內
- 一條 machine instruction 約 3 bytes → 最多 **約 42 條指令**
- 超過會出現：`Error A2075: jump destination too far: by 14 (bytes)`
- 💡 現代開發環境多半已解決此限制

### 5. 修改 ECX 的注意事項
**必須儲存與還原 ECX**：
```asm
.data
count DWORD ?
.code
    mov ecx, 100    ; set loop count
Top:
    mov count, ecx  ; save the count
    .
    mov ecx, 20     ; modify ecx
    .
    mov ecx, count  ; restore loop count
    loop Top
```

### 6. Your Turn 練習解答

**Q1**：AX 最終值？
```asm
mov ax, 6
mov ecx, 4
L1:
    inc ax
    loop L1
```
→ AX 從 6 開始，迴圈跑 4 次 → **AX = 10**

**Q2**：迴圈跑幾次？
```asm
mov ecx, 0
X2:
    inc ax
    loop X2
```
→ ECX=0 → 第一次 LOOP 時 ECX-1 變成 −1（unsigned 環繞）= FFFFFFFFh
→ 然後跑到 0 才停 → **4,294,967,296 次**（**注意陷阱**）

🔑 **背重點**：**ECX = 0 時做 LOOP 會跑 2³² 次**！

### 7. Nested Loop（巢狀迴圈）
**外層 ECX 必須保存**：
```asm
.data
count DWORD ?
.code
    mov ecx, 100        ; 外層 100 次
L1:
    mov count, ecx      ; 保存外層 ECX
    mov ecx, 20         ; 內層 20 次
L2: .
    .
    loop L2             ; 內層迴圈
    mov ecx, count      ; 還原外層 ECX
    loop L1             ; 外層迴圈
```

### 8. 綜合範例：陣列加總
```asm
.data
intarray WORD 100h, 200h, 300h, 400h
.code
    mov edi, OFFSET intarray   ; intarray 位址
    mov ecx, LENGTHOF intarray ; 迴圈計數（ECX = 4）
    mov ax, 0                  ; 累加器歸零
L1:
    add ax, [edi]              ; 加入一個元素
    add edi, TYPE intarray     ; 指向下一個元素
    loop L1                    ; 重複到 ECX=0
```

---

## 六、Summary：四種定址方式

| 定址方式 | 範例 |
|---------|------|
| **Direct operand** | `mov ecx, var1` |
| **Direct-offset operand** | `mov al, [array + 1]` |
| **Indirect operand**（用 register 當指標）| `mov esi, OFFSET array`<br>`mov ax, [esi]` |
| **Indexed operand**（reg + 常數） | `mov esi, 0`<br>`mov al, [array + esi]` |

---

# 🔥 速背重點整理（考前衝刺）

## ⭐ MOV 三大規則
1. **兩個 operand 大小必須相同**
2. **不可 memory-to-memory**
3. **EIP 不可當 destination**
4. **MOV 不影響 flags**

## ⭐ MOVZX vs MOVSX
| 指令 | 上半填 | 用於 |
|------|--------|------|
| **MOVZX** | 0 | unsigned |
| **MOVSX** | sign bit | signed |

> destination **必須是 register**

## ⭐ XCHG 規則
- 至少一個 operand 是 register
- 不可 immediate
- mem-to-mem 要用 register 當橋樑

## ⭐ EFLAGS Status Flags（必背）
| Flag | 觸發 |
|------|------|
| **ZF** Zero | 結果 = 0 |
| **SF** Sign | 結果負（=destination 最高 bit） |
| **CF** Carry | **U**nsigned 溢位 |
| **OF** Overflow | **S**igned 溢位 |
| **AF** Aux Carry | bit3 → bit4 進位 |
| **PF** Parity | 低位元組 1 的個數為偶數 |

> ⭐ **MOV 不影響任何 flag**

## ⭐ OF 判斷（加法 Rule of Thumb）
**只在以下兩種情況 OF=1**：
- **兩個正數相加 → 結果為負**
- **兩個負數相加 → 結果為正**

> 不同號相加 OF 必為 0

## ⭐ Hardware：OF = (進入 MSB 的 carry) XOR (從 MSB 出來的 carry)

## ⭐ CF 加減差異
- **加法**：CF = MSB 進位出去的 bit
- **減法**：CF = invert（MSB 進位出去的 bit）

## ⭐ 五大運算子比較
| Operator | 回傳 |
|----------|------|
| **OFFSET** | label 到 segment 起點的距離 |
| **TYPE** | 單一元素大小（bytes） |
| **LENGTHOF** | 元素個數 |
| **SIZEOF** | LENGTHOF × TYPE（總 bytes） |
| **PTR** | 覆寫 label 預設型別 |

## ⭐ LABEL Directive
- 給同位置取別名 + 改型別
- **不配置儲存空間**
- 取代 PTR

## ⭐ Little Endian（x86）
`myDouble DWORD 12345678h` 在記憶體：
```
0000: 78
0001: 56
0002: 34
0003: 12
```

## ⭐ 四大定址方式
1. **Direct**：`mov ecx, var1`
2. **Direct-offset**：`mov al, [array+1]`
3. **Indirect**：`[esi]`（register 當指標）
4. **Indexed**：`[array+esi]` 或 `array[esi]`

## ⭐ Indirect Operand 規則
- 用之前**必須初始化** register
- 大小模糊時用 **PTR**：`inc WORD PTR [esi]`

## ⭐ Indexed Operand 帶 scale factor
`arrayD[esi * 4]` 或 `arrayD[esi * TYPE arrayD]`

## ⭐ JMP vs LOOP
| 指令 | 條件 | 邏輯 |
|------|------|------|
| **JMP target** | 無條件 | `EIP ← target` |
| **LOOP target** | 條件 | `ECX--`，若 ECX>0 則跳 |

## ⭐ LOOP 陷阱
- 範圍 **−128 ~ +127 bytes**
- **ECX = 0 時 LOOP 會跑 2³² 次**（環繞）
- 修改 ECX 要保存 / 還原
- Nested loop 外層 ECX 一定要存

## ⭐ INC 與 DEC
- 邏輯：`destination ± 1`
- 影響 flags（**但不影響 CF**！這是冷知識）

## ⭐ NEG
- 把 operand 轉成 2's complement（取負）
- operand 可為 register / memory
- 若結果無法儲存（如 NEG -128 in byte）→ OF=1

## ⭐ Direct-Offset 注意
- `[label + 常數]`：合法（label 是常數）
- `[reg + 常數]`：合法
- `mov eax, ebx+1`：❌ 無效（不是 [...] 形式）

## ⭐ Spanning multiple lines
- **以逗號結尾** → 同一宣告（LENGTHOF / SIZEOF 涵蓋全部）
- **分開 WORD** → label 只指第一行

## ⭐ LAHF / SAHF
- **LAHF**：EFLAGS 低位元組 → AH
- **SAHF**：AH → EFLAGS 低位元組

## ⭐ Specialized Registers 複習
| Register | 用途 |
|----------|------|
| EAX | **A**ccumulator（乘除法） |
| **ECX** | **C**ounter（**LOOP 用**） |
| ESP | **S**tack **P**ointer |
| ESI, EDI | **I**ndex（source/dest） |
| EBP | **B**ase Pointer |
