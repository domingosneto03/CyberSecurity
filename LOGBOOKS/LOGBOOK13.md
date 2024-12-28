# Packet Sniffing and Spoofing Lab

## Setup

- Construimos o container. 

```bash
$ dcbuild
$ dcup
```
- A VM será o `atacante` e os restantes containers serão `HostA` e `HostB`

- Obtemos a shell dos containers.

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

- Nesta tarefa exploramos como gerar pacotes ICMP spoofados utilizando o Scapy. O objetivo é criar pacotes ICMP com um endereço IP de origem arbitrário e observar como o receptor responde a esses pacotes.

- Criamos o ficheiro `icmp_spoofer.py`

```py
#!/usr/bin/env python3
from scapy.all import *

ip = IP()
ip.src = "1.2.3.4"  # Endereço IP spoofado
ip.dst = "10.9.0.6"  # Destino: hostB

icmp = ICMP()

pkt = ip/icmp
send(pkt)

print("Spoofed packet sent!")
```

- Iniciamos a captura no Wireshark

- Executamos o ficheiro com permissões root:

```bash
$ chmod a+x icmp_spoofer.py
$ sudo ./icmp_spoofer.py 
```

- A captura dos pacotes foi feita com sucesso

![image](/screenshots/LB13_5.png)

- Utilizámos o Wireshark no hostB para capturar e observar o pacote ICMP spoofado.

- As características do pacote capturado foram:

  - Endereço IP de Origem: 1.2.3.4 (spoofado).
  - Endereço IP de Destino: 10.9.0.6 (hostB).
  - Tipo ICMP: Echo Request.
  - O hostB respondeu automaticamente ao pacote com um Echo Reply, enviado para 1.2.3.4 (IP spoofado).

## Task 1.3

- Nesta tarefa temos como objetivo enviar pacotes ICMP (Echo Request) para um IP externo, como 8.8.8.8 (servidor DNS do Google) e capturar os pacotes ICMP para analisar o tráfego gerado

- No shell do hostA, enviamos pacotes ICMP para 8.8.8.8:

```bash
root@0e66e2208714:/# ping 8.8.8.8
```

- O servidor 8.8.8.8 aceita pacotes ICMP e responde automaticamente com Echo Reply, tornando-o ideal para testes de conectividade.

![image](/screenshots/LB13_6.png)
![image](/screenshots/LB13_7.png)

- Os pacotes ICMP capturados incluem:
  - Origem e Destino (IP):
    - Echo Request: Origem = 10.9.0.5, Destino = 8.8.8.8.
    - Echo Reply: Origem = 8.8.8.8, Destino = 10.9.0.5
  - Tipo e Código (ICMP):
    - Echo Request (Tipo 8, Código 0).
    - Echo Reply (Tipo 0, Código 0).
  
- Começamos então a implementar o nosso próprio traceroute utilizando o scapy. Criamos o ficheiro `traceroute_tool.py`

```py
#!/usr/bin/env python3
from scapy.all import *

# Definir o destino
destination = "8.8.8.8"

for ttl in range(1, 30):
    print(f"[*] Sending packet with TTL={ttl}")
    pkt = IP(dst=destination, ttl=ttl) / ICMP()
    reply = sr1(pkt, verbose=0, timeout=2)

    if reply is None:
        print(f"[!] TTL={ttl}: No response")
        continue

    print(f"[+] TTL={ttl}: Received from {reply.src}")

    # Verificar se alcançou o destino
    if reply.type == 0:  # ICMP Echo Reply
        print(f"[✓] Destiny {destination} reached in {ttl} hops!")
        break
```

- Executamos o ficheiro

```bash
$ chmod a+x traceroute_tool.py
$ sudo ./traceroute_tool.py
```
![image](/screenshots/LB13_8.png)

- O Wireshark e o Scapy funcionam como ferramentas complementares: o Scapy envia os pacotes e o Wireshark ajuda a observar o tráfego para confirmar os resultados.

## Task 1.4

- Nesta tarefa, o objetivo é capturar pacotes ICMP Echo Requests (ping), filtrar os pacotes desejados, e então responder a esses pacotes com ICMP Echo Replies personalizados utilizando Scapy.

- Criamos o ficheiro `icmp_sniffer_responder.py`

```py
#!/usr/bin/env python3
from scapy.all import *

def respond_to_icmp(pkt):
    if ICMP in pkt and pkt[ICMP].type == 8:
        print(f"[+] Captured Echo Request from {pkt[IP].src}")
        
        reply = IP(src=pkt[IP].dst, dst=pkt[IP].src) / ICMP(type=0, id=pkt[ICMP].id, seq=pkt[ICMP].seq)
        send(reply)
        print(f"[+] Echo Reply sent to {pkt[IP].src}")

print("[*] Initializing sniffer...")
sniff(iface='br-5b39555aa03d', filter="icmp", prn=respond_to_icmp)
```

- Enquanto o script estava em execução, o hostA enviou pacotes ICMP para o 10.9.0.1

```bash
$ chmod a+x icmp_sniffer_responder.py
$ sudo ./icmp_sniffer_responder.py
```
```bash
root@0e66e2208714:/# ping 10.9.0.1
```

- O script capturou os pacotes ICMP Echo Request enviados pelo hostA.

- Para cada Echo Request, um Echo Reply foi gerado e enviado ao hostA com sucesso.

![image](/screenshots/LB13_9.png)
![image](/screenshots/LB13_10.png)
![image](/screenshots/LB13_11.png)

- Alguns pacotes duplicados apareceram devido à resposta automática da VM e ao script Scapy, que enviaram respostas ICMP simultaneamente.

- O sniffer capturou os Echo Requests e gerou respostas personalizadas corretamente. O tráfego capturado no Wireshark validou o sucesso do envio e recebimento de pacotes ICMP.
