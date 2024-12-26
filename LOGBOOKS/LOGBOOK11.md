# Public-Key Infrastructure Lab

## Initial Setup

- Construimos o container. 

```bash
$ dcbuild
$ dcup
```

- Obtemos a shell do container.

```bash
$ dockps
$ docksh 47
```

- Configuramos o DNS.

```bash
$ sudo nano /etc/hosts
```

![image](/screenshots/LB11_1.png)

## Task 1

- Tinhamos como objetivo criar um certificado autoassinado para estabelecer uma Autoridade Certificadora (CA) raiz. Este será usado para emitir certificados para outros servidores ou entidades.

- Acedemos ao ficheiro openssl.cnf através do comando:
```bash
$ sudo nano /usr/lib/ssl/openssl.cnf
```

- Copiamos a secção [ CA_default ] para o nosso ficheiro `my_openssl.cnf`
```cnf
[ ca ]
default_ca = CA_default

[ CA_default ]

dir             = ./demoCA              # Where everything is kept
certs           = $dir/certs            # Where the issued certs a>
crl_dir         = $dir/crl              # Where the issued crl are>
database        = $dir/index.txt        # database index file.
unique_subject = no                    # Set to 'no' to allow cre>
                                        # several certs with same >
new_certs_dir   = $dir/newcerts         # default place for new ce>

certificate     = $dir/cacert.pem       # The CA certificate
serial          = $dir/serial           # The current serial number
crlnumber       = $dir/crlnumber        # the current crl number
                                        # must be commented out to>
crl             = $dir/crl.pem          # The current CRL
private_key     = $dir/private/cakey.pem# The private key

x509_extensions = usr_cert              # The extensions to add to>

# Comment out the following two lines for the "traditional"
# (and highly broken) format.
name_opt        = ca_default            # Subject Name options
cert_opt        = ca_default            # Certificate field options

# Extension copying option: use with caution.
# copy_extensions = copy

# Extensions to add to a CRL. Note: Netscape communicator chokes o>
# so this is commented out by default to leave a V1 CRL.
# crlnumber must also be commented out to leave a V1 CRL.
# crl_extensions        = crl_ext

default_days    = 365                   # how long to certify for
default_crl_days= 30                    # how long before next CRL
default_md      = default               # use public key default MD
preserve        = no                    # keep passed DN ordering
# A few difference way of specifying how similar the request shoul>
# For type CA, the listed attributes must be the same, and the opt>
# and supplied fields are just that :-)
policy          = policy_match
```


- Copiamos o ficheiro de configuração `openssl.cnf` para o diretório atual.
- Criamos as seguintes pastas e ficheiros:
  - `demoCA/certs`: Para armazenar os certificados emitidos.
  - `demoCA/crl`: Para armazenar listas de revogação.
  - `demoCA/newcerts`: Diretório padrão para novos certificados.
  - `demoCA/index.txt`: Ficheiro vazio criado.
  - `demoCA/serial`: Contém o número inicial do serial, por exemplo, `1000`.

- A linha `unique_subject = no` foi descomentada para permitir a criação de múltiplos certificados com o mesmo assunto.

- Utilizamos o seguinte comando para criar o certificado autoassinado:
  ```bash
  $ openssl req -x509 -newkey rsa:4096 -sha256 -days 3650 \
  -keyout ca.key -out ca.crt \
  -subj "/CN=www.domiCA.com/O=Domi CA LTDA./C=PT" \
  -passout pass:password
  ```

- Verificámos o conteúdo do certificado com os seguintes comandos:
    - Para visualizar os detalhes do certificado:
    ```bash
    openssl x509 -in ca.crt -text -noout
    ```
    - Para verificar a chave privada:
    ```bash
    openssl rsa -in ca.key -text -noout
    ```

- Foi então possível responder às seguintes perguntas:

1. What part of the certificate indicates this is a CA’s certificate?
    - A extensão `X509v3 Basic Constraints` contém o campo:
    ```graphql
    X509v3 Basic Constraints: critical
        CA:TRUE
    ```
    Isto indica que o certificado é de uma Autoridade Certificadora.

2. What part of the certificate indicates this is a self-signed certificate?
    - O campo `Issuer` e o campo `Subject` são idênticos:
    ```mathematica
    Issuer: CN=www.minhaCA.com, O=Minha CA LTDA., C=PT
    Subject: CN=www.minhaCA.com, O=Minha CA LTDA., C=PT
    ```
    Isto confirma que o certificado é autoassinado.

3. In the RSA algorithm, we have a public exponent e, a private exponent d, a modulus n, and two secret numbers p and q, such that n = pq. Please identify the values for these elements in your certificate
and key files.

    - Expoente público (e): Pode ser encontrado no ficheiro da chave pública. 
    ```makefile
    Exponent: 65537 (0x10001)
    ```
    - Expoente privado (d): Pode ser encontrado no ficheiro da chave privada.
    - Módulo (n): Valor associado à chave pública/privada.
    - Números secretos (p e q): Podem ser encontrados no ficheiro da chave privada.
    ```makefile
    prime1: [valor de p]
    prime2: [valor de q]
    modulus: [valor de n]
    ```

- O certificado gerado foi armazenado em ca.crt e a chave privada em ca.key.
- A CA está pronta para assinar pedidos de certificado (CSRs) de outras entidades.

## Task 2

- Criamos um Pedido de Assinatura de Certificado (CSR) para o servidor web **www.neto2024.com**, que será enviado à CA para ser assinado e convertido num certificado.

- Foi usado o seguinte comando para criar a chave privada e o CSR:
  ```bash
  openssl req -newkey rsa:2048 -sha256 \
  -keyout server.key -out server.csr \
  -subj "/CN=www.neto2024.com/O=Neto Servidor LTDA./C=PT" \
  -passout pass:password
  ```
- Para suportar múltiplos nomes de domínio, foi usado o seguinte comando:

```bash
openssl req -newkey rsa:2048 -sha256 \
-keyout server.key -out server.csr \
-subj "/CN=www.neto2024.com/O=Neto Servidor LTDA./C=PT" \
-passout pass:password \
-addext "subjectAltName = DNS:www.neto2024.com, DNS:www.alternativo1.com, DNS:www.alternativo2.com"
```

- O CSR inclui:
    - A chave pública do servidor.
    - Informações do servidor: Nome Comum (CN), Organização (O) e País (C).
    - Extensão de Nomes Alternativos (SAN), se adicionados.

- Nomes alternativos são adicionados porque navegadores verificam se o Nome Comum (CN) ou os Nomes Alternativos (SAN) coincidem com o hostname do servidor. Se não coincidir, a ligação será recusada.

## Task 3


