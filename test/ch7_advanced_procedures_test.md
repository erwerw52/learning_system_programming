# CH7 進階程序 — 練習題

> 主題：Stack Frame · 遞迴 · INVOKE · 參數 ｜ 共 18 題（原題 10・補充 8）

---

### Q01 `原題`

下列程式用的是哪種 calling convention？說明理由。

```asm
push 10
push 20
call AddTwo
mov sum, eax
...
AddTwo PROC
  push ebp
  mov  ebp, esp
  mov  eax, [ebp + 8]
  sub  eax, [ebp + 12]
  pop  ebp
  ret  8
AddTwo ENDP
```

**答案：** **STDCALL**。因為由被呼叫者用 `ret 8` 清除參數，呼叫端不需要像 C calling 那樣在 call 之後加 `add esp, n`。

---

### Q02 `原題`

STDCALL 與 C calling convention 在「清除堆疊」上的主要差異？

**答案：** C calling 由**呼叫者（caller）**在 call 之後 `add esp, n`；STDCALL 由**被呼叫者（callee）**用 `ret n` 清除參數。

---

### Q03 `原題`

```c
void makeArray(){
  char myString[30];
  for(int i=0;i<30;i++) myString[i]='*';
}
```

myString 在堆疊上實際佔幾 bytes？

**答案：** **32 bytes**（每個堆疊項目進位到 4 的倍數，30 → 32）。

---

### Q04 `原題`

```asm
push OFFSET val2
push OFFSET val1
call Swap
```

這是 passing by value 還是 by reference？為什麼？

**答案：** **by reference**。push OFFSET 傳的是變數的位址，不是值。

---

### Q05 `原題`

Passing by Value 與 by Reference 的差異？傳遞大型結構（如陣列）為何建議用 by Reference？

**答案：** by Value 把變數副本 push 到 stack；by Reference 傳記憶體位址。大型結構若用 by Value 會把整個結構複製到 stack，**降速且消耗大量 stack 空間**。

---

### Q06 `原題`

為什麼取堆疊參數或區域變數的位址要用 LEA，而不是 OFFSET？

**答案：** 堆疊變數的位址要到**執行期（run-time）**才能確定，而 OFFSET 只能在編譯期計算常數偏移量。

---

### Q07 `原題`

為什麼 recursion 可能造成 stack overflow？

**答案：** 每次遞迴呼叫都建立新的 stack frame，若沒有終止條件（base case）會無限增加，耗盡堆疊空間。

---

### Q08 `原題`

建立 Stack Frame 的前四個關鍵步驟？

**答案：** 1. 把參數 push 到 stack；2. CALL 自動把 return address push 到 stack；3. 被呼叫程序保存舊的 EBP（push ebp）；4. 設定 EBP = ESP。

---

### Q09 `原題`

```asm
main PROC
  mov ecx, 5
  mov eax, 0
  call CalcSum
  call WriteDec
  ...
CalcSum PROC
  cmp ecx, 0
  jz  L2
  add eax, ecx
  dec ecx
  call CalcSum
L2: ret
CalcSum ENDP
```

最後輸出為何？

**答案：** **15**（= 5+4+3+2+1）。

---

### Q10 `原題`

```asm
push 6
push 5
call AddTwo
...
AddTwo PROC
  push ebp
  mov  ebp, esp
  mov  eax, [ebp + 12]
  add  eax, [ebp + 8]
  pop  ebp
  ret
AddTwo ENDP
```

執行 pop ebp 前 EAX=？ESP 指向哪？pop ebp 後 ESP 指向哪？

**答案：** pop ebp 前 EAX = **11**；ESP 指向**舊的 EBP**；pop ebp 後 ESP 指向 **return address**。

---

### Q11 `補充`

`INVOKE DumpArray, OFFSET array, LENGTHOF array, TYPE array` 會展開成什麼？

**答案：**

```asm
push TYPE array        ; 反向 push（最後一個先 push）
push LENGTHOF array
push OFFSET array
call DumpArray
```

---

### Q12 `補充`

為什麼不能直接 `push charVal`（BYTE）？該怎麼傳 8 位元引數？

**答案：** PUSH 不接受 8-bit operand。要先擴展到 32 位元：

```asm
movzx eax, charVal
push  eax
```

---

### Q13 `補充`

傳遞 64 位元引數時，要先 push 高位還是低位部分？為什麼？

**答案：** 先 push **高位 doubleword**，再 push 低位。這樣在堆疊上才符合 **little-endian**（低位元組在最低位址）。

---

### Q14 `補充`

區域變數 `char X; int Y; char name[20]; double Z;` 共需 `sub esp,` 多少？

**答案：** 各進位到 4 的倍數：X=4、Y=4、name=20、Z=8 → 共 **36 bytes**，`sub esp, 36`。

---

### Q15 `補充`

Arguments 與 Parameters 的定義差別？

**答案：** **Arguments（引數）**是呼叫端傳出的值；**Parameters（參數）**是被呼叫程序接收的值。

---

### Q16 `補充`

寫出 EBP 偏移量的黃金法則（參數與區域變數的方向）。

**答案：** `[EBP+4]`=return address、`[EBP+8]`=第一個參數、`[EBP+12]`=第二個參數（參數在 **+** 方向）；`[EBP-4]`=第一個區域變數、`[EBP-8]`=第二個（區域變數在 **−** 方向）。

---

### Q17 `補充`

區域變數的「建立」與「銷毀」分別用什麼指令？

**答案：** 建立：`sub esp, 總大小`（進位到 4 的倍數）；銷毀：`mov esp, ebp`（把 ESP 設回 EBP 回收空間）。

---

### Q18 `補充`

Register 參數與 Stack 參數各自的優缺點？

**答案：** **Register 參數**快，但暫存器常被佔用，呼叫前後需 push/pop 保存還原（overhead）；**Stack 參數**是高階語言通用做法，較有彈性。
