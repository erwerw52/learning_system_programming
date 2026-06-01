# CH3 資料傳送／運算 — 練習題

> 主題：MOV · 旗標 · 運算子 · 定址 · LOOP ｜ 共 17 題（原題 3・補充 14）

---

### Q01 `原題`

Carry Flag (CF) 的角色是什麼？

**答案：** 當運算結果是超出範圍的**無號數**時，CF = 1。

---

### Q02 `原題`

陣列 `arrayD DWORD 1,2,3`，把值重排為 3, 1, 2。

**答案：**

```asm
mov  eax, arrayD
xchg eax, [arrayD + 4]
xchg eax, [arrayD + 8]
mov  arrayD, eax
```

---

### Q03 `原題`

mov 指令可以做記憶體對記憶體的搬移嗎？

**答案：** **不行**，mov 不支援兩個 memory operand 之間的搬移（最多一個 memory operand）。

---

### Q04 `補充`

`count SWORD -16`，執行 `movzx ecx, count` 會發生什麼問題？該用什麼指令？

**答案：** −16 = FFF0h，MOVZX 高位補 0 → 0000FFF0h = **65520**（不再是 −16，錯誤）。signed 應改用 **MOVSX**（高位補符號位元）。

---

### Q05 `補充`

給定 `array1 WORD 30 DUP(?),0,0`，求 TYPE、LENGTHOF、SIZEOF。

**答案：** TYPE = **2**；LENGTHOF = 30+2 = **32**；SIZEOF = 32×2 = **64**。

---

### Q06 `補充`

給定 `digitStr BYTE "12345678",0`，LENGTHOF 與 SIZEOF 各是多少？

**答案：** 8 個字元 + 1 個 null = LENGTHOF **9**；TYPE=1 → SIZEOF **9**。

---

### Q07 `補充`

`myDouble DWORD 12345678h`（little-endian）。`WORD PTR myDouble`、`BYTE PTR [myDouble+1]` 各取到什麼？

**答案：** 記憶體：78 56 34 12。`WORD PTR myDouble` = **5678h**；`BYTE PTR [myDouble+1]` = **56h**。

---

### Q08 `補充`

`myBytes BYTE 12h,34h,56h,78h`。`WORD PTR [myBytes]` 與 `DWORD PTR myBytes` 各為何？

**答案：** `WORD PTR [myBytes]` = **3412h**；`DWORD PTR myBytes` = **78563412h**（little-endian 反讀）。

---

### Q09 `補充`

`mov al, -128` 後 `neg al`，結果與 OF 為何？

**答案：** AL = 10000000b（仍是 −128），**OF = 1**，因為 +128 超出 byte signed 範圍。

---

### Q10 `補充`

`mov ecx, 0` 後接 `loop L1`，迴圈會跑幾次？

**答案：** **2³² = 4,294,967,296 次**。ECX=0 先減 1 變成 FFFFFFFFh（環繞），是經典陷阱。

---

### Q11 `補充`

用 indexed 定址寫出存取 DWORD 陣列第 4 個元素（索引 esi=3）的指令。

**答案：**

```asm
mov esi, 3
mov eax, arrayD[esi * 4]      ; 或 arrayD[esi * TYPE arrayD]
```

---

### Q12 `補充`

寫出 `Rval = Xval - (-Yval + Zval)`（不可修改 Xval/Yval/Zval）。

**答案：**

```asm
mov ebx, Yval
neg ebx
add ebx, Zval
mov eax, Xval
sub eax, ebx
mov Rval, eax
```

---

### Q13 `補充`

列出五大資料運算子各回傳什麼：OFFSET、TYPE、LENGTHOF、SIZEOF、PTR。

**答案：** **OFFSET**：label 到 segment 起點的距離；**TYPE**：單一元素 bytes；**LENGTHOF**：元素個數；**SIZEOF**：LENGTHOF×TYPE；**PTR**：覆寫 label 預設型別。

---

### Q14 `補充`

XCHG 的使用規則為何？兩個 memory 要對換該怎麼做？

**答案：** 至少一個 operand 必須是 register、不可 immediate、不可 mem-to-mem。兩個 memory 對換要用 register 當橋：`mov ax,val1` / `xchg ax,val2` / `mov val1,ax`。

---

### Q15 `補充`

`mov bl, 1` 後 `sub bl, 2`，BL 值與 Sign Flag 為何？

**答案：** BL = FFh = −1，最高位 = 1 → **SF = 1**。

---

### Q16 `補充`

`mov al, 80h` 後 `add al, 92h`，OF 為何？（用加法 rule of thumb 說明）

**答案：** 80h=−128、92h=−110，兩個**負數相加結果變正** → **OF = 1**。

---

### Q17 `補充`

JMP 與 LOOP 的差別？

**答案：** **JMP** 無條件跳躍（EIP←target）；**LOOP** 先 `ECX−=1`，若 ECX>0 才跳（條件跳躍）。

---

### Q18 `補充`

`arrayW WORD 1000h,2000h,3000h`，`mov ax, [arrayW+2]` 與 `mov ax, [arrayW+4]` 各為何？

**答案：** `[arrayW+2]` = **2000h**；`[arrayW+4]` = **3000h**（每個 WORD 2 bytes）。
