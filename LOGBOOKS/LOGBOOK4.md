# Environment Variable and Set-UID Program Lab

## Task 1

- Através do comando `printenv` é possível listar todas as variáveis de ambiente do sistema.

- Experimentámos o comando `printenv PWD` para obter o path completo do diretório atual.

![image](screenshots/LB4_3.png)

- Com o comando `export` foi possível criar uma variável no sistema.

![image](screenshots/LB4_4.png)

## Task 2

- Após a compilação do ficheiro `myprintenv.c`, corremos e guardamos o seu output em `file` que contém uma lista completa das variáveis do processo filho.

- Após uma alteração no ficheiro `myprintenv.c` repetimos o processo guardando o output em `file2` que contém a lista de variáveis do processo pai.

- Tendo como objetivo descobrir se as variáveis de ambiente do processo pai são herdadas pelo processo filho, fez-se uma comparação entre os dois ficheiros com o seguinte comando:

```bash
diff file file2
```

- Um output vazio indica que não existem diferenças nos dois ficheiros concluindo então que as variáveis de ambiente são herdadas.

![image](screenshots/LB4_5.png)

## Task 3

- Executar o ficheiro `myenv.c` gera um output vazio.

![image](screenshots/LB4_6.png)

- Observando o comando `execve`, deduzimos que, por causa do terceiro parâmetro se encontrar a NULL, as variáveis de ambiente não são passadas.

```c
execve("/usr/bin/env", argv, NULL);
``` 

- Alterando esse parâmetro pela variável `environ`, o array passa a ser usado para passar as variáveis de ambiente.

```c
execve("/usr/bin/env", argv, environ);
``` 

![image](screenshots/LB4_7.png)

## Task 4

- Usando o comando `system`, verificámos que foi criado um novo processo onde são passadas todas variaveis de ambiente do processo anterior.

- No caso de `execve` é mantido o processo atual e as mesmas variáveis de ambiente, substituindo o processo em execução pelo comando passado como argumento, que so mantém as variáveis de ambiente se estas forem passadas como argumento.

## Task 5

- Apenas a variável `PATH` e a variável de ambiente personalizada entraram no processo filho Set-UID. Como o LD_LIBRARY_PATH é usado pelo carregador de linker dinâmico para localizar bibliotecas compartilhadas em tempo de execução, permitir mudanças em um programa Set-UID poderia representar um risco significativo de segurança. Um invasor poderia manipular essa variável para carregar bibliotecas maliciosas, o que poderia levar à execução de código arbitrário ou outras vulnerabilidades de segurança. 

- As variáveis `PATH` não são tão problemáticas porque afetam onde o shell procura por arquivos executáveis, mas não controlam diretamente o comportamento dos programas executados, assim como as variáveis personalizadas, que não têm um comportamento pré-definido associado ao sistema.

![image](screenshots/LB4_1.png)
*Antes de correr o programa Set-UID*

![image](screenshots/LB4_2.png)
*Depois de correr o programa Set-UID*


## Task 6

- Após criarmos e compilarmos o programa set-UID e atribuirmos root privileges, procedemos em criar o nosso código malicioso.

```c
#include <stdio.h>
#include <unistd.h>

int main() {
    printf("Malicious code!\n");
    if (geteuid() == 0) {
        printf("This code is running with root privilege\n");
    } else {
        printf("No root privilege for this code\n");
    }
    return 0;
}
```

- Compilamos o código num executável nomeado `ls`. Para garantir que o nosso código fosse executado em vez da chamada do sistema `/bin/ls`, modificámos a variável de amiente `PATH` para prioritizar o nosso diretório atual.

```bash
export PATH=/home/seed:$PATH
```

- Ao corrermos diretamente o comando `ls` é gerado o output do nosso código malicioso.

![image](screenshots/LB4_8.png)

- Através do comando seguinte foi possível ultrapassar o mecanismo de segurança para que os privilégios não caiam automaticamente.

```bash
sudo ln -sf /bin/zsh /bin/sh
```

## Task 8

**Step 1**


