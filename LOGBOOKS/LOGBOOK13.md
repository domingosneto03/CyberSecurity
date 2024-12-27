# Packet Sniffing and Spoofing Lab

## Setup

- Construimos o container. 

```bash
$ dcbuild
$ dcup
```
- A VM será o `atacante` e os restantes containers serão `HostA` e `HostB`

- Obtemos a shell dos containers container.

```bash
$ dockps
$ docksh 0e # hostA
$ docksh bf # hostB
```

## Task 1.1.A

- Encontramos o nome da interface correspondente utilizando o seguinte comando:

```bash
$ docker network ls
```

![image](/screenshots/LB13_1.png)

- O nome da interface que pretendemos é `br-5b39555aa03d`

- Criamos o ficheiro `sniffer.py`

```py
#!/usr/bin/env python3
from scapy.all import *
def print_pkt(pkt):
    pkt.show()
pkt = sniff(iface='br-5b39555aa03d', filter='icmp', prn=print_pkt)
```

- Executamos o ficheiro com privilégios root

```bash
$ chmod a+x sniffer.py
$ sudo ./sniffer.py
```

- A nossa host atacante está pronta para receber capturas de pacotes transmitidos.

- No HostA enviamos pacotes ICMP para hostB e vice versa. Observou-se as respetivas capturas na VM

```bash
root@0e66e2208714:/# ping 10.9.0.6 # do hostA para hostB
root@bf0a6620e04a:/# ping 10.9.0.5 # do hostB para hostA
```

![image](/screenshots/LB13_2.png)

```bash
###[ Ethernet ]### 
  dst       = 02:42:0a:09:00:06
  src       = 02:42:0a:09:00:05
  type      = IPv4
###[ IP ]### 
     version   = 4
     ihl       = 5
     tos       = 0x0
     len       = 84
     id        = 46864
     flags     = DF
     frag      = 0
     ttl       = 64
     proto     = icmp
     chksum    = 0x6f7c
     src       = 10.9.0.5
     dst       = 10.9.0.6
     \options   \
###[ ICMP ]### 
        type      = echo-request
        code      = 0
        chksum    = 0x9ad0
        id        = 0x26
        seq       = 0x1
###[ Raw ]### 
           load      = '\xcf\xd0ng\x00\x00\x00\x00Z\xfd\x05\x00\x00\x00\x00\x00\x10\x11\x12\x13\x14\x15\x16\x17\x18\x19\x1a\x1b\x1c\x1d\x1e\x1f !"#$%&\'()*+,-./01234567'

###[ Ethernet ]### 
  dst       = 02:42:0a:09:00:05
  src       = 02:42:0a:09:00:06
  type      = IPv4
###[ IP ]### 
     version   = 4
     ihl       = 5
     tos       = 0x0
     len       = 84
     id        = 3415
     flags     = 
     frag      = 0
     ttl       = 64
     proto     = icmp
     chksum    = 0x5936
     src       = 10.9.0.6
     dst       = 10.9.0.5
     \options   \
###[ ICMP ]### 
        type      = echo-reply
        code      = 0
        chksum    = 0xa2d0
        id        = 0x26
        seq       = 0x1
###[ Raw ]### 
           load      = '\xcf\xd0ng\x00\x00\x00\x00Z\xfd\x05\x00\x00\x00\x00\x00\x10\x11\x12\x13\x14\x15\x16\x17\x18\x19\x1a\x1b\x1c\x1d\x1e\x1f !"#$%&\'()*+,-./01234567'

...

###[ Ethernet ]### 
  dst       = 02:42:0a:09:00:06
  src       = 02:42:0a:09:00:05
  type      = IPv4
###[ IP ]### 
     version   = 4
     ihl       = 5
     tos       = 0x0
     len       = 84
     id        = 47328
     flags     = DF
     frag      = 0
     ttl       = 64
     proto     = icmp
     chksum    = 0x6dac
     src       = 10.9.0.5
     dst       = 10.9.0.6
     \options   \
###[ ICMP ]### 
        type      = echo-request
        code      = 0
        chksum    = 0xef2e
        id        = 0x26
        seq       = 0x5
###[ Raw ]### 
           load      = '\xd3\xd0ng\x00\x00\x00\x00\x00\x9b\x07\x00\x00\x00\x00\x00\x10\x11\x12\x13\x14\x15\x16\x17\x18\x19\x1a\x1b\x1c\x1d\x1e\x1f !"#$%&\'()*+,-./01234567'

###[ Ethernet ]### 
  dst       = 02:42:0a:09:00:05
  src       = 02:42:0a:09:00:06
  type      = IPv4
###[ IP ]### 
     version   = 4
     ihl       = 5
     tos       = 0x0
     len       = 84
     id        = 3658
     flags     = 
     frag      = 0
     ttl       = 64
     proto     = icmp
     chksum    = 0x5843
     src       = 10.9.0.6
     dst       = 10.9.0.5
     \options   \
###[ ICMP ]### 
        type      = echo-reply
        code      = 0
        chksum    = 0xf72e
        id        = 0x26
        seq       = 0x5
###[ Raw ]### 
           load      = '\xd3\xd0ng\x00\x00\x00\x00\x00\x9b\x07\x00\x00\x00\x00\x00\x10\x11\x12\x13\x14\x15\x16\x17\x18\x19\x1a\x1b\x1c\x1d\x1e\x1f !"#$%&\'()*+,-./01234567'
```

![image](/screenshots/LB13_3.png)

```bash
###[ Ethernet ]### 
  dst       = 02:42:0a:09:00:05
  src       = 02:42:0a:09:00:06
  type      = IPv4
###[ IP ]### 
     version   = 4
     ihl       = 5
     tos       = 0x0
     len       = 84
     id        = 3658
     flags     = 
     frag      = 0
     ttl       = 64
     proto     = icmp
     chksum    = 0x5843
     src       = 10.9.0.6
     dst       = 10.9.0.5
     \options   \
###[ ICMP ]### 
        type      = echo-reply
        code      = 0
        chksum    = 0xf72e
        id        = 0x26
        seq       = 0x5
###[ Raw ]### 
           load      = '\xd3\xd0ng\x00\x00\x00\x00\x00\x9b\x07\x00\x00\x00\x00\x00\x10\x11\x12\x13\x14\x15\x16\x17\x18\x19\x1a\x1b\x1c\x1d\x1e\x1f !"#$%&\'()*+,-./01234567'

###[ Ethernet ]### 
  dst       = 02:42:0a:09:00:05
  src       = 02:42:0a:09:00:06
  type      = IPv4
###[ IP ]### 
     version   = 4
     ihl       = 5
     tos       = 0x0
     len       = 84
     id        = 4238
     flags     = DF
     frag      = 0
     ttl       = 64
     proto     = icmp
     chksum    = 0x15ff
     src       = 10.9.0.6
     dst       = 10.9.0.5
     \options   \
###[ ICMP ]### 
        type      = echo-request
        code      = 0
        chksum    = 0x60bb
        id        = 0x22
        seq       = 0x1
###[ Raw ]### 
           load      = 'a\xd1ng\x00\x00\x00\x00\x06\x16\x02\x00\x00\x00\x00\x00\x10\x11\x12\x13\x14\x15\x16\x17\x18\x19\x1a\x1b\x1c\x1d\x1e\x1f !"#$%&\'()*+,-./01234567'

...

###[ Ethernet ]### 
  dst       = 02:42:0a:09:00:05
  src       = 02:42:0a:09:00:06
  type      = IPv4
###[ IP ]### 
     version   = 4
     ihl       = 5
     tos       = 0x0
     len       = 84
     id        = 4944
     flags     = DF
     frag      = 0
     ttl       = 64
     proto     = icmp
     chksum    = 0x133d
     src       = 10.9.0.6
     dst       = 10.9.0.5
     \options   \
###[ ICMP ]### 
        type      = echo-request
        code      = 0
        chksum    = 0x435f
        id        = 0x22
        seq       = 0x5
###[ Raw ]### 
           load      = 'e\xd1ng\x00\x00\x00\x00\x1en\x03\x00\x00\x00\x00\x00\x10\x11\x12\x13\x14\x15\x16\x17\x18\x19\x1a\x1b\x1c\x1d\x1e\x1f !"#$%&\'()*+,-./01234567'

###[ Ethernet ]### 
  dst       = 02:42:0a:09:00:06
  src       = 02:42:0a:09:00:05
  type      = IPv4
###[ IP ]### 
     version   = 4
     ihl       = 5
     tos       = 0x0
     len       = 84
     id        = 56872
     flags     = 
     frag      = 0
     ttl       = 64
     proto     = icmp
     chksum    = 0x8864
     src       = 10.9.0.5
     dst       = 10.9.0.6
     \options   \
###[ ICMP ]### 
        type      = echo-reply
        code      = 0
        chksum    = 0x4b5f
        id        = 0x22
        seq       = 0x5
###[ Raw ]### 
           load      = 'e\xd1ng\x00\x00\x00\x00\x1en\x03\x00\x00\x00\x00\x00\x10\x11\x12\x13\x14\x15\x16\x17\x18\x19\x1a\x1b\x1c\x1d\x1e\x1f !"#$%&\'()*+,-./01234567'
```

- Agora sem privilégios root, executamos novamente `sniffer.py`

```bash
$ su seed
$ ./sniffer.py
```

![image](/screenshots/LB13_4.png)

- Com privilégios de root: O sniffer consegue capturar pacotes porque sniffing exige acesso direto à interface de rede, algo restrito a utilizadores com permissões administrativas. Sem privilégios de root: O programa falha porque os sistemas operacionais restringem acesso direto ao hardware de rede para proteger a privacidade e a segurança.

- Ao capturar um pacote ICMP com o pkt.show(), as seguintes camadas serão visíveis:

  1. Ethernet:
    - Contém os endereços MAC de origem e destino.
  2. IP:
    - Endereços IP de origem e destino.
    - TTL e protocolo (ICMP neste caso).
  3. ICMP:
    - Tipo (ex.: Echo Request ou Echo Reply).
    - Código e checksum.

## Task 1.1.B

- Configurámos filtros BPF (Berkeley Packet Filter) no nosso programa de sniffing para capturar apenas pacotes específicos.

  ### Filtro 1: Capturar Apenas Pacotes ICMP

  - Feito anteriormente

```py
pkt = sniff(iface='br-5b39555aa03d', filter='icmp', prn=print_pkt)
```
```bash
root@0e66e2208714:/# ping 10.9.0.6 # do hostA para hostB
root@bf0a6620e04a:/# ping 10.9.0.5 # do hostB para hostA
```

  ### Filtro 2: Capturar Pacotes TCP Vindos de um IP Específico para a Porta 23

  ```py
  sniff(iface='br-c93733e9f913', filter='tcp and src host 10.9.0.5 and dst port 23', prn=print_pkt)
  ```

  - Para testar a captura de pacotes usamos o seguinte comando:

  ```bash
  root@0e66e2208714:/# telnet 10.9.0.6 23
  ```

  - Apenas pacotes TCP com origem no IP 10.9.0.5 e destino na porta 23 de hostB foram capturados.

  ```bash
  ###[ Ethernet ]### 
    dst       = 02:42:0a:09:00:06
    src       = 02:42:0a:09:00:05
    type      = IPv4
  ###[ IP ]### 
      version   = 4
      ihl       = 5
      tos       = 0x10
      len       = 60
      id        = 3766
      flags     = DF
      frag      = 0
      ttl       = 64
      proto     = tcp
      chksum    = 0x17da
      src       = 10.9.0.5
      dst       = 10.9.0.6
      \options   \
  ###[ TCP ]### 
          sport     = 48362
          dport     = telnet
          seq       = 1996051077
          ack       = 0
          dataofs   = 10
          reserved  = 0
          flags     = S
          window    = 64240
          chksum    = 0x144b
          urgptr    = 0
          options   = [('MSS', 1460), ('SAckOK', b''), ('Timestamp', (3671324248, 0)), ('NOP', None), ('WScale', 7)]

  ###[ Ethernet ]### 
    dst       = 02:42:0a:09:00:06
    src       = 02:42:0a:09:00:05
    type      = IPv4
  ###[ IP ]### 
      version   = 4
      ihl       = 5
      tos       = 0x10
      len       = 52
      id        = 3767
      flags     = DF
      frag      = 0
      ttl       = 64
      proto     = tcp
      chksum    = 0x17e1
      src       = 10.9.0.5
      dst       = 10.9.0.6
      \options   \
  ###[ TCP ]### 
          sport     = 48362
          dport     = telnet
          seq       = 1996051078
          ack       = 435428466
          dataofs   = 8
          reserved  = 0
          flags     = A
          window    = 502
          chksum    = 0x1443
          urgptr    = 0
          options   = [('NOP', None), ('NOP', None), ('Timestamp', (3671324259, 1846721987))]

  ...

  ```

  ### Filtro 3: Capturar Pacotes de ou para uma Sub-rede

  ```py
  sniff(iface='br-c93733e9f913', filter='net 128.230.0.0/16', prn=print_pkt)
  ```

  - Para testar a captura de pacotes usamos o seguinte comando:

  ```bash
  root@0e66e2208714:/# ping 128.230.1.1
  ```
  - Não havia conectividade com a sub-rede 128.230.0.0/16, resultando em 100%  de perda de pacotes.

- Os filtros BPF permitiram capturar apenas os pacotes de interesse, reduzindo o tráfego desnecessário no sniffer. Estes filtros são ferramentas eficazes para refinar a captura de pacotes em sniffers.

## Task 1.2





