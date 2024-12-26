# Public-Key Infrastructure Lab

## Initial Setup

- Construimos o container. 

```bash
$ dcbuild
$ dcup
```

![image](/screenshots/LB10_1.png)

- Obtemos a shell do container.

```bash
$ dockps
$ docksh 47
```

- Tentativa mal sucedida de configurar o DNS.

```bash
$ sudo nano /etc/hosts
```

![image](/screenshots/LB11_1.png)

## Task 1

- Tinhamos como onjetivo criar um certificado autoassinado para estabelecer uma Autoridade Certificadora (CA) raiz. Este será usado para emitir certificados para outros servidores ou entidades.

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


