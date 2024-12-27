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

