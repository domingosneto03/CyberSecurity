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

![image](screenshots/LB10_1.png)

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

![image](screenshots/LB10_2.png)

- Depois enviamos um request com o comando `download` e o resultado foi o seginte.

```
http://www.seedlab-hashlen.com/?myname=DomingosNeto&uid=1001&lstcmd=1&download=secret.txt&mac=dadcdd45b873b2f32ea675591ca045e23b81f1b1a2fa43f9562f6074b081f726
```

![image](screenshots/LB10_3.png)





