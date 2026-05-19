# Chapter 5：Conditional Processing 完整筆記

> 課程主軸：Boolean / 比較指令、條件式跳躍（Jcond）、條件式 Loop、條件式控制結構（IF / WHILE / Switch）

---

## 📚 目錄
1. [Boolean and Comparison Instructions](#一boolean-and-comparison-instructions)
2. [Conditional Jumps](#二conditional-jumps)
3. [Conditional Loop Instructions](#三conditional-loop-instructions)
4. [Conditional Structures](#四conditional-structures)
5. [速背重點整理](#速背重點整理)

---

## 一、Boolean and Comparison Instructions

### 1. Status Flags 複習
| Flag | 觸發時機 |
|------|---------|
| **Zero (ZF)** | 運算結果 = 0 |
| **Carry (CF)** | **Unsigned** 超出範圍 |
| **Sign (SF)** | destination 為負數 |
| **Overflow (OF)** | **Signed** 超出範圍 |

### 2. AND 指令 ⭐
- 對兩個 operand 的每個 bit 做 Boolean AND
- 語法：`AND destination, source`（operand 規則同 MOV）

```
   00111011
AND 00001111
   ────────
   0000 1011   ← 高 4 bits 被 cleared、低 4 bits 不變
```

| X | Y | X∧Y |
|---|---|----|
| 0 | 0 | 0 |
| 0 | 1 | 0 |
| 1 | 0 | 0 |
| 1 | 1 | 1 |

🔑 **AND 用途**：**清除選定 bits、保留其他**（與 0 → 清零；與 1 → 不變）

### 3. OR 指令 ⭐
- 語法：`OR destination, source`

```
   00111011
OR  00001111
   ────────
   0011 1111   ← 低 4 bits 被 set、高 4 bits 不變
```

| X | Y | X∨Y |
|---|---|----|
| 0 | 0 | 0 |
| 0 | 1 | 1 |
| 1 | 0 | 1 |
| 1 | 1 | 1 |

🔑 **OR 用途**：**設定選定 bits、保留其他**（與 1 → 設為 1；與 0 → 不變）

### 4. XOR 指令 ⭐
- 語法：`XOR destination, source`

```
   00111011
XOR 00001111
   ────────
   0011 0100   ← 低 4 bits 被反轉、高 4 bits 不變
```

| X | Y | X⊕Y |
|---|---|----|
| 0 | 0 | 0 |
| 0 | 1 | 1 |
| 1 | 0 | 1 |
| 1 | 1 | 0 |

🔑 **XOR 用途**：**反轉選定 bits、保留其他**（與 1 → 反轉；與 0 → 不變）

> 重要特性：**(X ⊕ Y) ⊕ Y = X**（可用於加密／解密）

### 5. NOT 指令
- 對單一 destination 做 Boolean NOT
- 語法：`NOT destination`

```
NOT 00111011
   ────────
   11000100   ← 全部反轉
```

### 6. Bit-Mapped Sets（位元映射集合）
- 用 binary bits 表示「集合成員」
- 又稱 **bit vector / bit map**

**應用**：
- 作業系統：表示記憶體區塊是否被使用
- 檔案系統：哪些 disk block 被佔用
- 資料庫：bit vector index（快速查詢欄位值是否出現過）

📝 **範例**：要儲存 {1, 3, 4}
- 方法一：陣列 `[1, 3, 4]` → 每個 4 bytes，共 12 bytes，查詢要遍歷
- 方法二：bit-mapped → `00011010`（位元 1、3、4 設為 1）→ **只 1 byte**，查詢用 bitwise 操作

### 7. Bit-Mapped Set Operations

#### Set Intersection（交集）→ 用 AND
```asm
mov eax, SetX         ; 00000111 (SetX)
and eax, SetY         ; 01100011 (SetY)
                      ; 00000011 (Intersection)
```

#### Set Union（聯集）→ 用 OR
```asm
mov eax, SetX         ; 00000111 (SetX)
or  eax, SetY         ; 01100011 (SetY)
                      ; 01100111 (Union)
```

#### Set Complement（補集）→ 用 NOT
```asm
mov eax, SetX         ; 00000111 (SetX)
not eax               ; 11111000 (Complement)
```

### 8. AND / OR 經典應用

#### 應用一：小寫轉大寫（用 AND）
ASCII：`'a'=61h` vs `'A'=41h`，**只差 bit 5**
```
'a' = 0 1 1 0 0 0 0 1 = 61h
'A' = 0 1 0 0 0 0 0 1 = 41h
              ↑
            bit 5
```
```asm
mov al, 'a'              ; AL = 01100001b
and al, 11011111b        ; AL = 01000001b ('A')
```

#### 應用二：binary 數字轉 ASCII 數字（用 OR）
```asm
mov al, 6                ; AL = 00000110b
or  al, 00110000b        ; AL = 00110110b ('6')
```
> ASCII '6' = 36h = 00110110b，差 bits 4 和 5

### 9. TEST 指令 ⭐
- 執行 **nondestructive AND**（不破壞 destination）
- 與 AND 行為相同，但 **不修改 operand**
- **僅影響 flags**（特別是 Zero flag）

📝 範例：檢查 AL 的 bit 0 或 bit 1 是否被 set
```asm
test al, 00000011b
jnz  ValueFound        ; 結果不為 0 表示至少一個 bit 是 1
```

### 10. CMP 指令 ⭐⭐⭐
- **Nondestructive subtraction**：destination − source（**但不寫回**）
- 只修改 flags
- 語法：`CMP destination, source`

#### Case 1：`destination == source` → ZF=1
```asm
mov al, 5
cmp al, 5            ; Zero flag set (ZF=1)
```

#### Case 2：`destination < source`（unsigned）→ CF=1
```asm
mov al, 4
cmp al, 5            ; Carry flag set (CF=1)
```
過程：4 − 5 = 4 + (−5)
```
   00000100 (=4)
 + 11111011 (=-5)
 ──────────
   11111111         ← MSB 進位=0 → invert → CF=1
```

#### Case 3：`destination > source`（unsigned）→ CF=0, ZF=0
```asm
mov al, 6
cmp al, 5            ; CF=0, ZF=0
```

#### Signed 比較
- 用 **SF 與 OF 的關係** 判斷

📝 範例：`5 > -2`（dest > src）
```asm
mov al, 5
cmp al, -2           ; SF == OF（兩者都=0）
```
```
   00000101 (=5)
 + 00000010 (=+2, -(-2)的二補數)
 ──────────
   00000111
   SF=0, OF=0 → SF=OF → dest > src
```

📝 範例：`-1 < 5`
```asm
mov al, -1
cmp al, 5            ; SF != OF
```
```
   11111111 (=-1)
 + 11111011 (=-5)
 ──────────
   11111010
   SF=1, OF=0 → SF!=OF → dest < src
```

### 11. CMP 結果與 Flags 對照表 ⭐⭐⭐

#### Unsigned 比較
| CMP 結果 | ZF | CF |
|----------|----|----|
| dest < source | 0 | **1** |
| dest > source | 0 | 0 |
| dest = source | **1** | 0 |

#### Signed 比較
| CMP 結果 | Flags |
|----------|-------|
| dest < source | **SF ≠ OF** |
| dest > source | **SF = OF** |
| dest = source | **ZF = 1** |

### 12. 經典練習題 ⭐
**CMP a, b**：`a = 00001111`, `b = 11110110`，填出 flags 與解讀

過程：a − b = a + (−b 的二補數 = 00001010)
```
   00001111 (=a)
 + 00001010 (=-b)
 ──────────
   00011001
   ZF=0, SF=0, CF=1, OF=0
```

**解讀**：
- **若視為 unsigned**：a=15, b=246 → **a < b**（CF=1）
- **若視為 signed**：a=15, b=-10 → **a > b**（SF=OF=0）

🔑 **CPU 沒有 unsigned/signed 概念**！CPU 只做 binary 計算並設定 flags，**由軟體決定** 怎麼解讀。

### 13. 個別設定/清除 Flags

| 目標 | 方法 |
|------|------|
| **Set ZF** | `AND al, 0` |
| **Clear ZF** | `OR al, 1` |
| **Set SF** | `OR al, 80h` |
| **Clear SF** | `AND al, 7Fh` |
| **Set CF** | `STC`（Set Carry） |
| **Clear CF** | `CLC`（Clear Carry） |
| **Set OF** | 兩正數相加得負（造成 overflow） |
| **Clear OF** | `OR eax, 0` |

---

## 二、Conditional Jumps

### 1. Jcond 概念
- 當特定 register 或 flag 條件成立時跳到 label

### 2. Jumps Based on Specific Flags（看 flag）
| 助記符 | 描述 | 條件 |
|--------|------|------|
| **JZ** | Jump if Zero | ZF=1 |
| **JNZ** | Jump if Not Zero | ZF=0 |
| **JC** | Jump if Carry | CF=1 |
| **JNC** | Jump if Not Carry | CF=0 |
| **JO** | Jump if Overflow | OF=1 |
| **JNO** | Jump if Not Overflow | OF=0 |
| **JS** | Jump if Signed | SF=1 |
| **JNS** | Jump if Not Signed | SF=0 |
| **JP** | Jump if Parity (even) | PF=1 |
| **JNP** | Jump if Not Parity (odd) | PF=0 |

### 3. Jumps Based on Equality（看相等）
| 助記符 | 描述 |
|--------|------|
| **JE** | Jump if equal（leftOp = rightOp）|
| **JNE** | Jump if not equal |
| **JCXZ** | Jump if CX = 0 |
| **JECXZ** | Jump if ECX = 0 |

### 4. Jumps Based on Unsigned Comparisons ⭐
| 助記符 | 描述 |
|--------|------|
| **JA** | Jump if above（leftOp > rightOp）|
| **JNBE** | Jump if not below or equal（同 JA）|
| **JAE** | Jump if above or equal（leftOp ≥ rightOp）|
| **JNB** | Jump if not below（同 JAE）|
| **JB** | Jump if below（leftOp < rightOp）|
| **JNAE** | Jump if not above or equal（同 JB）|
| **JBE** | Jump if below or equal（leftOp ≤ rightOp）|
| **JNA** | Jump if not above（同 JBE）|

🔑 **背口訣**：**A**bove / **B**elow 是 **unsigned**

### 5. Jumps Based on Signed Comparisons ⭐
| 助記符 | 描述 |
|--------|------|
| **JG** | Jump if greater（leftOp > rightOp）|
| **JNLE** | Jump if not less than or equal（同 JG）|
| **JGE** | Jump if greater than or equal |
| **JNL** | Jump if not less（同 JGE）|
| **JL** | Jump if less |
| **JNGE** | Jump if not greater than or equal（同 JL）|
| **JLE** | Jump if less than or equal |
| **JNG** | Jump if not greater（同 JLE）|

🔑 **背口訣**：**G**reater / **L**ess 是 **signed**

### 6. 經典應用

#### 應用 1：判斷大小
```asm
; unsigned EAX > EBX → 跳到 Larger
cmp eax, ebx
ja  Larger

; signed EAX > EBX → 跳到 Greater
cmp eax, ebx
jg  Greater
```

#### 應用 2：判斷小於等於
```asm
; unsigned EAX <= Val1
cmp eax, Val1
jbe L1                ; below or equal

; signed EAX <= Val1
cmp eax, Val1
jle L1
```

#### 應用 3：找較大／較小值
```asm
; unsigned：AX 與 BX 較大者存入 Large
mov Large, bx
cmp ax, bx
jna Next              ; AX 沒有大於 BX
mov Large, ax
Next:

; signed：AX 與 BX 較小者存入 Small
mov Small, ax
cmp bx, ax
jnl Next              ; BX 沒有小於 AX
mov Small, bx
Next:
```

#### 應用 4：檢查記憶體
```asm
; ESI 指向的 word 是否為 0
cmp WORD PTR [esi], 0
je  L1

; EDI 指向的 doubleword 是否為偶數
test DWORD PTR [edi], 1   ; 檢查 bit 0
jz   L2                   ; bit 0=0 → 偶數
```

📐 **奇偶判斷原理**：
- 奇數 = 二進位以 **1** 結尾
- 用 `test ..., 1` 萃取 bit 0
- 若 bit 0 = 0 → 偶數 → ZF=1 → JZ 跳

#### 應用 5：檢查多個 bit
```asm
; 若 AL 的 bits 0, 1, 3 全部 set 則跳 L1
and al, 00001011b        ; clear unwanted bits
cmp al, 00001011b        ; check remaining bits
je  L1                   ; all set? jump
```

### 7. 字串加密程式（XOR）⭐
```asm
KEY = 239
.data
buffer  BYTE 100 DUP(0)
bufSize DWORD ?

.code
    mov edx, OFFSET buffer
    mov ecx, SIZEOF buffer
    call ReadString
    mov bufSize, eax
    mov ecx, bufSize
    mov esi, 0
L1:
    xor buffer[esi], KEY      ; 每個 byte XOR KEY
    inc esi
    loop L1
```

🔑 **加密 = 解密**：因為 `(X ⊕ Y) ⊕ Y = X`，所以同樣的程式可以加密與解密

範例輸出：
```
Enter the plain text: Attack at dawn.
Cipher text: «¢¢Äîä-Ä¢-ïÄÿü-Gs
Decrypted: Attack at dawn.
```

---

## 三、Conditional Loop Instructions

### 1. LOOPZ / LOOPE（loop if zero / equal）
- 語法：`LOOPZ destination` 或 `LOOPE destination`
- 邏輯：
  1. `ECX ← ECX − 1`
  2. **如果 ECX > 0 且 ZF=1，跳到 destination**

### 2. LOOPNZ / LOOPNE（loop if not zero / not equal）
- 語法：`LOOPNZ destination` 或 `LOOPNE destination`
- 邏輯：
  1. `ECX ← ECX − 1`
  2. **如果 ECX > 0 且 ZF=0，跳到 destination**

### 3. LOOPNZ 範例：找陣列中第一個正數
```asm
.data
array    SBYTE -3,-6,-1,-10,10,30,40,4
sentinel SBYTE 0
.code
    mov esi, OFFSET array
    mov ecx, LENGTHOF array
next:
    test BYTE PTR [esi], 80h   ; 測試 sign bit
    pushfd                     ; 保存 flags
    add esi, TYPE array
    popfd                      ; 還原 flags
    loopnz next                ; ZF=0 且 ECX>0 繼續
    jnz quit                   ; 沒找到
    sub esi, TYPE array        ; ESI 指向找到的正數
quit:
```

📐 **為什麼用 PUSHFD/POPFD？**
- `add esi, ...` 會修改 flags
- 必須在 LOOPNZ 之前 **保存 test 的結果**
- 所以 push → add → pop

📐 **原理**：正數 → sign bit = 0
```
-3 = 11111101    test  10000000 (80h)
                ─────────────────
                  10000000    ← 非零 → ZF=0 → LOOPNZ 繼續

10 = 00001010    test  10000000
                ─────────────────
                  00000000    ← 為零 → ZF=1 → LOOPNZ 跳出
```

### 4. 練習：找第一個非零值
```asm
.data
array    SWORD 50 DUP(?)
sentinel SWORD 0FFFFh
.code
    mov esi, OFFSET array
    mov ecx, LENGTHOF array
L1: cmp WORD PTR [esi], 0       ; 檢查是否為 0
    pushfd                       ; 保存 flags
    add esi, TYPE array
    popfd                        ; 還原 flags
    loope L1                     ; ZF=1 且 ECX>0 繼續（都是 0 就繼續）
    jz   quit                    ; 全是 0
    sub  esi, TYPE array         ; ESI 指向找到的非零值
quit:
```

### 5. LOOP 系列比較表
| 指令 | 跳轉條件 |
|------|---------|
| **LOOP** | ECX > 0 |
| **LOOPZ / LOOPE** | ECX > 0 **且** ZF=1 |
| **LOOPNZ / LOOPNE** | ECX > 0 **且** ZF=0 |

---

## 四、Conditional Structures

### 1. Block-Structured IF
**IF-ELSE 翻譯為組合語言** → 用 `CMP` + 條件跳躍

📝 範例：`if (op1 == op2) X=1; else X=2;`
```asm
    mov eax, op1
    cmp eax, op2
    jne L1
    mov X, 1
    jmp L2
L1: mov X, 2
L2:
```

### 2. 練習：unsigned 的 IF
**Pseudocode**：
```c
if (ebx <= ecx) {
    eax = 5;
    edx = 6;
}
```

**Assembly**（短路寫法）：
```asm
    cmp ebx, ecx
    ja  next        ; unsigned 不小於等於 → 跳出
    mov eax, 5
    mov edx, 6
next:
```

🔑 **反向條件**：若條件不成立就跳走，較簡潔。

### 3. 練習：signed 的 IF-ELSE
**Pseudocode**：
```c
if (var1 <= var2)
    var3 = 10;
else {
    var3 = 6;
    var4 = 7;
}
```

**Assembly**：
```asm
    mov eax, var1
    cmp eax, var2
    jle L1                  ; signed <=
    mov var3, 6
    mov var4, 7
    jmp L2
L1: mov var3, 10
L2:
```

### 4. Compound Expression with AND（短路求值）⭐

**Pseudocode**：`if (al > bl) AND (bl > cl) X = 1;`

**版本一**（直觀）：
```asm
    cmp al, bl              ; first expression
    ja  L1
    jmp next
L1: cmp bl, cl              ; second expression
    ja  L2
    jmp next
L2: mov X, 1
next:
```

**版本二**（**反向條件、節省 29% 程式碼**）⭐：
```asm
    cmp al, bl              ; first expression
    jbe next                ; quit if false
    cmp bl, cl              ; second expression
    jbe next                ; quit if false
    mov X, 1                ; both are true
next:
```

🔑 **AND 短路**：任一條件 false 就跳出 → 用 **反向條件 + 跳出**

### 5. 練習：unsigned 複合 AND
**Pseudocode**：
```c
if (ebx <= ecx && ecx > edx) {
    eax = 5;
    edx = 6;
}
```

**Assembly**：
```asm
    cmp ebx, ecx
    ja  next             ; 第一個不成立 → 跳出
    cmp ecx, edx
    jbe next             ; 第二個不成立 → 跳出
    mov eax, 5
    mov edx, 6
next:
```

### 6. Compound Expression with OR

**Pseudocode**：`if (al > bl) OR (bl > cl) X = 1;`

**Assembly**：
```asm
    cmp al, bl              ; al > bl?
    ja  L1                  ; yes → 直接執行
    cmp bl, cl              ; no → 試第二個
    jbe next                ; no → 跳過
L1: mov X, 1
next:
```

🔑 **OR 短路**：任一條件 true 就跳到 body → 用 **正向條件 + 跳到 body**

### 7. WHILE 迴圈
**Pseudocode**：
```c
while (eax < ebx)
    eax = eax + 1;
```

**Assembly**：
```asm
top:
    cmp eax, ebx           ; 檢查條件
    jae next               ; false → 離開
    inc eax                ; loop body
    jmp top                ; 重複
next:
```

🔑 **WHILE 模板**：
- 開頭 cmp + 反向條件跳出
- 結尾 jmp 回頭

### 8. 練習：WHILE 範例
**Pseudocode**：
```c
while (ebx <= val1) {
    ebx = ebx + 5;
    val1 = val1 - 1;
}
```

**Assembly**：
```asm
top:
    cmp ebx, val1
    ja  next               ; unsigned >
    add ebx, 5
    dec val1
    jmp top
next:
```

### 9. Table-Driven Selection（表驅動選擇）⭐
- 用 **table lookup** 取代多重 IF 或 switch
- 表中放入 **lookup values** 與 **labels/procedures 的 offsets**
- 適合 **大量比較** 的情境

#### Step 1：建立表
```asm
.data
CaseTable BYTE 'A'                   ; lookup value
   DWORD Process_A                   ; procedure 位址
   EntrySize = ($ - CaseTable)
   BYTE 'B'
   DWORD Process_B
   BYTE 'C'
   DWORD Process_C
   BYTE 'D'
   DWORD Process_D
NumberOfEntries = ($ - CaseTable) / EntrySize
```

📐 **表的結構**：
```
'A' [00000120] 'B' [00000130] 'C' [00000140] 'D' [00000150]
 ↑      ↑
lookup  Process_B 位址
value
```

#### Step 2：搜尋表
```asm
    mov ebx, OFFSET CaseTable        ; EBX 指向表
    mov ecx, NumberOfEntries         ; 迴圈計數

L1: cmp al, [ebx]                    ; 找到？
    jne L2
    call NEAR PTR [ebx + 1]          ; 是 → 呼叫 procedure
    jmp L3
L2: add ebx, EntrySize               ; 下一個 entry
    loop L1
L3:
```

🔑 **重點**：`call NEAR PTR [ebx + 1]` 中 NEAR PTR 是 procedure pointer 必須的。

---

# 🔥 速背重點整理（考前衝刺）

## ⭐ 四大 Bitwise 指令用途
| 指令 | 用途 | 口訣 |
|------|------|------|
| **AND** | **clear** bits | 與 0 清零 |
| **OR** | **set** bits | 與 1 設一 |
| **XOR** | **invert** bits | 與 1 反轉 |
| **NOT** | 全部反轉 | – |

## ⭐ AND/OR/XOR 與 0/1 對應表
| 操作 | 與 0 | 與 1 |
|------|------|------|
| AND | → 0 | 不變 |
| OR | 不變 | → 1 |
| XOR | 不變 | 反轉 |

## ⭐ TEST vs AND
| 指令 | 修改 destination？ | 影響 flags？ |
|------|-------------------|------------|
| **AND** | ✅ | ✅ |
| **TEST** | ❌ | ✅ |

## ⭐ CMP vs SUB
| 指令 | 修改 destination？ | 影響 flags？ |
|------|-------------------|------------|
| **SUB** | ✅ | ✅ |
| **CMP** | ❌ | ✅ |

## ⭐ CMP 結果對照表（必背！）
### Unsigned
| 結果 | ZF | CF |
|------|----|----|
| dest < src | 0 | **1** |
| dest > src | 0 | 0 |
| dest = src | **1** | 0 |

### Signed
| 結果 | 條件 |
|------|------|
| dest < src | **SF ≠ OF** |
| dest > src | **SF = OF** |
| dest = src | **ZF = 1** |

## ⭐ Jcond 助記符記憶法
| 字母 | 意義 | 用於 |
|------|------|------|
| **A** | Above | **U**nsigned > |
| **B** | Below | **U**nsigned < |
| **G** | Greater | **S**igned > |
| **L** | Less | **S**igned < |
| **E** | Equal | 相等 |
| **N** | Not | 否定 |
| **Z** | Zero | ZF |
| **C** | Carry | CF |
| **S** | Sign | SF |
| **O** | Overflow | OF |

🔑 **常見組合**：JA、JAE、JB、JBE（unsigned），JG、JGE、JL、JLE（signed），JE、JNE、JZ、JNZ

## ⭐ Set/Clear Flags 速查
| 目標 | 指令 |
|------|------|
| Set ZF | `AND op, 0` |
| Clear ZF | `OR op, 1` |
| Set SF | `OR op, 80h` |
| Clear SF | `AND op, 7Fh` |
| Set CF | **STC** |
| Clear CF | **CLC** |
| Clear OF | `OR eax, 0` |

## ⭐ LOOP 系列（必背）
| 指令 | 條件 |
|------|------|
| **LOOP** | ECX>0 |
| **LOOPZ / LOOPE** | ECX>0 **且** ZF=1 |
| **LOOPNZ / LOOPNE** | ECX>0 **且** ZF=0 |

> 都會 `ECX -= 1` 再判斷

## ⭐ PUSHFD/POPFD 在 Loop 中的用途
- 保護 **CMP/TEST 設定的 flags** 不被中間運算（如 `add esi`）破壞
- 模式：`test → pushfd → 中間運算 → popfd → loopXX`

## ⭐ IF/ELSE 翻譯模式
```
if (cond) BODY1; else BODY2;
↓
    cmp ...
    j(反向 cond) L1
    BODY1
    jmp L2
L1: BODY2
L2:
```

## ⭐ Short-Circuit 翻譯模式
### AND（任一 false 就跳出）
```
if (A && B) BODY;
↓
    cmp A...
    j(!A) next       ; A false → 跳出
    cmp B...
    j(!B) next       ; B false → 跳出
    BODY
next:
```

### OR（任一 true 就執行）
```
if (A || B) BODY;
↓
    cmp A...
    j(A) L1          ; A true → 跑 BODY
    cmp B...
    j(!B) next       ; B false → 跳出
L1: BODY
next:
```

## ⭐ WHILE 翻譯模式
```
while (cond) BODY;
↓
top:
    cmp ...
    j(反向 cond) next
    BODY
    jmp top
next:
```

## ⭐ XOR 加解密原理
`(X ⊕ KEY) ⊕ KEY = X` → **加密與解密用同一個 procedure**

## ⭐ Bit-Mapped Set 三大運算
| 集合運算 | bitwise 指令 |
|---------|------------|
| **交集**（∩）| AND |
| **聯集**（∪）| OR |
| **補集**（complement）| NOT |

## ⭐ 奇偶判斷
**奇數** ↔ 二進位最低位 = **1**
```asm
test value, 1
jz  IsEven           ; bit 0 = 0 → 偶數
```

## ⭐ Table-Driven 流程
1. **建表**：lookup value + procedure offset
2. **走訪**：比對 lookup value
3. **匹配**：`call NEAR PTR [entry + offset]`

## ⭐ CPU 不認 signed/unsigned
- CPU 只做二進位計算與設 flags
- **由程式決定** 把結果解讀為 signed 或 unsigned

📝 同一組 flags 可能：
- 視為 unsigned → CF=1 表 `a<b`
- 視為 signed → SF=OF 表 `a>b`

## ⭐ 經典應用速查
| 任務 | 解法 |
|------|------|
| 小寫→大寫 | `AND al, 11011111b`（清 bit 5）|
| binary 數字→ASCII | `OR al, 00110000b`（設 bits 4,5）|
| 字串加密 | `XOR each_byte, KEY` |
| 檢查多個 bit 全 set | AND 過濾 → CMP → JE |
| 找正數 | TEST 80h → LOOPNZ |
| 找非零 | CMP 0 → LOOPE |
| 偶數判斷 | TEST val, 1 → JZ |

## ⭐ Signed vs Unsigned 跳躍指令完整對照
| 比較 | Unsigned | Signed |
|------|----------|--------|
| > | **JA** | **JG** |
| ≥ | **JAE** / JNB | **JGE** / JNL |
| < | **JB** | **JL** |
| ≤ | **JBE** / JNA | **JLE** / JNG |
| = | JE | JE |
| ≠ | JNE | JNE |
