Noch nichts überprüft weil Ahnungslos..

# x86-64 Register & ABI Cheat Sheet (Linux / System V ABI)

## 1. Überblick über die General Purpose Registers (GPR)

x86-64 hat 16 allgemeine Register:

```
rax, rbx, rcx, rdx,
rsi, rdi,
rbp, rsp,
r8, r9, r10, r11,
r12, r13, r14, r15
```

Jedes Register hat Unterregister:

```
rax (64)
├─ eax (32)
│  └─ ax (16)
│     ├─ ah (8, high)
│     └─ al (8, low)
```

Bei r8–r15:

* r8d (32), r8w (16), r8b (8)
* keine ah/bh/etc.

---

## 2. Registerrollen nach Linux System V AMD64 ABI

### 2.1 Funktionsargumente (in dieser Reihenfolge)

1. `rdi` – 1. Argument
2. `rsi` – 2. Argument
3. `rdx` – 3. Argument
4. `rcx` – 4. Argument
5. `r8` – 5. Argument
6. `r9` – 6. Argument

Weitere Argumente → Stack.

### 2.2 Rückgabewert

* `rax` – Rückgabewert für Integer, Pointer, kleine Strukturen.
* Größere Structs → hidden pointer.

### 2.3 Caller-saved Register

Funktion darf sie zerstören, Caller muss sichern:

* `rax`, `rcx`, `rdx`, `rsi`, `rdi`, `r8`, `r9`, `r10`, `r11`

### 2.4 Callee-saved Register

Funktion muss sichern, wenn sie sie nutzt:

* `rbx`, `rbp`, `r12`, `r13`, `r14`, `r15`

`spp` ist Spezialfall (Stack Pointer). Nie als normales Register genutzt.

---

## 3. Die wichtigsten Register im Detail

### `rax` – Accumulator / Return Value

* Speichert Rückgabewerte.
* Häufig temporär verwendet.
* `xor eax, eax` = schnellste Art, rax auf 0 zu setzen.
* Bei variadischen Funktionen (printf): `eax = Anzahl der Floating-Point-Args` → oft 0.

### `rbx` – Callee-saved, oft Basisregister

* Muss gesichert werden (`push rbx`).
* Oft genutzt für konstante Pointer, Kontext, Schleifenvariablen.

### `rcx` – 4. Argument / Counter

* Traditionelles Counter-Register.
* Wird von `rep movsb` etc. genutzt.

### `rdx` – 3. Argument / high-part von Mul/Div

* Wird für Multiplikation, Division gebraucht.

### `rsi` / `rdi` – Parameterregister

* `rdi` = erstes Argument (oft Zielpointer)
* `rsi` = zweites Argument (oft Quellenpointer)

Sehr typisch bei `memcpy`, `strcpy`, `read`, `write` usw.

### `rbp` – Base Pointer

* Zeigt auf den aktuellen Stackframe.
* Klassischer Prolog:

```
push rbp
mov rbp, rsp
sub rsp, <size>
```

* Lokale Variablen → `[rbp - offset]`

### `rsp` – Stack Pointer

* Zeigt auf die aktuelle Stackspitze.
* Wird durch push/pop und Prolog/Epilog verändert.

### `r8–r15` – zusätzliche Register

* Werden oft für weitere Argumente oder temporäre Werte genutzt.
* In optimiertem Code intensiv verwendet.

---

## 4. Typische Compiler-Muster

### 4.1 Funktionsprolog mit Canary

```
push rbp
mov  rbp, rsp
sub  rsp, 0x30

mov  rax, QWORD PTR fs:0x28       ; Canary laden
mov  QWORD PTR [rbp-0x8], rax
xor  eax, eax
```

→ Stack Smashing Protector aktiv.

### 4.2 Variadischer Funktionsaufruf (printf)

```
mov rdi, <fmt>
mov eax, 0        ; Anzahl Floating-Point-Args
call printf@plt
```

### 4.3 Rückgabewert

```
mov eax, <konstant>
ret
```

oder

```
xor eax, eax
ret
```

### 4.4 Inline-String-Copy

```
lea    rax, [rbp-0x20]
movabs rcx, 0x77202c6f6c6c6548
mov    [rax], rcx
movabs rsi, 0xa21646c726f77
mov    [rax+0x7], rsi
```

→ Compiler ersetzt strcpy durch direkten String-Write.

---

## 5. Stack Canary Erkennung

Typisches Muster (SSP aktiv):

```
mov rax, fs:0x28            ; Canary laden
mov [rbp-0x8], rax
...
mov rdx, [rbp-0x8]
sub rdx, fs:0x28
je  ok
call __stack_chk_fail@plt
```

→ Canary schützt nur die Return-Adresse, nicht andere lokale Variablen.

---

## 6. Unterschied zu Windows x64 ABI (Kurzfassung)

### Argumentreihenfolge

Windows:

1. `rcx`
2. `rdx`
3. `r8`
4. `r9`

Linux:

1. `rdi`
2. `rsi`
3. `rdx`
4. `rcx`
5. `r8`
6. `r9`

### Variadische Funktionen

* Linux benötigt `eax = #float args`
* Windows nicht.

### Callee-saved

Windows speichert zusätzlich `rdi`, `rsi`.

---

## 7. Mini-Merkzettel

```
Linux SysV ABI:

Args : rdi, rsi, rdx, rcx, r8, r9
Ret  : rax
Caller-saved : rax, rcx, rdx, rsi, rdi, r8, r9, r10, r11
Callee-saved : rbx, rbp, r12–r15

rbp/rsp → Stackframe
fs:0x28 → Stack Canary

xor eax, eax → rax = 0 (Returnwert oder varargs)
lea [rbp - off] → Adresse einer lokalen Variable
call <foo>@plt → Funktionsaufruf aus libc
```

---

## 8. Nützliche RE-Faustregeln

* Wenn `rdi` vor Call gesetzt → erster Funktionsparameter.
* Wenn `eax` vor printf = 0 → variadischer Aufruf.
* Wenn `lea reg, [rbp-offset]` → lokaler Buffer.
* Wenn `fs:0x28` im Prolog → SSP aktiv.
* Wenn `mov eax, <wert>` vor ret → Rückgabewert.
* Wenn `push rbx/r12...` am Anfang → callee-saved wird genutzt.

---

Wenn du möchtest, erweitere ich dir das Cheat Sheet noch um:

* Calling Conventions (fastcall, stdcall, Windows x64)
* typische Optimierungs-Muster (O0 vs O2)
* ROP-relevante Patterns
* Frame-Pointer-Omission (FPO)-Erkennung
