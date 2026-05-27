# Chapter 7：Advanced Procedures（進階程序）

> 本章四大主題：
> 1. **Stack Frames（堆疊框架）** — 程序如何用堆疊傳參數、存區域變數、保存暫存器
> 2. **Recursion（遞迴）**
> 3. **INVOKE（呼叫指令）**
> 4. **Advanced Use of Parameters（參數進階用法：8/16/64 位元參數）**

---

## 📚 目錄
1. [Procedure Parameters（程序參數的兩種型態）](#一procedure-parameters程序參數的兩種型態)
2. [Stack Frame（堆疊框架）](#二stack-frame堆疊框架)
3. [Arguments vs Parameters（引數 vs 參數）](#三arguments-vs-parameters引數-vs-參數)
4. [Access to Stack Parameters（用 EBP 存取堆疊參數）](#四access-to-stack-parameters用-ebp-存取堆疊參數)
5. [Cleaning Up the Stack（清除堆疊參數）](#五cleaning-up-the-stack清除堆疊參數)
6. [Local Variables（區域變數）](#六local-variables區域變數)
7. [Saving and Restoring Registers（保存與還原暫存器）](#七saving-and-restoring-registers保存與還原暫存器)
8. [完整的 Stack Frame 範例](#八example完整的-stack-framea-complete-stack-frame)
9. [LEA Instruction（載入有效位址）](#九lea-instruction載入有效位址)
10. [Recursion（遞迴）](#十recursion遞迴)
11. [INVOKE Directive（INVOKE 指令）](#十一invoke-directiveinvoke-指令)
12. [Advanced Use of Parameters（參數進階用法）](#十二advanced-use-of-parameters參數進階用法)
13. [速背重點整理](#速背重點整理)

---

# 一、Procedure Parameters（程序參數的兩種型態）

| 型態 | 說明 | 特點 |
|------|------|------|
| **Register parameters（暫存器參數）** | 用暫存器（EAX/EBX/ECX…）傳參數 | 為了**最佳化執行速度**（但不一定總是更快！） |
| **Stack parameters（堆疊參數）** | 把參數 push 到堆疊上傳遞 | **幾乎所有高階語言都用這種** |

### 兩種呼叫方式對比（以 DumpMem 為例）

```asm
; 【暫存器參數】                  ; 【堆疊參數】
mov esi, OFFSET array            push OFFSET array
mov ecx, LENGTHOF array          push LENGTHOF array
mov ebx, TYPE array              push TYPE array
call DumpMem                     call DumpMem
```

### 圖解

```
【Register Parameter】              【Stack Parameter】
  mov eax,10                          push 10
  mov ebx,20                          push 20
  call sys_xyz                        call sys_xyz
       │                                   │
   CPU 暫存器直接帶值                  堆疊上放 10、20
   eax=10, ebx=20                     sys_xyz 從堆疊取值
```

## ⚠️ 暫存器參數的缺點（Disadvantages）

- 暫存器**經常被用來存放資料值**。
- 所以當暫存器要當參數用時，**呼叫前必須先 push 保存，呼叫後再 pop 還原** → 造成額外開銷（Overhead）！

```asm
push ebx              ; 先保存
push ecx
push esi
mov esi, OFFSET array
mov ecx, LENGTHOF array
mov ebx, TYPE array
call DumpMem
pop esi               ; 再還原
pop ecx
pop ebx
```

---

# 二、Stack Frame（堆疊框架）

> **Stack Frame = Activation Record（活化記錄）**
> - 呼叫程序時在堆疊上**建立**的區域。
> - 程序回傳時**釋放（de-allocated）**。

### 圖解：巢狀呼叫時的堆疊框架

```
void bar() {}
void foo() { bar(); }
int main() { foo(); }

Stack 變化：
  main → +foo → +bar → -bar → -foo → 結束
  ┌────┐  ┌────┐  ┌────┐
  │bar │ ← 一層 stack frame
  ├────┤  ├────┤  ├────┤
  │foo │  │foo │  │foo │
  ├────┤  ├────┤  ├────┤
  │main│  │main│  │main│
  └────┘  └────┘  └────┘
```

## Stack Frame 儲存什麼？

1. **Passed arguments（傳入的引數）**
2. **Subroutine return address（回傳位址）**
3. **Local variables（區域變數）**
4. **Any temporary saved registers（暫時保存的暫存器）**

## 🔑 Stack Frame 建立的六大步驟（必背）

```
1. 呼叫程式把【引數 arguments】push 到堆疊（若有）
2. 執行 CALL → 把【return address】push 到堆疊
3. 被呼叫的程序把【EBP】push 到堆疊
4. 設定 EBP = ESP
   → 從此 EBP 成為存取參數的【基準點 base reference】
5. 若有【區域變數】→ 減少 ESP 預留空間（sub esp, n）
6. 若有【需保存的暫存器】→ push 到堆疊
```

---

# 三、Arguments vs Parameters（引數 vs 參數）

| 名詞 | 定義 |
|------|------|
| **Arguments（引數）** | 呼叫程式**傳出**的值 |
| **Parameters（參數）** | 被呼叫程序**接收**的值 |

```c
static int func1(int x)    // x 是 parameter（參數）
{
    int t = 8;             // local variable
    return (x + t);
}
int main() {
    int b = 0;
    b = func1(a);          // a 是 argument（引數）
}
```

## Passing by Value vs Passing by Reference（傳值 vs 傳址）

| 型態 | 傳遞內容 | 對應 C++ |
|------|---------|----------|
| **Passing by value（傳值）** | 變數的**副本（copy）** | `AddTwo(5, 6)` |
| **Passing by reference（傳址）** | 變數的**位址（address）** | `Swap(&val1, &val2)` |

### 傳值範例
```asm
.code
   push 6           ; 第二個引數
   push 5           ; 第一個引數
   call AddTwo      ; EAX = sum
```
```
堆疊（高位址在上）：
   高位址  ┌──────┐
          │  6   │
          │  5   │
          │return│ ← ESP
   低位址  └──────┘
等同 C++： int sum = AddTwo(5, 6);
```

### 傳址範例
```asm
.data
val1 DWORD 5
val2 DWORD 6
.code
   push OFFSET val2
   push OFFSET val1
   call Swap        ; 等同 Swap(&val1, &val2);
```

## Passing Arrays（傳遞陣列）

- 傳陣列等資料結構**一定要用 passing by reference（傳址）**。
- 若用傳值 → **拖慢程式**、**浪費寶貴的堆疊空間**。

```asm
.data
count = 100
array DWORD count DUP(?)
.code
   push OFFSET array    ; 傳陣列位址
   push COUNT           ; 傳長度
   call ArrayFill
```

---

# 四、Access to Stack Parameters（用 EBP 存取堆疊參數）

- 程序用 **EBP 的偏移量** 來存取堆疊參數，例如 `[ebp + 8]`。
- EBP 又稱 **base pointer（基底指標）** 或 **frame pointer（框架指標）**。
  - 因為它存放**堆疊框架的基底位址**。
  - 函式開始時保存 EBP，結束時必須還原。

## 🌟 經典範例：AddTwo（存取堆疊參數）

```asm
.data
sum DWORD ?
.code
   push 6              ; 第二個引數
   push 5              ; 第一個引數
   call AddTwo         ; EAX = sum
   mov  sum, eax       ; 存結果

AddTwo PROC
   push ebp
   mov  ebp, esp        ; 設定堆疊框架基底
   mov  eax, [ebp + 12] ; 第二個引數 (6)
   add  eax, [ebp + 8]  ; 第一個引數 (5)
   pop  ebp
   ret
AddTwo ENDP
```

### 堆疊框架圖（進入 AddTwo 後）

```
   ┌──────────────┐
   │  00000006    │ [EBP + 12]   ← 第二引數
   │  00000005    │ [EBP + 8]    ← 第一引數
   │ return addr  │ [EBP + 4]    ← 回傳位址
   │     EBP      │ ← EBP, ESP   ← 舊的 EBP
   └──────────────┘
```

> 🔑 **偏移量記憶**：`[EBP+4]`=回傳位址，`[EBP+8]`=第一個引數，`[EBP+12]`=第二個引數…（每個 +4）

## 用符號常數讓程式更好讀

```asm
y_param EQU [ebp + 12]
x_param EQU [ebp + 8]

AddTwo PROC
   push ebp
   mov  ebp, esp
   mov  eax, y_param
   add  eax, x_param
   pop  ebp
   ret
AddTwo ENDP
```

---

# 五、Cleaning Up the Stack（清除堆疊參數）

- `RET` 指令只會把回傳位址 pop 到 EIP。
- **問題**：傳入的參數（如 5、6）仍留在堆疊上 → 必須清除！
- **誰來清？** → 由 **Calling Convention（呼叫慣例）** 決定。

## 兩種主要呼叫慣例

| | **C Calling Convention** | **STDCALL Calling Convention** |
|---|--------------------------|-------------------------------|
| 參數 push 順序 | 反向（先 push B 再 push A） | 反向（同 C） |
| 誰清堆疊 | **Caller（呼叫者）** | **Callee（被呼叫的程序自己）** |
| 清法 | `call` 之後加 `add esp, n` | `ret n`（給 RET 一個整數） |
| 用途 | 呼叫 C/C++ 寫的函式 | 呼叫 Windows API 函式 |

### C Calling Convention 範例（caller 清）
```asm
Example1 PROC
   push 6
   push 5
   call AddTwo
   add  esp, 8       ; ★ 呼叫者清除參數（2×4=8 bytes）
   ret
Example1 ENDP
```

### STDCALL Calling Convention 範例（callee 清）
```asm
AddTwo PROC
   push ebp
   mov  ebp, esp
   mov  eax, [ebp + 12]
   add  eax, [ebp + 8]
   pop  ebp
   ret  8            ; ★ 被呼叫者清除參數（給 RET 整數 8）
AddTwo ENDP
```

## 兩者優缺點比較

| 優點 | 說明 |
|------|------|
| **STDCALL 優點** | 少一道 `add esp,8` 指令 → 每次呼叫少一個指令，程式較小 |
| **C 優點** | 支援**不定數目參數**（如 `printf`），因為只有 caller 知道傳了幾個 |

> 💡 **為什麼 C 能支援不定參數？**
> 因為清堆疊是 caller 的責任，只有 caller 知道實際傳了幾個參數。
> STDCALL 由 callee 清，callee 必須事先知道參數數量 → 無法支援不定參數。

## 為什麼要定義 Calling Convention？

當你呼叫**別人寫的函式** `void xyz(int a, int b, int c)` 時，你必須知道：
- 如何傳參數：by stack 還是 by registers
- 傳遞順序：正向還是反向
- 誰負責清參數：caller 還是 callee
- 哪些暫存器必須被保存
- 函式如何回傳值

> **64 位元**：Microsoft x64 calling convention，由 **caller** 負責移除所有參數（細節略）。

---

# 六、Local Variables（區域變數）

| | Global（全域） | Local（區域） |
|---|---------------|---------------|
| 宣告位置 | data segment | 程序內部 |
| 屬性 | static global | — |
| 生命週期 (lifetime) | 整個程式執行期間 | 單一程序內建立、使用、銷毀 |
| 可見性 (visibility) | 整個原始檔的所有程序 | 僅該程序 |
| 建立位置 | 資料區 | **runtime stack（執行期堆疊）** |

> Note：這裡的 static ≠ 靜態變數，而是指「生命週期 = 程式執行期間」。

## 🌟 建立區域變數範例 1

```c
void MySub() {
    int X, Y;
    X = 10;
    Y = 20;
}
```
對應組語：
```asm
MySub PROC
   push ebp
   mov  ebp, esp
   sub  esp, 8                  ; ★ 建立區域變數（預留 8 bytes）
   mov  DWORD PTR [ebp-4], 10   ; X
   mov  DWORD PTR [ebp-8], 20   ; Y
   mov  esp, ebp                ; ★ 移除區域變數
   pop  ebp
   ret
MySub ENDP
```

### 圖解（區域變數在 EBP 下方）
```
   ┌──────────────┐
   │ Return addr  │
   │     EBP      │ ← EBP
   │   10 (X)     │ [EBP-4]
   │   20 (Y)     │ [EBP-8]
   └──────────────┘
```

> 🔑 **參數在 EBP 正方向（+），區域變數在 EBP 負方向（-）**

## 用符號常數
```asm
X_local EQU DWORD PTR [ebp-4]
Y_local EQU DWORD PTR [ebp-8]
...
   mov X_local, 10
   mov Y_local, 20
```

## 📌 區域變數總結
```
建立：sub esp, 總大小      （從 ESP 減去）
結束：mov esp, ebp         （把 ESP 設回 EBP，回收空間）
```

## 範例 2：堆疊對齊規則（重要考點！）

```c
void MySub() {
    char   X = 'X';      // 1 byte
    int    Y = 10;       // 4 bytes
    char   name[20];     // 20 bytes
    double Z = ...;      // 8 bytes
}
```

> ⚠️ **每個堆疊項目預設 4 bytes** → 每個變數大小**無條件進位到 4 的倍數**！

| 變數 | 實際 Bytes | 進位後偏移 |
|------|-----------|-----------|
| X | 4（char 1→4） | EBP-4 |
| Y | 4 | EBP-8 |
| name | 20 | **EBP-28** |
| Z | 8 | EBP-36 |

→ 共保留 **36 bytes**：`sub esp, 36`

---

# 七、Saving and Restoring Registers（保存與還原暫存器）

- 程序若會**修改某些暫存器的值**，應該先 **save**（push）再 **restore**（pop）。
- 否則呼叫者的暫存器值會被破壞！

### 圖解：沒保存 vs 有保存
```
【沒保存】呼叫 xyz 後 eax、ebx 被 xyz 改掉 → ❌ 資料遺失

【有保存】
xyz:
   push eax        ; 先保存
   push ebx
   mov eax, 0x59   ; 隨意使用
   mov ebx, 0x25
   pop ebx         ; 還原（反向順序！）
   pop eax
   → ✔ eax、ebx 維持原值
```

> 🔑 **push/pop 順序相反**：先 push eax 後 push ebx，還原時要先 pop ebx 後 pop eax。

---

# 八、Example：完整的 Stack Frame（A Complete Stack Frame）

整合「參數 + 區域變數 + 保存暫存器」的完整範例（AddTwo 有 2 參數、2 區域變數、修改 ECX/EDX）：

```asm
AddTwo PROC
   push ebp
   mov  ebp, esp
   sub  esp, 8          ; 建立 2 個區域變數
   push ecx             ; 保存 ECX
   push edx             ; 保存 EDX

   mov  ecx, [ebp+8]    ; 取參數 2
   add  edx, [ebp+12]   ; 取參數 1
   mov  DWORD PTR [ebp-4], 10   ; X
   mov  DWORD PTR [ebp-8], 20   ; Y

   pop  edx             ; 還原 EDX
   pop  ecx             ; 還原 ECX
   mov  esp, ebp        ; 移除區域變數
   pop  ebp             ; 還原 EBP
   ret  8               ; 清除參數
AddTwo ENDP
```

### 完整堆疊框架圖（由高位址到低位址）
```
   高位址 ┌──────────────┐
         │ Parameter 1  │ [EBP+12]   ┐
         │ Parameter 2  │ [EBP+8]    │
         │Return address│ [EBP+4]    │
         │     EBP      │ ← EBP      │ stack
         │   10 (X)     │ [EBP-4]    │ frame
         │   20 (Y)     │ [EBP-8]    │
         │  ECX (100)   │            │
         │  EDX (5)     │ ← ESP      ┘
   低位址 └──────────────┘
```

> 建立順序：push ebp → mov ebp,esp → sub esp（區域變數）→ push 暫存器
> 還原順序：pop 暫存器 → mov esp,ebp → pop ebp → ret n（完全相反）

## 簡單堆疊框架（Sub1→Sub2→Sub3 無參數無變數）
```
呼叫到 Sub3 時，堆疊有三個 stack frame，每個只存 return address：
   ┌─────────────┐
   │(ret to main)│ ← Sub1 的框架
   │(ret to Sub1)│ ← Sub2 的框架
   │(ret to Sub2)│ ← Sub3 的框架  ← ESP
   └─────────────┘
```

## Reserving Stack Space（預留堆疊空間）

- 程式模板用 `.stack 4096` 配置 4096 bytes 堆疊空間。
- 要預留**足夠**空間，特別是建立**陣列**作為區域變數時。
- **巢狀呼叫**時：堆疊必須大到能容納**任一時刻所有 stack frame 的總和**（含作用中的區域變數）。

### 程式模板
```asm
.386
.model flat, stdcall
.stack 4096
ExitProcess PROTO, dwExitCode:DWORD
.code
main PROC
   ; 你的程式碼
   INVOKE ExitProcess, 0
main ENDP
END main
```

## 複習：相關指令
- **PUSHFD / POPFD**：push/pop EFLAGS 暫存器
- **PUSHAD**：push 全部 32 位元通用暫存器，順序：EAX, ECX, EDX, EBX, ESP, EBP, ESI, EDI
- **POPAD**：以**反向順序**pop 回來

```asm
.data
saveFlags DWORD ?
.code
   pushfd               ; 把 flags push 到堆疊
   pop saveFlags        ; 複製到變數
   ...
   push saveFlags       ; push 回去
   popfd                ; 還原 flags
```

---

# 九、LEA Instruction（載入有效位址）

## 為什麼不能用 OFFSET？
- 問：如何取得 `[EBP+12]` 或 `[EBP-4]` 的偏移量（位址）？
- 用 OFFSET？**❌ 不行！**
  - OFFSET 只能回傳**常數偏移量**。
  - 但 EBP 的值**要到執行期（run-time）才能得知**。

## 解法：LEA（Load Effective Address）
- **LEA 可回傳任何間接運算元的偏移量**。
- 因為間接運算元可能用到暫存器，其偏移量只能在**執行期**計算。
- **取得堆疊參數和區域變數的位址，必須用 LEA**。

### LEA 範例
```c
void makeArray() {
    char myString[30];
    for(int i=0; i<30; i++)
        myString[i] = '*';
}
```
> 雖然陣列 30 bytes，但 ESP 減 **32**（每個堆疊項目 4 bytes，30→進位到 32）。

```asm
makeArray PROC
   push ebp
   mov  ebp, esp
   sub  esp, 32              ; myString 在 EBP-30
   lea  esi, [ebp-30]        ; ★ 載入有效位址（不能用 OFFSET）
   mov  ecx, 30              ; 迴圈計數
L1:
   mov  BYTE PTR [esi], '*'  ; 填一個位置
   inc  esi
   loop L1
   add  esp, 32              ; 或 mov esp, ebp
   pop  ebp
   ret
makeArray ENDP
```

> 🔑 **OFFSET vs LEA**：
> - OFFSET → 靜態常數位址（如全域變數），組譯期決定
> - LEA → 動態位址（堆疊參數、區域變數），執行期決定

---

# 十、Recursion（遞迴）

## 什麼是遞迴？
- 程序**呼叫自己**；或 A 呼叫 B，B 又呼叫 A。

### 無窮遞迴範例（會堆疊溢位）
```asm
Endless PROC
   mov  edx, OFFSET endlessStr
   call WriteString
   call Endless          ; ★ 呼叫自己 → 永不停止
   ret
Endless ENDP
```
> 每次呼叫自己用掉 **4 bytes**（回傳位址）→ 記憶體慢慢填滿 → **stack overflow → 例外**。

## 🌟 範例 1：遞迴計算總和 CalcSum（1+2+…+n）

```asm
; Receives: ECX = count (n)，Returns: EAX = sum
main PROC
   mov  ecx, 5          ; count = 5
   mov  eax, 0          ; 存放總和
   call CalcSum
L1:call WriteDec
   call Crlf
   exit
main ENDP

CalcSum PROC
   cmp  ecx, 0          ; 檢查計數
   jz   L2              ; =0 則結束
   add  eax, ecx        ; 否則加到總和
   dec  ecx             ; 計數減 1
   call CalcSum         ; ★ 遞迴呼叫
L2:ret
CalcSum ENDP
```

### 堆疊框架追蹤表
| Pushed On Stack | ECX | EAX |
|-----------------|-----|-----|
| L1 | 5 | 0 |
| L2 | 4 | 5 |
| L2 | 3 | 9 |
| L2 | 2 | 12 |
| L2 | 1 | 14 |
| L2 | 0 | **15** |

→ 最終 EAX = 15（= 5+4+3+2+1）

## 🌟 範例 2：遞迴計算階乘 Factorial（n!）

### C/C++ 版本
```c
int factorial(int n) {
    if(n == 0)
        return 1;            // base case（基底情況）
    else
        return n * factorial(n-1);
}
```

### 遞迴過程圖解（計算 5!）
```
遞迴呼叫（往下展開）         回溯相乘（往上回傳）
5! = 5 * 4!          →       5 * 24 = 120
4! = 4 * 3!          →       4 * 6  = 24
3! = 3 * 2!          →       3 * 2  = 6
2! = 2 * 1!          →       2 * 1  = 2
1! = 1 * 0!          →       1 * 1  = 1
0! = 1 (base case)   →       1
```

### 組語版本
```asm
main PROC
   push 12              ; 計算 12!
   call Factorial
ReturnMain:
   call WriteDec
   call Crlf
   exit
main ENDP

; Receives: [ebp+8] = n，Returns: eax = n!
Factorial PROC
   push ebp
   mov  ebp, esp
   mov  eax, [ebp+8]    ; 取 n
   cmp  eax, 0          ; n < 0?
   ja   L1              ; 是：繼續
   mov  eax, 1          ; 否（n=0）：回傳 1
   jmp  L2
L1:
   dec  eax
   push eax             ; Factorial(n-1)
   call Factorial       ; ★ 遞迴呼叫

; 以下指令在每次遞迴回傳後執行
ReturnFact:
   mov  ebx, [ebp+8]    ; 取 n
   mul  ebx             ; eax = eax * ebx
L2:
   pop  ebp
   ret  4               ; 清除堆疊
Factorial ENDP
```

### 堆疊框架圖（計算 12!）
```
   ┌────────────┐
   │     12     │ n
   │ ReturnMain │ ← 從 main 呼叫
   │   ebp_0    │
   ├────────────┤
   │     11     │ n-1
   │ ReturnFact │ ← 從 Factorial 呼叫
   │   ebp_1    │
   ├────────────┤
   │     10     │ n-2
   │ ReturnFact │
   │   ebp_2    │
   ├────────────┤
   │   (etc...) │
   └────────────┘
```
> 🔑 每次遞迴呼叫用掉 **12 bytes**（n 參數 4 + 回傳位址 4 + ebp 4）。
> 回傳位址：從 main 呼叫存 ReturnMain；從 Factorial 自呼叫存 ReturnFact。

---

# 十一、INVOKE Directive（INVOKE 指令）

## INVOKE = CALL 指令 + 自動 push 引數

```
語法：INVOKE procedureName, argumentList
```

### 範例 1
```asm
INVOKE ExitProcess, 0
```
組譯器自動展開為：
```asm
push 0
call ExitProcess
```

### 範例 2（多參數，反向 push）
```asm
INVOKE DumpArray, OFFSET array, LENGTHOF array, TYPE array
```
展開為：
```asm
push TYPE array          ; ★ 反向 push（最後一個先 push）
push LENGTHOF array
push OFFSET array
call DumpArray
```

> 🔑 INVOKE 自動以**反向順序**把引數 push 到堆疊，省去手動 push 的麻煩。

---

# 十二、Advanced Use of Parameters（參數進階用法）

## 1. 傳遞 8 位元與 16 位元引數

> ⚠️ **PUSH 不允許 8 位元運算元**（16 位元雖允許但仍建議擴展）。
> → push 前必須先**擴展到 32 位元**。

### 傳 8 位元（字元）→ 用 MOVZX
```asm
; ❌ 錯誤
.data
charVal BYTE 'x'
.code
   push charVal         ; syntax error！PUSH 不接受 8-bit

; ✔ 正確：用 MOVZX 擴展到 32 位元
.code
   movzx eax, charVal
   push  eax
   call  Uppercase
```

### Uppercase 程序範例
```asm
; 接收一個字元引數，回傳其大寫於 AL
Uppercase PROC
   push ebp
   mov  ebp, esp
   mov  al, [ebp+8]     ; AL = 字元
   cmp  al, 'a'         ; 小於 'a'？
   jb   L1              ; 是：不處理
   cmp  al, 'z'         ; 大於 'z'？
   ja   L1              ; 是：不處理
   sub  al, 20h         ; 否：轉成大寫（小寫 - 20h = 大寫）
L1:pop  ebp
   ret  4               ; 清除堆疊
Uppercase ENDP
```
> 💡 ASCII：小寫字母 - 20h = 對應大寫（如 'a'=61h, 'A'=41h，差 20h）。

### 傳 16 位元 → 用 MOVZX zero-extend
```asm
; ❌ 錯誤
.data
word1 WORD 1234h
word2 WORD 4111h
.code
   push word1
   push word2
   call AddTwo          ; error（AddTwo 預期 32-bit）

; ✔ 正確
   movzx eax, word1
   push  eax
   movzx eax, word2
   push  eax
   call  AddTwo
```

## 2. 傳遞 64 位元引數（多字組整數）

> 🔑 規則：**先 push 高位部分（high-order part first）**
> 因為整數在堆疊上要符合 **little-endian（小端序）**：低位元組放在最低位址。

### WriteHex64 範例
```asm
.data
longVal DQ 1234567800ABCDEFh
.code
   push DWORD PTR longVal + 4   ; 高位 doubleword (12345678)
   push DWORD PTR longVal       ; 低位 doubleword (00ABCDEF)
   call WriteHex64
```

### 堆疊圖（必須是 little-endian）
```
   ┌────────────┐
   │  12345678  │ [EBP+12]  ← 高位（先 push）
   │  00ABCDEF  │ [EBP+8]   ← 低位（後 push）
   │return addr │ [EBP+4]
   │    EBP     │ ← EBP, ESP
   └────────────┘
```

### WriteHex64 程序
```asm
WriteHex64 PROC
   push ebp
   mov  ebp, esp
   mov  eax, [ebp+12]   ; 高位 doubleword
   call WriteHex
   mov  eax, [ebp+8]    ; 低位 doubleword
   call WriteHex
   pop  ebp
   ret  8               ; 清除堆疊
WriteHex64 ENDP
```

---

# 🔥 速背重點整理

### 1. 兩種參數型態
- **暫存器參數**：快但有 push/pop overhead
- **堆疊參數**：高階語言通用

### 2. Stack Frame 六步驟（必背順序）
```
①push 引數 ②CALL push 回傳位址 ③push EBP
④mov ebp,esp ⑤sub esp(區域變數) ⑥push 保存暫存器
```

### 3. EBP 偏移量黃金法則
```
[EBP+4]  = 回傳位址
[EBP+8]  = 第一個參數    ┐ 參數在「正」方向 (+)
[EBP+12] = 第二個參數    ┘
[EBP-4]  = 第一個區域變數 ┐ 區域變數在「負」方向 (-)
[EBP-8]  = 第二個區域變數 ┘
```

### 4. 兩種 Calling Convention（最常考！）
| | C | STDCALL |
|---|---|---------|
| 誰清堆疊 | **Caller** | **Callee** |
| 清法 | `add esp, n` | `ret n` |
| 不定參數 | ✔ 支援 | ✘ 不支援 |
| 用途 | C/C++ 函式 | Windows API |

### 5. 區域變數
```
建立：sub esp, 大小（進位到 4 的倍數）
銷毀：mov esp, ebp
```

### 6. OFFSET vs LEA
- **OFFSET** → 常數位址（全域變數），組譯期
- **LEA** → 動態位址（堆疊參數/區域變數），執行期

### 7. 保存暫存器
- push/pop **順序相反**（先 push 的後 pop）

### 8. INVOKE
- = CALL + 自動**反向** push 引數

### 9. 參數大小擴展（push 前處理）
```
8-bit  → movzx eax, charVal → push eax （PUSH 不接受 8-bit）
16-bit → movzx eax, word    → push eax
64-bit → 先 push 高位，再 push 低位（little-endian）
```

### 10. 遞迴重點
- 必須有 **base case（基底情況）** 否則 stack overflow
- 每次呼叫都建立一個 stack frame，用掉堆疊空間
- CalcSum：1+…+5=15；Factorial 階乘每次呼叫用 12 bytes

---

✅ **本章一句話總結**：程序透過堆疊框架傳參數（[EBP+8]起）、存區域變數（[EBP-4]起）、保存暫存器；清堆疊由 Calling Convention 決定（C=caller / STDCALL=callee）；取動態位址用 LEA；遞迴靠堆疊框架層層堆疊，必須有終止條件。
