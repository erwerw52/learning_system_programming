# CH4 程序 — 練習題

> 主題：Stack · CALL/RET · Irvine32 Library ｜ 共 17 題（原題 9・補充 8）

---

### Q01 `原題`

執行 RET 時，CPU 從 stack 取出什麼數值、放入哪個暫存器？

**答案：** 取出返回的記憶體位址（return address），放入 **EIP**。

---

### Q02 `原題`

POP 指令執行時的兩個主要步驟為何？

**答案：** 1. 把 stack 頂端內容複製到暫存器／變數；2. **ESP += bytes**（依 operand 大小）。

---

### Q03 `原題`

`mov eax,35` 後依序 WriteBin / WriteDec / WriteHex（各接 Crlf），輸出為何？

**答案：**

```
0000000000110101b
53
35h
```

---

### Q04 `原題`

要保存所有 32 位元通用暫存器的狀態，用哪個指令？恢復指令為何？

**答案：** 用 **PUSHAD** 保存，用 **POPAD** 以相反順序恢復。

---

### Q05 `原題`

`push eax`、`push ebx`、`pop ecx` 之後，ESP 相對原始值如何變化？

**答案：** 兩次 push（−8）、一次 pop（+4）→ ESP = **原始值 − 4**。

---

### Q06 `原題`

Local Label 與 Global Label 在程序定義中的差異？

**答案：** Local Label 僅在定義它的 procedure 內可見；Global Label（雙冒號 `::`）在整個程式都可見。

---

### Q07 `原題`

寫一段程式：清空畫面、暫停 1000 ms、再顯示暫存器狀態。

**答案：**

```asm
call Clrscr
mov  eax, 1000
call Delay
call DumpRegs
```

---

### Q08 `原題`

`mov eax,1` / `mov ebx,2` / `push eax` / `push ebx` / `pop eax` / `pop ebx`，最後 EAX = ?

**答案：** **EAX = 2**（LIFO：先 pop 到 eax 的是後 push 的 ebx 值 2）。

---

### Q09 `原題`

PROC 中的 USES 運算子有什麼作用？

**答案：** 與 PROC 一起使用，列出要保存的暫存器；MASM 會自動在開頭 PUSH、結尾 POP（順序相反）。

---

### Q10 `補充`

CALL 指令執行時的兩個步驟為何？

**答案：** 1. 把下一條指令的 offset（return address）**push 到 stack**；2. 把被呼叫程序的位址載入 **EIP**。

---

### Q11 `補充`

main 呼叫 Sub1，Sub1 呼叫 Sub2，Sub2 呼叫 Sub3。執行到 Sub3 時，stack 上有幾個 return address？

**答案：** **3 個**（return to main、to Sub1、to Sub2）。每進一層多一個，每 RET 取走一個。

---

### Q12 `補充`

用 Irvine32 寫出「黃字藍底」顯示字串的設色指令。

**答案：** `mov eax, yellow + (blue * 16)` 再 `call SetTextColor`（背景要 ×16）。

---

### Q13 `補充`

呼叫 ReadString 與 DumpMem 各需要設定哪些暫存器？

**答案：** **ReadString**：EDX=緩衝區位址、ECX=最大字元數，回傳 EAX=實際字元數。**DumpMem**：ESI=起始位址、ECX=單位個數、EBX=單位大小。

---

### Q14 `補充`

`INCLUDE Irvine32.inc`、`irvine32.lib`、`kernel32.lib` 三者各扮演什麼角色？

**答案：** `INCLUDE Irvine32.inc` 引入函式 prototype；`irvine32.lib` 提供函式實作；`kernel32.lib` 由 Win32 SDK 提供，連到底層 OS 呼叫。

---

### Q15 `補充`

PUSH 指令執行時的兩個步驟為何？

**答案：** 1. **ESP −= 4（或 2）**（往較低位址成長）；2. 把值複製到 ESP 指到的位置。

---

### Q16 `補充`

Runtime Stack 的四種主要用途為何？

**答案：** 1. 暫存暫存器；2. 儲存 return address（call/ret 自動）；3. 傳遞 arguments；4. 配置 local variables。

---

### Q17 `補充`

用 PUSHFD/POPFD 將 EFLAGS 暫存到變數 saveFlags，之後再還原。

**答案：**

```asm
pushfd            ; flags → stack
pop  saveFlags    ; → 變數
; ...
push saveFlags    ; 變數 → stack
popfd             ; → flags
```

---

### Q18 `補充`

C 程式中什麼情況下函式不需要 prototype？

**答案：** 若該函式的**定義出現在呼叫它的程式（如 main）之前**，編譯器已知其簽章，就不需要 prototype。
