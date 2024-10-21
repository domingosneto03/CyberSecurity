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
