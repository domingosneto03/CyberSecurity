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



