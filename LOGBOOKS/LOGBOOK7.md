# Cross-Site Scripting (XSS)

## Questão 1

### Task 1: Posting a Malicious Message to Display an Alert Window

Nesta task temos o objetivo de injetar o código ```alert("XSS");``` no Egg profile, alterando a brief description, de modo a que qualquer utilizador ao abrir o perfil receba o alerta "XSS":

![image](/screenshots/LB7_1.png)

![image](/screenshots/LB7_2.png)

### Task 2: Posting a Malicious Message to Display Cookies

Esta task também tem o objetivo de injetar código que irá ser executado sempre que algum utilizador veja o profile, no entanto retorna os cookies no alerta.

![image](/screenshots/LB7_3.png)

### Task 3: Stealing Cookies from the Victim’s Machine

Como na tarefa anterior, apenas os utilizadores podiam ver os cookies do seu próprio navegador. Nesta tarefa, nós, como atacantes, queremos que os cookies sejam enviados para a nossa máquina.

Como foi proposto, podemos fazer isso injetando uma tag HTML que realizará um pedido. Neste caso, a tag img pode ser utilizada, uma vez que o URL no campo src é usado para realizar um pedido HTTP GET. Assim, podemos escrever o endereço da máquina do atacante com uma porta específica na qual o atacante estará a ouvir. Desta forma, ao ouvir nessa porta utilizando, por exemplo, o netcat, o atacante pode ver os pedidos enviados a partir do navegador dos utilizadores, com os seus cookies como parâmetros no URL.

![image](/screenshots/LB7_4.png)

### Task 4: Becoming the Victim’s Friend

Na última tarefa, queremos escrever um verme XSS que adiciona automaticamente o utilizador "Samy" como amigo de qualquer utilizador que visite a sua página.

Podemos fazer algo semelhante às outras tarefas e incorporar um programa JavaScript no perfil de Samy. No entanto, este código deve fazer com que qualquer utilizador que visite a página envie o pedido adequado para adicionar Samy como amigo.

Utilizámos a extensão do firefox proposta para analisar os HTTP requests, ao clicar no botão "Add friend" conseguimos as seguintes informações:

![image](/screenshots/LB7_5.png)

Com esta informação, podemos criar um pedido que adicionará o Samy como amigo quando executado num script nos navegadores de outros utilizadores. Este pedido tem a seguinte URL: ```http://www.seed-server.com/action/friends/add?friend=59&__elgg_ts=user_ts&__elgg_token=user_token```, sendo que os campos ```__elgg_ts``` e ```__elgg_token``` são gerados no script, de acordo com o token atual do utilizador.

![image](/screenshots/LB7_6.png)

Relativamente às perguntas da task 4: 

Como parte do pedido HTTP a ser enviado para o servidor, precisamos das linhas 1 e 2 para identificar corretamente a pessoa que está a adicionar o Samy como amigo.

Se a aplicação Elgg fornecer apenas o modo Editor para o campo "Sobre Mim", ou seja, não for possível alternar para o modo Texto, ainda é possível lançar um ataque bem-sucedido? O modo Editor adiciona código HTML extra ao texto digitado no campo, o que faz com que o código não seja executável. Portanto, o ataque não será lançado.

## Questão 2

### Há várias modalidades de ataques XSS (Reflected, Stored ou DOM). Em qual/quais pode enquadrar este ataque e porquê?

Este ataque é considerado Stored porque o código JavaScript malicioso é injetado e salvo na base de dados. Quando um outro utilizador visita o perfil do Sammy, o código é refletido de volta e executado no navegador da vítima, sem que ela saiba.

