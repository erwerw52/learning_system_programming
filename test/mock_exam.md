# Assembly Language for x86 Processors — 模擬期末考

> **出題依據**：`textbooks/` 內 Ch1~Ch7 完整課程筆記
> **範圍**：Chapter 1 ~ Chapter 7（**至 LEA 指令之前**；不含 LEA、遞迴、INVOKE 展開、8/16/64-bit 進階參數傳遞）
> **教科書**：Kip Irvine, *Assembly Language for x86 Processors*, 7th Ed.（MASM / Irvine32）
> **題型**：Part A 選擇題 / Part B 簡答題
> **題目為英文，答案與解析為中文。**

---

## Part A — Multiple Choice（選擇題）

### Chapter 1 — Basic Concepts

**A1.** Which statement about the relationship between languages is correct?
- (A) Assembly language and machine language have a one-to-many relationship
- (B) Assembly language and machine language have a one-to-one relationship
- (C) Assembly language and C++ have a one-to-one relationship
- (D) Machine language and C++ have a one-to-one relationship

**答案：(B)**
解析：組合語言 ↔ 機器語言為 **one-to-one**；C++/Java ↔ 組合語言為 **one-to-many**。

---

**A2.** What is the role of a **linker**?
- (A) Translates assembly source code into machine code
- (B) Traces program execution and examines memory
- (C) Combines one or more object files into a single executable file
- (D) Loads the executable into memory and runs it

**答案：(C)**
解析：Linker 把 assembler 產生的多個 object files 結合成單一可執行檔。追蹤執行是 debugger；翻譯原始碼是 assembler。

---

**A3.** Is assembly language portable, and why?
- (A) Yes, because it runs on any operating system
- (B) Yes, because it is translated one-to-one
- (C) No, because it is tied to a specific processor family
- (D) No, because it cannot be assembled

**答案：(C)**
解析：組合語言**不可移植**，因為它綁定特定處理器家族（processor family）。

---

**A4.** What is the decimal value of the binary number `100101`?
- (A) 35
- (B) 37
- (C) 41
- (D) 45

**答案：(B) 37**
解析：32 + 4 + 1 = 37（對應筆記中 37 = 100101₂ 的反向）。

---

**A5.** Convert decimal `422` to hexadecimal.
- (A) 1A6h
- (B) 1C6h
- (C) 2A6h
- (D) 1B6h

**答案：(A) 1A6h**
解析：422 / 16 = 26 餘 6；26 / 16 = 1 餘 A；1 / 16 = 0 餘 1 → 由下往上讀 = 1A6h。

---

**A6.** Variable `var1` is at address `00400020h` and the next variable starts at `0040006Ah`. How many bytes does `var1` occupy?
- (A) 4Ah = 74 bytes
- (B) 4Ah = 64 bytes
- (C) 40h = 64 bytes
- (D) 6Ah = 106 bytes

**答案：(A) 74 bytes**
解析：0040006Ah − 00400020h = 4Ah = 74（十進位）。

---

**A7.** Which of the following hexadecimal bytes represents a **negative** signed integer?
- (A) 7Fh
- (B) 6Ah
- (C) 9Dh
- (D) 41h

**答案：(C) 9Dh**
解析：最高 hex digit > 7 即為負數（sign bit = 1）。9 > 7 → 負；7F、6A、41 皆為正。

---

**A8.** Given Boolean operator precedence, how is `¬X ∨ Y` evaluated?
- (A) OR first, then NOT
- (B) NOT first, then OR
- (C) AND first, then OR
- (D) Left to right

**答案：(B) NOT 先，再 OR**
解析：布林優先序為 **NOT > AND > OR**（括號最大）。

---

**A9.** Which register is used as a **loop counter** by the LOOP instruction?
- (A) EAX
- (B) ECX
- (C) ESP
- (D) EBP

**答案：(B) ECX**
解析：ECX = **C**ounter，LOOP 指令以它計數。EAX=Accumulator、ESP=Stack Pointer、EBP=Base Pointer。

---

**A10.** Which status flag is set when an **unsigned** operation produces a result that is out of range?
- (A) Overflow Flag (OF)
- (B) Sign Flag (SF)
- (C) Carry Flag (CF)
- (D) Zero Flag (ZF)

**答案：(C) Carry Flag (CF)**
解析：CF 表 **unsigned** 溢位；OF 表 **signed** 溢位（口訣：Carry=u**n**signed、**O**verflow=**S**igned）。

---

**A11.** What does the **EIP** register hold?
- (A) The top of the stack
- (B) The address of the next instruction to be executed
- (C) The base of the current stack frame
- (D) The processor status flags

**答案：(B)**
解析：EIP（Instruction Pointer）儲存下一條要執行的指令位址。

---

**A12.** In the 64-bit x86-64 architecture, how many general-purpose registers are there and what is the instruction pointer called?
- (A) 8 registers, EIP
- (B) 8 registers, RIP
- (C) 16 registers, RIP
- (D) 32 registers, RIP

**答案：(C) 16 個暫存器，RIP**
解析：x86-64 有 16 個 64-bit 通用暫存器（新增 R8~R15），指令指標為 RIP。

---

### Chapter 2 — Assembly Language Fundamentals

**A13.** Why must the hex literal for "A5" be written as `0A5h` in MASM?
- (A) To indicate it is a signed value
- (B) To prevent the assembler from interpreting it as an identifier
- (C) Because hex values must always be 3 digits
- (D) To align it to a doubleword boundary

**答案：(B)**
解析：以**字母開頭**的 hex 數字必須加前綴 0，否則 assembler 會把它當成 identifier（識別字）。

---

**A14.** Which directive declares a 32-bit **signed** integer?
- (A) DWORD
- (B) SDWORD
- (C) SWORD
- (D) QWORD

**答案：(B) SDWORD**
解析：S 前綴 = Signed；SDWORD 為 32-bit 有號整數。DWORD 是 32-bit 無號。

---

**A15.** For `val1 DWORD 12345678h` on an x86 (little-endian) machine, what byte is stored at the **lowest** address?
- (A) 12h
- (B) 34h
- (C) 56h
- (D) 78h

**答案：(D) 78h**
解析：x86 採 **Little Endian**，最低有效 byte（LSB = 78h）放在最低位址。

---

**A16.** What is the result of the following data declarations' `LENGTHOF` and `SIZEOF`?
```
var3 BYTE 4 DUP("STACK")
```
- (A) LENGTHOF = 5, SIZEOF = 5
- (B) LENGTHOF = 4, SIZEOF = 4
- (C) LENGTHOF = 20, SIZEOF = 20
- (D) LENGTHOF = 4, SIZEOF = 20

**答案：(C) LENGTHOF = 20, SIZEOF = 20**
解析：`"STACK"` 是 5 個 byte，重複 4 次 = 20 bytes；每個元素都是 BYTE（TYPE=1），故 LENGTHOF = SIZEOF = 20。

---

**A17.** What is the difference between `.data` and `.data?`?
- (A) `.data?` is for code, `.data` is for data
- (B) `.data?` declares uninitialized data and keeps the EXE file smaller
- (C) `.data?` cannot be accessed at runtime
- (D) There is no difference after loading

**答案：(B)**
解析：`.data?` 宣告未初始化資料，在 EXE 中只是 reserve，**降低檔案大小**；但載入記憶體後兩者佔用相同。

---

**A18.** Which directive defines a symbolic constant that **can be redefined** later?
- (A) `=` (Equal-Sign)
- (B) `EQU`
- (C) `TEXTEQU`
- (D) Both A and C

**答案：(D) `=` 與 `TEXTEQU` 皆可重新定義**
解析：`=` 可重新定義（整數）；`TEXTEQU` 可重新定義（文字 macro）；**`EQU` 不可**重新定義。

---

**A19.** Given `List BYTE 10, 20, 30, 40` immediately followed by `ListSize = ($ - List)`, what is `ListSize`?
- (A) 1
- (B) 4
- (C) 8
- (D) 40

**答案：(B) 4**
解析：`$` 回傳當前 offset（byte 為單位），減去 List 起點 = 4 bytes（因 BYTE 陣列每元素 1 byte）。若是 WORD 要 ÷2、DWORD 要 ÷4。

---

**A20.** In the instruction format `Label: Mnemonic Operand(s) ; Comment`, which part is **required**?
- (A) Label
- (B) Mnemonic
- (C) Operand
- (D) Comment

**答案：(B) Mnemonic（指令助記符）**
解析：只有 Mnemonic 是必須的；Label、Operand（依指令）、Comment 皆為選擇性。

---

### Chapter 3 — Data Transfers, Addressing, and Arithmetic

**A21.** Which of the following MOV instructions is **legal**?
- (A) `mov esi, wVal`（ESI 32-bit, wVal WORD）
- (B) `mov eip, dVal`
- (C) `mov bVal2, bVal`（兩者皆 memory）
- (D) `mov eax, ebx`

**答案：(D)**
解析：MOV 三大規則：兩 operand 大小須相同（排除 A）、EIP 不可當 destination（排除 B）、不可 mem-to-mem（排除 C）。

---

**A22.** `count` is declared `SWORD -16`. After `movzx ecx, count`, what is the problem?
- (A) Nothing, ECX = -16 correctly
- (B) ECX = 65520, because MOVZX zero-extends and destroys the sign
- (C) It is a syntax error
- (D) ECX = 0

**答案：(B)**
解析：−16 = FFF0h，MOVZX 高位補 0 → 0000FFF0h = 65520，不再是 −16。有號值應用 **MOVSX**（符號擴充）。

---

**A23.** Which instruction does **NOT** affect any flags?
- (A) ADD
- (B) SUB
- (C) MOV
- (D) NEG

**答案：(C) MOV**
解析：MOV 不影響任何 flag；ADD/SUB/NEG 皆會更新狀態旗標。

---

**A24.** `mov al, 0FFh` then `add al, 1`. What are AL and CF?
- (A) AL = 00h, CF = 1
- (B) AL = 00h, CF = 0
- (C) AL = 100h, CF = 1
- (D) AL = FEh, CF = 0

**答案：(A) AL = 00h, CF = 1**
解析：FFh（255）+ 1 超出 byte 無號上限 → AL = 00h，MSB 進位 → CF = 1。

---

**A25.** Under what condition is the **Overflow Flag (OF)** set to 1 during addition?
- (A) When two operands of different signs are added
- (B) When two positive numbers produce a negative result, or two negatives produce a positive
- (C) Whenever a carry comes out of the MSB
- (D) Whenever the result is zero

**答案：(B)**
解析：OF 加法 rule of thumb：兩正相加得負，或兩負相加得正。不同號相加 OF 必為 0。

---

**A26.** For `myDouble DWORD 12345678h`, what does `mov al, BYTE PTR [myDouble+1]` load into AL?
- (A) 78h
- (B) 56h
- (C) 34h
- (D) 12h

**答案：(B) 56h**
解析：Little Endian：offset0=78, offset1=56, offset2=34, offset3=12。故 [myDouble+1] = 56h。

---

**A27.** Which operator returns the **number of elements** in an array declaration?
- (A) TYPE
- (B) SIZEOF
- (C) LENGTHOF
- (D) OFFSET

**答案：(C) LENGTHOF**
解析：LENGTHOF = 元素個數；TYPE = 單元素 bytes；SIZEOF = LENGTHOF × TYPE；OFFSET = 距 segment 起點的位址。

---

**A28.** What is wrong with `inc [esi]` when ESI points to a WORD variable?
- (A) ESI cannot be used for indirect addressing
- (B) The operand size is ambiguous; needs `inc WORD PTR [esi]`
- (C) INC cannot operate on memory
- (D) Nothing is wrong

**答案：(B)**
解析：assembler 無法判斷 [esi] 指向 byte/word/dword，須用 PTR 指定大小：`inc WORD PTR [esi]`。

---

**A29.** Consider the code:
```
mov ax, 6
mov ecx, 4
L1:
    inc ax
    loop L1
```
What is the final value of AX?
- (A) 4
- (B) 6
- (C) 10
- (D) 24

**答案：(C) 10**
解析：AX 從 6 開始，LOOP 執行 4 次（每次 ECX−1，>0 才跳），inc 4 次 → 6 + 4 = 10。

---

**A30.** What happens if `LOOP` executes when `ECX = 0`?
- (A) It jumps 0 times (falls through immediately)
- (B) It loops exactly once
- (C) ECX wraps around and it loops 2³² times
- (D) It causes a runtime error

**答案：(C) 環繞執行 2³² 次**
解析：LOOP 先做 ECX−1，0−1 在無號下環繞為 FFFFFFFFh，要一路減到 0 才停 → 4,294,967,296 次（經典陷阱）。

---

### Chapter 4 — Procedures

**A31.** What are the two steps performed by the **PUSH** instruction (32-bit)?
- (A) ESP += 4, then write the value to [ESP]
- (B) ESP −= 4, then write the value to [ESP]
- (C) Write the value to [ESP], then ESP += 4
- (D) Write the value to [ESP], then ESP −= 4

**答案：(B)**
解析：PUSH 先 ESP −= 4（堆疊向低位址成長），再把值寫入 [ESP]。

---

**A32.** What two actions does **CALL** perform?
- (A) Pops the return address into EIP, then jumps
- (B) Pushes the return address (offset of next instruction) onto the stack, then loads the procedure address into EIP
- (C) Pushes EBP, then sets EBP = ESP
- (D) Saves all registers, then jumps

**答案：(B)**
解析：CALL：① push 下一條指令 offset（return address）；② EIP ← 被呼叫程序位址。RET 則相反（pop → EIP）。

---

**A33.** What does `PROC USES esi ecx` cause MASM to generate?
- (A) Declares esi and ecx as parameters
- (B) Automatic PUSH of esi, ecx at entry and POP in reverse order at exit
- (C) Reserves local variables named esi and ecx
- (D) Prevents esi and ecx from being modified

**答案：(B)**
解析：USES 讓 MASM 自動在開頭 PUSH 指定暫存器、結尾以相反順序 POP。

---

**A34.** A procedure pushes registers in the order `push esi` / `push ecx` / `push ebx`. To restore them correctly, in what order must they be popped?
- (A) esi, ecx, ebx
- (B) ebx, ecx, esi
- (C) ecx, ebx, esi
- (D) Any order

**答案：(B) ebx, ecx, esi（相反順序）**
解析：堆疊為 LIFO，POP 順序必須與 PUSH 相反。

---

**A35.** Which Irvine32 procedure expects the **offset of a null-terminated string in EDX**?
- (A) WriteInt
- (B) WriteDec
- (C) WriteString
- (D) WriteHex

**答案：(C) WriteString**
解析：WriteString 以 EDX = 字串位址輸出 null-terminated 字串；WriteInt/WriteDec/WriteHex 皆以 EAX 為輸入。

---

**A36.** To set yellow text on a blue background with `SetTextColor`, what value goes in EAX?
- (A) `yellow + blue`
- (B) `yellow * 16 + blue`
- (C) `yellow + (blue * 16)`
- (D) `blue + yellow`

**答案：(C) `yellow + (blue * 16)`**
解析：EAX = 前景 + (背景 × 16)，背景顏色須乘以 16。

---

**A37.** Which label declaration makes a label visible **outside** its own procedure?
- (A) `L1:`（single colon）
- (B) `L1::`（double colon）
- (C) `L1 PROC`
- (D) `L1 EQU $`

**答案：(B) `L1::`（雙冒號 = global label）**
解析：單冒號為 local label（僅本 procedure 可見）；雙冒號為 global label（全程式可見）。

---

### Chapter 5 — Conditional Processing

**A38.** Which instruction performs a bitwise AND but does **NOT** modify the destination, only the flags?
- (A) AND
- (B) TEST
- (C) CMP
- (D) OR

**答案：(B) TEST**
解析：TEST 執行 nondestructive AND，只影響 flags（尤其 ZF），不改變 operand。

---

**A39.** To convert the lowercase ASCII letter in AL to uppercase, which instruction is used?
- (A) `or al, 00100000b`
- (B) `and al, 11011111b`
- (C) `xor al, 00100000b`
- (D) `or al, 00110000b`

**答案：(B) `and al, 11011111b`**
解析：小寫與大寫只差 bit 5（'a'=61h, 'A'=41h）。用 AND 清除 bit 5 → 轉大寫。

---

**A40.** After `mov al, 4` then `cmp al, 5` (unsigned), which flags are set?
- (A) ZF = 1
- (B) CF = 1, ZF = 0
- (C) CF = 0, ZF = 0
- (D) OF = 1

**答案：(B) CF = 1, ZF = 0**
解析：unsigned 比較中 dest < source → CF = 1、ZF = 0（4 < 5）。

---

**A41.** Which bitwise instruction implements the **intersection** of two bit-mapped sets?
- (A) OR
- (B) XOR
- (C) AND
- (D) NOT

**答案：(C) AND**
解析：交集用 AND、聯集用 OR、補集用 NOT。

---

**A42.** Which jump is taken for an **unsigned** "jump if greater" after CMP?
- (A) JG
- (B) JA
- (C) JNL
- (D) JGE

**答案：(B) JA（Jump if Above）**
解析：unsigned 用 A/B（Above/Below）；signed 才用 G/L（Greater/Less）。JG/JNL/JGE 都是有號。

---

**A43.** What is the jump condition of `LOOPNZ` (LOOPNE)?
- (A) ECX > 0
- (B) ECX > 0 and ZF = 1
- (C) ECX > 0 and ZF = 0
- (D) ZF = 0 only

**答案：(C) ECX > 0 且 ZF = 0**
解析：LOOPNZ/LOOPNE 先 ECX−1，若 ECX>0 **且** ZF=0 則跳。LOOPZ/LOOPE 則需 ZF=1。

---

**A44.** Why are `PUSHFD` / `POPFD` used inside the LOOPNZ search loop in the notes?
- (A) To save the loop counter
- (B) To preserve the flags from TEST/CMP because `add esi, ...` would overwrite them
- (C) To push the array address
- (D) To clear the carry flag

**答案：(B)**
解析：`add esi, TYPE array` 會修改 flags，破壞 TEST/CMP 的結果，所以在中間運算前後用 PUSHFD/POPFD 保存與還原 flags，供 LOOPNZ 使用。

---

**A45.** Why does XOR encryption use the **same** procedure for both encrypting and decrypting?
- (A) Because XOR is faster than AND
- (B) Because `(X ⊕ KEY) ⊕ KEY = X`
- (C) Because XOR clears the carry flag
- (D) Because XOR does not modify operands

**答案：(B)**
解析：XOR 兩次同一 key 還原原值，故加密與解密可用同一段程式。

---

**A46.** To test whether the doubleword pointed to by EDI is **even**, which is correct?
- (A) `test DWORD PTR [edi], 1` then `jz IsEven`
- (B) `test DWORD PTR [edi], 1` then `jnz IsEven`
- (C) `cmp DWORD PTR [edi], 0` then `je IsEven`
- (D) `and DWORD PTR [edi], 0` then `jz IsEven`

**答案：(A)**
解析：偶數 bit 0 = 0。`test ..., 1` 萃取 bit 0；若為 0 → ZF=1 → JZ 跳到 IsEven。

---

### Chapter 6 — Integer Arithmetic (Shift / Rotate)

**A47.** What is the difference between a **logical** shift and an **arithmetic** shift?
- (A) Logical fills with the sign bit; arithmetic fills with 0
- (B) Logical fills with 0; arithmetic right-shift fills with the sign bit
- (C) They are identical
- (D) Logical is for left shifts only

**答案：(B)**
解析：邏輯移位（SHL/SHR）一律補 0；算術右移（SAR）補符號位元以保留正負號。

---

**A48.** Which instruction should be used for fast **signed** division by a power of 2?
- (A) SHR
- (B) SAR
- (C) ROR
- (D) SHL

**答案：(B) SAR**
解析：有號數除以 2ⁿ 用 SAR（補符號位元）；無號數用 SHR（補 0）。

---

**A49.** `mov al, 6Bh` then `shr al, 1`. What is AL?
- (A) D6h
- (B) 35h
- (C) 3Bh
- (D) B5h

**答案：(B) 35h**
解析：6Bh = 0110 1011，邏輯右移 1 位補 0 → 0011 0101 = 35h。

---

**A50.** `mov al, 8Ch` then `sar al, 1`. What is AL?
- (A) 46h
- (B) C6h
- (C) 18h
- (D) 8Ch

**答案：(B) C6h**
解析：8Ch = 1000 1100（負數），算術右移 1 位補符號位元 1 → 1100 0110 = C6h。

---

**A51.** What is the key difference between a **shift** and a **rotate**?
- (A) Shift wraps bits around; rotate discards bits
- (B) Shift discards bits (into CF); rotate wraps the bit around to the other end
- (C) They both discard bits
- (D) Rotate only works on signed numbers

**答案：(B)**
解析：移位會把移出的位元丟到 CF（遺失）；旋轉（ROL/ROR）把移出的位元繞回另一端（不遺失）。

---

**A52.** To multiply EAX by 36 using shifts, the notes decompose 36 as:
- (A) (EAX << 6) + (EAX << 0)
- (B) (EAX << 5) + (EAX << 2)
- (C) (EAX << 4) + (EAX << 2)
- (D) (EAX << 5) + (EAX << 1)

**答案：(B) (EAX << 5) + (EAX << 2)**
解析：36 = 32 + 4 = 2⁵ + 2² → `shl eax,5` 加 `shl ebx,2`。

---

### Chapter 7 — Advanced Procedures (before LEA)

**A53.** What is another name for a **stack frame**?
- (A) Data segment
- (B) Activation record
- (C) Heap block
- (D) Return address

**答案：(B) Activation record（活化記錄）**
解析：Stack frame 又稱 activation record，於呼叫程序時建立、回傳時釋放。

---

**A54.** After `push ebp` / `mov ebp, esp`, where is the **first** stack parameter located?
- (A) [ebp]
- (B) [ebp+4]
- (C) [ebp+8]
- (D) [ebp-4]

**答案：(C) [ebp+8]**
解析：[ebp] 存舊 EBP、[ebp+4] 存 return address、[ebp+8] 是第一個參數、[ebp+12] 第二個。

---

**A55.** Where are **local variables** located relative to EBP?
- (A) Positive offsets, e.g. [ebp+8]
- (B) Negative offsets, e.g. [ebp-4]
- (C) In the data segment
- (D) In EDX:EAX

**答案：(B) 負偏移，如 [ebp-4]**
解析：參數在 EBP 正方向，區域變數在 EBP 負方向。

---

**A56.** Under the **C calling convention**, who cleans the parameters off the stack, and how?
- (A) The callee, using `ret n`
- (B) The caller, using `add esp, n` after the call
- (C) The operating system
- (D) Nobody

**答案：(B) Caller，用 `add esp, n`**
解析：C 慣例由呼叫者清堆疊；STDCALL 由被呼叫者用 `ret n` 清。

---

**A57.** Why can the **C calling convention** support functions with a **variable number of arguments** (e.g. `printf`)?
- (A) Because the callee always knows the argument count
- (B) Because only the caller knows how many arguments were actually passed, and the caller cleans the stack
- (C) Because C uses registers for all parameters
- (D) Because STDCALL is faster

**答案：(B)**
解析：C 慣例由 caller 清堆疊，只有 caller 知道實際傳了幾個參數，故能支援不定數目參數。STDCALL 由 callee 清，必須事先知道參數數量。

---

**A58.** To create space for two DWORD local variables, which instruction is used after `mov ebp, esp`?
- (A) `add esp, 8`
- (B) `sub esp, 8`
- (C) `push ebp`
- (D) `sub ebp, 8`

**答案：(B) `sub esp, 8`**
解析：區域變數靠減少 ESP 預留空間（2×4 = 8 bytes）；結束時用 `mov esp, ebp` 回收。

---

**A59.** Which is the correct way to pass an **array** to a procedure, and why?
- (A) By value, because it is safer
- (B) By reference (push its offset), because by value wastes stack space and slows the program
- (C) By copying each element into registers
- (D) Arrays cannot be passed to procedures

**答案：(B) 傳參考（push 陣列位址）**
解析：傳陣列一定要用 by reference；傳值會拖慢程式並浪費寶貴的堆疊空間。

---

**A60.** A local variable `char name[20]` in a stack frame reserves how many bytes (given each stack item rounds up to 4)?
- (A) 20 bytes
- (B) 21 bytes
- (C) 20 bytes (already a multiple of 4)
- (D) 24 bytes

**答案：(C) 20 bytes**
解析：每個堆疊項目無條件進位到 4 的倍數；20 已是 4 的倍數，故維持 20 bytes。（對比：char 1 byte 會進位到 4、30 bytes 會進位到 32。）

---

## Part B — Short Answer（簡答題）

### Chapter 1

**B1.** Define **Assembler**, **Linker**, and **Debugger** in one sentence each.

**答案：**
- **Assembler**：把組合語言程式轉成機器語言程式的工具。
- **Linker**：把一個或多個 object files 結合成單一可執行檔。
- **Debugger**：讓程式設計師追蹤（trace）程式執行並檢查（examine）記憶體內容的工具。

---

**B2.** Give the **8-bit two's complement** representation of −1, and verify it.

**答案：** +1 = 00000001b，反轉 → 11111110b，加 1 → **11111111b（FFh）**。
驗證：00000001 + 11111111 = (1)00000000 → 低 8 位為 0 ✅，故 11111111b 確為 −1。

---

**B3.** List the six EFLAGS **status flags** and the condition that triggers each.

**答案：**
- **CF（Carry）**：unsigned 運算超出範圍。
- **OF（Overflow）**：signed 運算超出範圍。
- **SF（Sign）**：結果為負（destination 最高 bit = 1）。
- **ZF（Zero）**：結果為 0。
- **AF（Auxiliary Carry）**：bit 3 → bit 4 進位。
- **PF（Parity）**：結果低位元組中 1 的個數為偶數。

---

### Chapter 2

**B4.** Explain the difference between `EQU 10*10` and `EQU <10*10>`.

**答案：** `EQU 10*10`（無角括號）會被 assembler **計算為整數 100**；`EQU <10*10>` 以角括號包住文字，**直接複製文字 "10*10"** 不做計算。故 `M1 WORD matrix1` → `M1 WORD 100`，而 `M2 WORD matrix2` → `M2 WORD 10*10`。

---

**B5.** What is the purpose of the **`$`** (current location counter), and how do you use it to compute the number of elements in a WORD array?

**答案：** `$` 回傳當前程式敘述的 offset（以 byte 為單位）。對 WORD 陣列，元素個數 = `($ - list) / 2`，因為 `$ - list` 是總 byte 數，而每個 WORD 佔 2 bytes。（BYTE 陣列不必除、DWORD 要除以 4。）此敘述必須緊接在陣列宣告之後。

---

**B6.** What does the **NOP** instruction do and why is it used?

**答案：** NOP 佔 1 byte、不做任何事。用途是把程式碼**對齊**到偶數 doubleword 邊界，因為 IA-32 從對齊位址載入程式碼/資料較快，可避免跨 cache line 邊界造成的 pipeline stall。

---

### Chapter 3

**B7.** State the **three rules** of the MOV instruction, and note its effect on flags.

**答案：**
1. 兩個 operand 大小必須相同。
2. 最多一個 memory operand（不可 memory-to-memory）。
3. EIP（與 IP）不可當 destination。
另外，**MOV 不影響任何 flag**。

---

**B8.** Explain the difference between **MOVZX** and **MOVSX**, and when to use each.

**答案：** 兩者都把較小的值複製到較大的暫存器。**MOVZX**（zero extension）高位補 0，用於 **unsigned** 整數；**MOVSX**（sign extension）高位補符號位元，用於 **signed** 整數。destination 必須是 register。對負數用 MOVZX 會破壞其值（如 −16=FFF0h → 65520）。

---

**B9.** For `myBytes BYTE 12h, 34h, 56h, 78h`, what is the value of `eax` after `mov eax, DWORD PTR myBytes`? Explain why.

**答案：** EAX = **78563412h**。因為 x86 是 Little Endian，記憶體由低到高為 12 34 56 78，組成 doubleword 時最高位址的 byte 是最高位 → 78 56 34 12 → 78563412h。

---

**B10.** Why must you **initialize a register** before using it as an indirect operand, and why is `PTR` sometimes required?

**答案：** 間接運算元 `[esi]` 會把暫存器內容當作位址 dereference；若 ESI 未初始化會指向無效位址造成 **general protection fault**。當組譯器無法判斷指標所指資料大小時（byte/word/dword），須用 **PTR** 指定，如 `inc WORD PTR [esi]`。

---

### Chapter 4

**B11.** Describe the two steps of **PUSH** and the two steps of **POP** (32-bit).

**答案：**
- **PUSH**：① ESP −= 4（向低位址成長）；② 把值寫入 [ESP]。
- **POP**：① 把 [ESP] 的內容讀到 register/variable；② ESP += 4。
（PUSH 先減後寫；POP 先讀後加。POP 沒有 immediate 版本。）

---

**B12.** In a chain of nested calls `main → Sub1 → Sub2 → Sub3`, describe what is on the stack when `Sub3` begins executing.

**答案：** 堆疊上有三個 return address（每進入一層 CALL push 一個）：由下到上為 (return to main)、(return to Sub1)、(return to Sub2)，ESP 指向最上層 (return to Sub2)。每執行一次 RET 就 pop 走一個，回到上一層。

---

**B13.** What three Irvine32 inputs does **DumpMem** require, and in which registers?

**答案：** DumpMem 需要：**ESI** = 起始位址（offset）、**ECX** = 單位個數（LENGTHOF）、**EBX** = 單位大小（TYPE）。它以 16 進位顯示該記憶體區塊。

---

### Chapter 5

**B14.** Compare **AND vs TEST** and **SUB vs CMP** in terms of whether they modify the destination and the flags.

**答案：**
| 指令 | 修改 destination？ | 影響 flags？ |
|------|:---:|:---:|
| AND | ✅ | ✅ |
| TEST | ❌ | ✅ |
| SUB | ✅ | ✅ |
| CMP | ❌ | ✅ |

TEST 是 nondestructive AND；CMP 是 nondestructive subtraction，都只設旗標不改 operand。

---

**B15.** Fill in the flags for `CMP a, b` where `a = 00001111b`, `b = 11110110b`, and give both the unsigned and signed interpretation.

**答案：** a − b = a + (−b 的二補數 00001010) = 00011001，過程：ZF=0、SF=0、CF=1、OF=0。
- **Unsigned**：a=15、b=246 → CF=1 表 **a < b**。
- **Signed**：a=15、b=−10 → SF=OF（皆 0）表 **a > b**。
重點：CPU 本身不分 signed/unsigned，只做二進位計算並設 flags，由軟體（所選的條件跳躍）決定解讀方式。

---

**B16.** Write the assembly for the following using a **short-circuit (reverse-condition)** AND, with unsigned comparisons:
```
if (al > bl) AND (bl > cl)
    X = 1;
```

**答案：**
```asm
    cmp al, bl
    jbe next        ; 第一條件 false（al 不大於 bl）→ 跳出
    cmp bl, cl
    jbe next        ; 第二條件 false → 跳出
    mov X, 1        ; 兩條件皆成立
next:
```
解析：AND 短路 → 任一條件 false 就用反向條件（JBE）跳出，較省程式碼。

---

**B17.** Translate this `while` loop into assembly (unsigned):
```
while (ebx <= val1) {
    ebx = ebx + 5;
    val1 = val1 - 1;
}
```

**答案：**
```asm
top:
    cmp ebx, val1
    ja  next        ; unsigned：ebx > val1（條件不成立）→ 離開
    add ebx, 5
    dec val1
    jmp top
next:
```
解析：WHILE 模板 = 開頭 cmp + 反向條件跳出、結尾 jmp 回頭。

---

### Chapter 6

**B18.** Complete the following, giving each AL value in hexadecimal:
```
mov al, 6Bh
shr al, 1      ; (a)
shl al, 3      ; (b)
mov al, 8Ch
sar al, 1      ; (c)
sar al, 3      ; (d)
```

**答案：**
- (a) 6Bh = 0110 1011，邏輯右移 1（補 0）→ 0011 0101 = **35h**
- (b) 35h = 0011 0101，邏輯左移 3（補 0）→ 1010 1000 = **A8h**
- (c) 8Ch = 1000 1100，算術右移 1（補符號 1）→ 1100 0110 = **C6h**
- (d) C6h = 1100 0110，算術右移 3（補符號 1）→ 1111 1000 = **F8h**

---

**B19.** State the multiply/divide shortcut for shifts, and which instruction to use for signed vs unsigned division.

**答案：** 左移 n 位 = 乘以 2ⁿ（SHL/SAL）；右移 n 位 = 除以 2ⁿ。除法時：**無號數用 SHR**（補 0），**有號數用 SAR**（補符號位元、保留正負號）。另外 SAL 與 SHL 完全相同。

---

**B20.** Why does `rol dl, 4` and `ror dl, 4` give the same result for `dl = 3Fh`?

**答案：** 因為對 8 位元的值旋轉 4 位剛好是寬度的一半，左旋 4 = 右旋 4。3Fh = 0011 1111 旋轉 4 位 → 1111 0011 = F3h（兩者皆同）。旋轉不會遺失位元，移出的位元繞回另一端。

---

### Chapter 7 (before LEA)

**B21.** What does a **stack frame** contain? List the items.

**答案：** Stack frame（activation record）包含：① 傳入的 arguments（引數）；② subroutine return address（回傳位址）；③ local variables（區域變數）；④ 任何暫時保存的 registers（暫存器）。

---

**B22.** List the six steps of creating a stack frame, in order.

**答案：**
1. 呼叫程式把 arguments push 到堆疊（若有）。
2. 執行 CALL → 把 return address push 到堆疊。
3. 被呼叫程序把 EBP push 到堆疊。
4. 設定 `EBP = ESP`（EBP 成為存取參數的基準點）。
5. 若有區域變數 → `sub esp, n` 預留空間。
6. 若有需保存的暫存器 → push 到堆疊。

---

**B23.** Explain the difference between **passing by value** and **passing by reference**, with the C++ analogy.

**答案：** **傳值（by value）**：把變數的副本推入堆疊，被呼叫程序改的是副本，不影響原值（如 `AddTwo(5, 6)` → push 5、push 6）。**傳址（by reference）**：把變數的位址推入堆疊，程序可透過位址修改原始變數（如 `Swap(&val1, &val2)` → push OFFSET val1、push OFFSET val2）。

---

**B24.** Compare the **C** and **STDCALL** calling conventions: who cleans the stack, how, and the trade-offs.

**答案：**
| | C | STDCALL |
|---|---|---|
| 誰清堆疊 | Caller（呼叫者） | Callee（被呼叫者） |
| 清法 | call 後 `add esp, n` | `ret n` |
| 用途 | C/C++ 函式 | Windows API |

- **STDCALL 優點**：每次呼叫少一道 `add esp, n` 指令，程式較小。
- **C 優點**：支援不定數目參數（如 printf），因為只有 caller 知道實際傳了幾個參數。

---

**B25.** Distinguish **global** vs **local** variables in terms of where they are declared, their lifetime, visibility, and where they are created.

**答案：**
| | Global（全域） | Local（區域） |
|---|---|---|
| 宣告位置 | data segment | 程序內部 |
| 生命週期 | 整個程式執行期間 | 單一程序內建立、使用、銷毀 |
| 可見性 | 整個原始檔所有程序 | 僅該程序 |
| 建立位置 | 資料區 | runtime stack（執行期堆疊） |

區域變數建立：`sub esp, 大小`（進位到 4 的倍數）；銷毀：`mov esp, ebp`。