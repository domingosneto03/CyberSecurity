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

- Copiamos o ficheiro openssl.cnf para o nosso diretorio através do comando:
```bash
$ sudo cp /usr/lib/ssl/openssl.cnf ~/Labsetup/my_openssl.cnf
```
- Efetuamos mudanças no ficheiro `my-openssl.cnf` sempre que necessário

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

- Usámos a CA criada na Task 1 para assinar o Pedido de CSR gerado na Task 2, criando assim um certificado válido para o servidor **www.neto2024.com**.

- Para permitir a cópia de extensões (como os Nomes Alternativos) do CSR para o certificado final, foi descomentada a linha no ficheiro `openssl.cnf`:
```bash
copy_extensions = copy
```

- Foi usado o comando seguinte para assinar o CSR e gerar o certificado:
```bash
openssl ca -config my-openssl.cnf -policy policy_anything \
-md sha256 -days 3650 \
-in server.csr -out server.crt -batch \
-cert ca.crt -keyfile ca.key
```

- Verificamos o conteudo gerado com o comando:
```bash
openssl x509 -in server.crt -text -noout
```
![image](/screenshots/LB11_2.png)

- Os nomes alternativos foram corretamente incluídos no certificado.

## Task 4

- Configurámos um website HTTPS no servidor Apache usando o certificado gerado.

- Para isso, precisamos do CA e da chave privada correspondente. Utilizando a shell do container, criamos o um ficheiro `VirtualHost` em `/etc/apache2/sites-available/neto2024_apache_ssl.conf`

```apache
<VirtualHost *:443>
    DocumentRoot /var/www/neto2024
    ServerName www.neto2024.com
    ServerAlias www.alternativo1.com www.alternativo2.com
    DirectoryIndex index.html

    SSLEngine On
    SSLCertificateFile /certs/server.crt
    SSLCertificateKeyFile /certs/server.key
</VirtualHost>
```

- Para que as configurações funcionassem corretamente, criámos o nosso website em `/var/www/neto2024` e copiamos os certificados para a pasta /certs do container.

```bash
root@47864ee4f9d8:/# mkdir -p /var/www/neto2024
root@47864ee4f9d8:/# echo "<h1>Bem-vindo ao www.neto2024.com</h1>" > /var/www/neto2024/index.html
```
```bash
$ docker cp server.crt 47:/certs/
$ docker cp server.key 47:/certs/
```

- Habilitamos o suporte SSL e o website com os seguintes comandos:

```bash
root@47864ee4f9d8:/# a2enmod ssl
root@47864ee4f9d8:/# a2ensite neto2024_apache_ssl
root@47864ee4f9d8:/# service apache2 restart
```

![image](/screenshots/LB11_3.png)

- Com a configuração feita, acedemos ao website https://www.neto2024.com onde nos deparamos com uma proteção.

![image](/screenshots/LB11_4.png)

- A falha inicial ocorre porque a CA utilizada não é reconhecida pelos navegadores. Este é um comportamento esperado e projetado para proteger os utilizadores de certificados maliciosos. Importar o certificado da CA manualmente no navegador resolve o problema, estabelecendo a confiança entre o cliente (navegador) e o servidor.

Foi necessário então importar o certificado da CA no navegador. Portanto acedemos a `about:preferences#privacy` e selecionamos o ficheiro `ca.crt` que geramos inicialmente

![image](/screenshots/LB11_5.png)

- Este certificado é conhecido como um certificado raiz, porque foi autoassinado pela própria CA e serve como o ponto de confiança para todos os certificados que ela emitir.

- O `ca.crt` é essencial porque representa a CA local criada por nós. Ao carregar o ficheiro no navegador, garantimos que ele confie nos certificados assinados pela nossa CA, permitindo a validação correta do certificado do servidor.

- Desta forma, o nosso website já era confiado pelo navegador.

![image](/screenshots/LB11_5.png)

## Task 5
