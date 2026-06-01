# CH5 條件處理 — 練習題

> 主題：AND/OR/XOR · CMP · Jcond · IF/WHILE ｜ 共 18 題（原題 5・補充 13）

---

### Q01 `原題`

在 CMP 中，如何判斷 signed 計算後的 OF？（用 AND/OR/XOR/Invert 哪一個運算）

**答案：** 用**最高位元的進位**與**次高位元的進位**做 **XOR**。

---

### Q02 `原題`

CMP 指令會改變 destination 的值嗎？若不會，為何還需要 CMP？

**答案：** **不會**。CMP 是 nondestructive subtraction，目的是更新 flags，搭配條件跳躍指令使用。

---

### Q03 `原題`

在 CMP 中，當 destination < source（signed）時，Sign flag 與 Overflow flag 的關係？

**答案：** **SF ≠ OF**。

---

### Q04 `原題`

`mov dl,-12` / `shr dl,1` / `cmp dl,-5` / `jg L1`，會跳到 L1 嗎？

**答案：** **會**。SHR 是邏輯右移，dl 變成正數（大於 −5），signed 比較 jg 成立，進入 L1。

---

### Q05 `原題`

Overflow flag 是什麼？和 Carry flag 有何不同？

**答案：** **OF** 指有號數溢位；**CF** 指無號數進位／超出範圍。

---

### Q06 `補充`

要把 AL 的小寫字母轉成大寫，用哪個 bitwise 指令？寫出指令。

**答案：** 用 AND 清除 bit 5：`and al, 11011111b`（'a'=61h → 'A'=41h）。

---

### Q07 `補充`

`mov al, 6` 後 `or al, 00110000b`，AL 變成什麼？用途為何？

**答案：** AL = 00110110b = 36h = ASCII **'6'**。用途：binary 數字轉 ASCII 數字字元。

---

### Q08 `補充`

TEST 與 AND 有何差別？舉一個檢查 bit 的例子。

**答案：** TEST 是 nondestructive AND，**不修改 destination**、只影響 flags。例：`test al, 00000011b` / `jnz ValueFound`。

---

### Q09 `補充`

unsigned 與 signed 的「大於 / 小於」分別用哪些條件跳躍？

**答案：** Unsigned：> **JA**、< **JB**、≥ JAE、≤ JBE。Signed：> **JG**、< **JL**、≥ JGE、≤ JLE。口訣：A/B=unsigned，G/L=signed。

---

### Q10 `補充`

`CMP a,b` 其中 a=00001111、b=11110110。填出 ZF/SF/CF/OF，並說明 unsigned 與 signed 的解讀。

**答案：** a−b → ZF=0, SF=0, CF=1, OF=0。Unsigned：a=15 < b=246（CF=1）；Signed：a=15 > b=−10（SF=OF）。同一組 flags 解讀不同。

---

### Q11 `補充`

LOOPZ/LOOPE 與 LOOPNZ/LOOPNE 的跳轉條件各為何？

**答案：** 兩者都先 `ECX−=1`。LOOPZ/LOOPE：ECX>0 **且 ZF=1** 才跳；LOOPNZ/LOOPNE：ECX>0 **且 ZF=0** 才跳。

---

### Q12 `補充`

把 `if (op1 == op2) X=1; else X=2;` 翻譯成組合語言。

**答案：**

```asm
mov eax, op1
cmp eax, op2
jne L1
mov X, 1
jmp L2
L1: mov X, 2
L2:
```

---

### Q13 `補充`

把 `while (eax < ebx) eax = eax + 1;` 翻譯成組合語言。

**答案：**

```asm
top:
  cmp eax, ebx
  jae next          ; 反向條件離開
  inc eax
  jmp top
next:
```

---

### Q14 `補充`

用短路求值翻譯 `if (al > bl) AND (bl > cl) X = 1;`（unsigned）。

**答案：**

```asm
cmp al, bl
jbe next            ; 任一 false 就跳出
cmp bl, cl
jbe next
mov X, 1
next:
```

---

### Q15 `補充`

bit-mapped set 的交集、聯集、補集分別用哪個 bitwise 指令？

**答案：** 交集 = **AND**、聯集 = **OR**、補集 = **NOT**。

---

### Q16 `補充`

如何判斷 ESI 指向的數值是偶數？

**答案：** `test DWORD PTR [esi], 1` 取 bit 0，若 ZF=1（bit0=0）→ 偶數，`jz IsEven`。

---

### Q17 `補充`

XOR 為什麼能同時用於加密與解密？

**答案：** 因為 `(X ⊕ KEY) ⊕ KEY = X`，對同一把 KEY 做兩次 XOR 會還原，所以加密與解密可用同一段程式。

---

### Q18 `補充`

用短路求值翻譯 `if (al > bl) OR (bl > cl) X = 1;`（unsigned）。

**答案：**

```asm
cmp al, bl
ja  L1              ; 任一 true 就執行 body
cmp bl, cl
jbe next            ; 都 false 才跳出
L1: mov X, 1
next:
```

---

### Q19 `補充`

如何分別把 ZF 設為 1、把 CF 清為 0？

**答案：** Set ZF：`and op, 0`（結果 0）；Clear CF：`CLC`（Set CF 用 STC）。
