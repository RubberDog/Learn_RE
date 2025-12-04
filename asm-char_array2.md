```
Dump of assembler code for function main:
```
```
   0x0000555555555149 <+0>:	push   rbp
   0x000055555555514a <+1>:	mov    rbp,rsp
   0x000055555555514d <+4>:	sub    rsp,0x20
```
Aufbau des Stack-Frame, reservierung von 32 Byte
```
   0x0000555555555151 <+8>:	mov    rax,QWORD PTR fs:0x28
```
Liest den Stack-Canary - fs:0x28 ist immer (Linux x86_64 / glibc) der Stack-Canary
```
   0x000055555555515a <+17>:	mov    QWORD PTR [rbp-0x8],rax
``` 
Speichert den Canary im lokalen Stackframe
```
   0x000055555555515e <+21>:	xor    eax,eax
```
eax komplett auf 0 setzen -> xor weil billig und schnell
```
   0x0000555555555160 <+23>:	lea    rax,[rbp-0x20]
```
Adresse von str_a berechnen
```
   0x0000555555555164 <+27>:	movabs rcx,0x77202c6f6c6c6548
```
Hex-Bytes -> big endian, im speicher als little endian;
`48 65 6c 6c f6 2x 20 77` -> `H e l l o , w`
```
   0x000055555555516e <+37>:	mov    QWORD PTR [rax],rcx
   0x0000555555555171 <+40>:	movabs rsi,0xa21646c726f77
```
`77 6f 72 6c 64 21 0a` -> `w o r l d ! \n'
```
   0x000055555555517b <+50>:	mov    QWORD PTR [rax+0x7],rsi
```
Hier noch wichtig; das doppelte 'w' stört nicht, da der zweite `mov` ab Byte7 schreibt - 'w' wird einfach durch 'w' ersetzt, alles passt wieder. 2x 8 byte-blöcke nutzen ist einfach nur schneller.

`movabs` und danach ein `mov` mit zu ascii passenden hexwerten -> höchstwahrscheinlich eine optimierte / inline-Version von strcpy
```
   0x000055555555517f <+54>:	lea    rax,[rbp-0x20]
   0x0000555555555183 <+58>:	mov    rdi,rax
   0x0000555555555186 <+61>:	mov    eax,0x0
   0x000055555555518b <+66>:	call   0x555555555040 <printf@plt>
```
`mov eax,$` direkt vor einem call -> höchstwahrscheinlich varargs

vorbereitung für printf(str_a)

bei variadischen funktionen ( -> funktionen mir einer variablen Anzahl an Argumenten)muss eax die anzahl der XMM-Register mit Float-Args enthalten; hier keine Floats, also `0`
```
   0x0000555555555190 <+71>:	mov    eax,0x0
```
`return 0`
```
   0x0000555555555195 <+76>:	mov    rdx,QWORD PTR [rbp-0x8]
   0x0000555555555199 <+80>:	sub    rdx,QWORD PTR fs:0x28
   0x00005555555551a2 <+89>:	je     0x5555555551a9 <main+96>
   0x00005555555551a4 <+91>:	call   0x555555555030 <__stack_chk_fail@plt>
```
Canary-Check. `mov` lädt den Anfangs gespeicherten Canary in rdx, `sub` zieht den aktuellen Canary-Wert ab. Wenn nichts überschrieben wurde `== 0`, alles okay.

Andernfalls; ein Overflow hat den Canary zerstört, Ergebnis `!= 0`, `je` erlaubt den Sprung nicht, Programmabbruch durch stack_chk_fail

Muster zum merken: Canary-Check mit stack_chk_fail am Ende -> SSP ("Stack Smashing Protector") aktiv
```
   0x00005555555551a9 <+96>:	leave
   0x00005555555551aa <+97>:	ret
```
Stackframe abbauen, Programm beendet
```
End of assembler dump.
```
