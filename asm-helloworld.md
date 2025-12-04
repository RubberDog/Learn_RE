```
(gdb) disass main
Dump of assembler code for function main:
   0x0000555555555139 <+0>:	push   rbp      
   0x000055555555513a <+1>:	mov    rbp,rsp
```
Hier wird der Stack-Frame aufgebaut
```
   0x000055555555513d <+4>:	sub    rsp,0x10
```
Speicherreservierung (16 byte) für lokale Variablen
```
   0x0000555555555141 <+8>:	mov    DWORD PTR [rbp-0x4],0x0
```
int i liegt bei rbp - 4 byte auf dem Stack, wird auf 0 gesetzt
```
   0x0000555555555148 <+15>:	jmp    0x55555555515d <main+36>
```
Sprung zur loop-Bedingung
```
   0x000055555555514a <+17>:	lea    rax,[rip+0xeb3]        
   0x0000555555555151 <+24>:	mov    rdi,rax
   0x0000555555555154 <+27>:	call   0x555555555030 <puts@plt>
```
loop-Body; 
- `lea` berechnet die Adresse des String 
- `mov rdi,rax` erster Funktionsparameter `(rdi = char *) 
- `call` - offensichtlich, oder? `puts` wird aufgerufen
```
   0x0000555555555159 <+32>:	add    DWORD PTR [rbp-0x4],0x1
```
`i++`, der counter wird inkrementiert
```
   0x000055555555515d <+36>:	cmp    DWORD PTR [rbp-0x4],0x9
   0x0000555555555161 <+40>:	jle    0x55555555514a <main+17>
```
loop-Bedingung, Zeile cmp == `i < 10`. jle spring an die Startadresse des loop-Body
```
   0x0000555555555163 <+42>:	mov    eax,0x0
```
`return 0`
```
   0x0000555555555168 <+47>:	leave
   0x0000555555555169 <+48>:	ret
```
Hier wird nur noch der Stack-Frame abgebaut, `ret` = return zum caller
```
End of assembler dump.
```
