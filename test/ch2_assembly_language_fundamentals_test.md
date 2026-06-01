# CH2 AL 基礎 — 練習題

> 主題：MASM 元素 · 資料定義 · 符號常數 ｜ 共 17 題（原題 9・補充 8）

---

### Q01 `原題`

寫組合語言程式時，為什麼需要 `main PROC` 與 `main ENDP`？

**答案：** PROC 標記 procedure 起點、ENDP 標記其結尾，讓組譯器標記出該程式區段（procedure 的範圍）。

---

### Q02 `原題`

如何用目前位置計數器 `$` 計算名為 myArray 的 DWORD 陣列元素個數？寫出運算式。

**答案：** `($ − myArray) / 4`（因 $ 以 byte 計，DWORD 每元素 4 bytes）。

---

### Q03 `原題`

`msg BYTE "ABC",0`。msg 佔幾 bytes？msg+1 讀到什麼？msg+3 是什麼？

**答案：** 1. **4 bytes**；2. **'B'**；3. **null（0）**。

---

### Q04 `原題`

什麼是 directive？它和 instruction 有何不同？

**答案：** instruction 是 CPU 執行、會被組譯成機器碼的指令；directive 只是給 assembler 的指示，**不會轉成機器碼**、執行期不會跑。

---

### Q05 `原題`

Equal-Sign（`=`）與 `EQU` 在「重定義」上的主要差別？

**答案：** `=` 可以被重複給予新值（可重定義）；`EQU` 一旦定義後就不能修改。

---

### Q06 `原題`

NOP 指令如何貢獻 CPU 效能？

**答案：** 可把程式碼對齊到偶數位址或 4 的倍數位址，提高 CPU 的指令獲取（fetch）效率。

---

### Q07 `原題`

為什麼 `msg_g WORD "AB"` 不像 `msg BYTE "ABC",0` 需要 null 結尾？

**答案：** `WORD "AB"` 被視為一個 WORD 數值，不是字串；只有字串（字元陣列）才需 null 終止符。

---

### Q08 `原題`

`val1 DWORD A2345678h`，起始位址 0000、little-endian。位址 0002 存的是？若不寫 h 會怎樣？

**答案：** 1. **34h**；2. 沒寫 h 會**語法錯誤**，因為 A 不是有效的十進位數字。

---

### Q09 `原題`

`$`（目前位置計數器）最常用來做什麼？

**答案：** 回傳**當前程式敘述的 offset**，常用來計算陣列大小。

---

### Q10 `補充`

`A5h` 是有效的 hex literal 嗎？為什麼？正確怎麼寫？

**答案：** **不是**。以字母開頭的 hex 必須加前導 0，否則會被當成 identifier。正確：`0A5h`。

---

### Q11 `補充`

求 MASM 運算式的值：`-(3+4)*(6-1)`、`25 mod 3`、`16/5`。

**答案：** 分別為 **−35**、**1**、**3**（整數除法）。

---

### Q12 `補充`

下列各佔幾 bytes？`4 DUP("STACK")` 與 `BYTE 10, 3 DUP(0), 20`。

**答案：** `4 DUP("STACK")` = 4×5 = **20 bytes**；`10,3 DUP(0),20` = 1+3+1 = **5 bytes**。

---

### Q13 `補充`

`.data` 與 `.data?` 對 EXE 檔案大小有何差別？

**答案：** `.data?` 為未初始化資料，在 EXE 中只做 reserve，**不會增大檔案**；`.data` 會把初始值寫進檔案。但載入記憶體後兩者佔用相同。

---

### Q14 `補充`

`matrix1 EQU 10*10` 與 `matrix2 EQU <10*10>`，用於 WORD 時結果分別為？

**答案：** `EQU 10*10`（無 <>）→ 被計算成 **100**；`EQU <10*10>` → 文字直接複製成 **10*10**。

---

### Q15 `補充`

identifier（識別字）的命名規則為何？

**答案：** 1–247 個字元、不分大小寫；**首字必須是字母、`_`、`@` 或 `$`**，不可用數字開頭。

---

### Q16 `補充`

列出 reserved words（保留字）的五大類。

**答案：** 1. Instruction mnemonics（ADD/MOV…）2. Directives（.data/.code…）3. Type attributes（BYTE/WORD…）4. Operators（+ − …）5. Predefined symbols（@data…）。

---

### Q17 `補充`

`val DWORD 12345678h` 在 little-endian 記憶體中，由低位址到高位址的排列為何？

**答案：** **78 56 34 12**（最低有效位元組放在最低位址）。

---

### Q18 `補充`

指令的 operand 有哪四種類型？各舉一例。

**答案：** **constant**（`mov ax,96`）、**constant expression**（`mov ax,5+4`）、**register**（`mov ax,bx`）、**memory**（`mov count,bx`）。前兩者統稱 immediate value。
