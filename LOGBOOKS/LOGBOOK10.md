# Hash Length Extension Attack Lab

## Setup

- Construimos o container. 

```bash
$ dcbuild
$ dcup
```

- Certificamo-nos que o hostname está corretamente mapeado para o servidor.

```bash
$ sudo nano /etc/hosts
```

![image](/screenshots/LB10_1.png)

- Obtemos a shell do container.

```bash
$ dockps
$ docksh 39
```

## Task 1

- Com o o bjetivo de enviar requests ao servidor, obtemos os argumentos necessários.

- Na shell corremos os seguintes comandos:

```bash
# cat LabHome/key.txt
1001:123456
1002:983abe
1003:793zye
1004:88zjxc
1005:xciujk
```

- Escolhemos o `uid`: 1001 e a `key` correspondente: 123456.

- O nome a ser utilizado: DomingosNeto

- Desta forma, calculamos o MAC

```bash
$ echo -n "123456:myname=DomingosNeto&uid=1001&lstcmd=1" | sha256sum

dadcdd45b873b2f32ea675591ca045e23b81f1b1a2fa43f9562f6074b081f726  -
```

- Tendo o MAC calculado, foi possível contruir o request completo.

```
http://www.seedlab-hashlen.com/?myname=DomingosNeto&uid=1001&lstcmd=1&mac=dadcdd45b873b2f32ea675591ca045e23b81f1b1a2fa43f9562f6074b081f726
```

- Enviamos o request e obtivemos a seguinte resposta do servidor:

![image](/screenshots/LB10_2.png)

- Depois enviamos um request com o comando `download` e o resultado foi o seginte.

```
http://www.seedlab-hashlen.com/?myname=DomingosNeto&uid=1001&lstcmd=1&download=secret.txt&mac=dadcdd45b873b2f32ea675591ca045e23b81f1b1a2fa43f9562f6074b081f726
```

![image](/screenshots/LB10_3.png)

## Task 2

- Considerando a mensagem:

```
"123456:myname=DomingosNeto&uid=1001&lstcmd=1"
```

- O tamanho da mensagem é de 44 bytes. Portanto, o padding é `64-44=20`.

- 20 bytes são 44 bytes são `8*44=352` bits sendo igual a `0x0160`

```
"123456:myname=DomingosNeto&uid=1001&lstcmd=1"
"\x80"
"\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00"
"\x00\x00\x00\x00\x00\x00\x00\x01\x60"
```

- Em formato compatível com URL temos:

```
"%80%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%01%60"
```

## Task 3

Começamos a task por instalar as livrarias necessárias com o comando ```sudo apt install libssl-dev```.

Usando os paddying bytes obtidos na task anterior seguidos por "&download=secret.txt", substituímos o segundo campo da função ***SHA256_Update***.

```c
/* length_ext.c */
#include <stdio.h>
#include <arpa/inet.h>
#include <openssl/sha.h>

int main(int argc, const char *argv[])
{
    int i;
    unsigned char buffer[SHA256_DIGEST_LENGTH];
    SHA256_CTX c;
    SHA256_Init(&c);

    // Append additional message
    SHA256_Update(&c,
        "123456:myname=DomingosNeto&uid=1001&lstcmd=1"
        "\x80"
        "\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00"
        "\x00\x00\x00\x00\x00\x00\x00\x01\x60"
        "&download=secret.txt",
        64 + 20);

    SHA256_Final(buffer, &c);

    for (i = 0; i < 32; i++) {
        printf("%02x", buffer[i]);
    }
    printf("\n");

    return 0;
}
```
Compilando o código com ```gcc length_ext.c -o length_ext -lcrypto``` e o executando retorna :

![MAC](/screenshots/LB10_4.png)

Podemos assim construir o url :

```
http://www.seedlab-hashlen.com/?myname=DomingosNeto&uid=1001&lstcmd=1
%80%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%01%60
&download=secret.txt&mac=
b361c51dab6526590c77edeb6a6a387d938bb10c6e2df64f4e2bdc99fa4a74ae
```

![Task3 Result](/screenshots/LB10_5.png)
