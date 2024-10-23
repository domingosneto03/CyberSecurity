# Environment Variables

## Reconhecimento

Após uma verificação da aplicação web, conseguimos obter informações importantes a ser exploradas:

 - O texto "This website is HARD AS A SHELL! We fear nothing!" e a imagem que sugeriu uma exploit relacionada à shell utilizada.

![image](/screenshots/CTF4_1.jpg)

- A versão da shell bash.

![image](/screenshots/CTF4_2.jpg)

- Ao submeter um formulário sem input conseguimos descobrir que ficheiros estão no cgi-bin.

![image](/screenshots/CTF4_3.jpg)

## Vulnerabilidade

O CVE encontrado é o [CVE-2014-6271](https://nvd.nist.gov/vuln/detail/CVE-2014-6271) mais conhecido como Shellshock, com um score de 9.8.


## Ataque

Conhecido o CVE, verificámos que o servidor web não aceitava GET requests pelo curl,logo tentámos o seguinte comando pelo formulário 'VAR=() { :;}; echo Bash is vulnerable!' que verificou que o servidor era vulnerável

![image](/screenshots/CTF4_4.jpg)

- Procurámos por locais comuns onde a flag poderia estar guardada tais como o /etc/, /tmp/, /var/... até que a encontrámos em:

![image](/screenshots/CTF4_5.jpg)

- Utilizámos o comando env 'VAR=() { :;}; cat /var/flag/flag.txt' para mostrar o contéudo da flag.

![image](/screenshots/CTF4_6.jpg)


