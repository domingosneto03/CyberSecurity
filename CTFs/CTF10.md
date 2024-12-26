# Classical Encryption

## Reconhecimento

- Efetuamos uma breve análise ao ficheiro cifrado que nos foi atribuído e levámos em conta a informação de que se tratava de um excerto de um jornal português cifrado com uma cifra clássica.

- Fizemos a contagem de cada elemento para analiar as suas freqências.

```py
#!/usr/bin/env python3

from collections import Counter
import re

TOP_K  = 30
N_GRAM = 1

# Generate all the n-grams for value n
def ngrams(n, text):
    for i in range(len(text) -n + 1):
        # Ignore n-grams containing white space
        if not re.search(r'\s', text[i:i+n]):
           yield text[i:i+n]

# Read the data from the ciphertext
with open('L14G07.cph') as f:
    text = f.read()

# Count, sort, and print out the n-grams
for N in range(N_GRAM):
   print("-------------------------------------")
   print("{}-gram (top {}):".format(N+1, TOP_K))
   counts = Counter(ngrams(N+1, text))        # Count
   sorted_counts = counts.most_common(TOP_K)  # Sort 
   for ngram, count in sorted_counts:                  
       print("{}: {}".format(ngram, count))   # Print
```

- Obtivemos a seguinte organização dos dados:

![image](/screenshots/CTF10_1.png)

- Como se trata de um excerto sem espaços, apenas considerámos a frequência de um elemento (1-grama).

## Comparação

- Visto ser uma cifra clássica, explorámos a previsibilidade da frequência de letras esperada. Sendo o excerto de um jornal português, pesquisamos pela freqência de letras mais utiilizadas na língua portuguesa para servir de nosso guia inicial.

- Segundo o Wikipedia, as letras mais frequentes na llíngua portuguesa são as seguintes:

![image](/screenshots/CTF10_2.png)

- Devido ao facto da frequência nos primeiros símbolos ser unanimamente alta e diminuir drasticamente a partir do 13º elemento da tabela, previmos que a probabilidade da chave divergir bastante dos dados da tabela de frequência de letras mais usadas seria elevada.

- No excerto dado, seguindo a ordem da tabela 1-gram, os 3 primeiros símbolos têm uma frequência que se destaca e os 10 seguintes têm uma frequência muito parecida, o que torna complexa a determinação da letra correspondente. Os restantes símbolos têm uma frequência bastante reduzida em comparação aos anteriores e são os que estão mariotariamente presentes na flag que queremos descobrir por estarem entre `{}` (os únicos símbolos que não são cifrados).

## Decifração

- Começamos por substituir `<` e `|` por `A` e `E` por serem os primeiros elementos das tabelas. Para nossa surpresa, reparámos que não era algo que fazia sentido porque a sílaba `ae` não aparece frequentemente nas palavras portuguesas. 

![image](/screenshots/CTF10_3.png)

_tr '<|' 'AE'_

- Tentámos trocar a ordem das letras para `ea` pelo que pareceu uma boa abordagem inicialmente, mas com o decorrer das substituições perdeu sentido. Portanto, para facilitar, assumimos que `<` correspondia a `A` de acordo com a tabela de frequências.

- O símbolo `|` substituímos pela letra `O` por ser a terceira letra mais frequente e pelo conjunto `ao` fazer mais sentido visto ser dos ditongos mais presentes na nossa língua.

- Substituímos `[` por `E` para ir de acordo com a alta frequência de símbolos e letras, mas deparámo-nos com outro impasse pois estávamos confiantes que o símbolo seguinte teria que ser um `S` ou um `R` e que seria possível observar algumas palavras simples a se formarem. Isso não aconteceu pelo que nenhuma das letras fez sentido a longo prazo. Algo que dificultou a substituição por `S` ou `R` foi a falta de palavras com letras iguais seguidas que serviria como confirmação de uma das letras indicadas.

![image](/screenshots/CTF10_4.png)

_tr '<|[~>' 'AOESR'_

![image](/screenshots/CTF10_5.png)

_tr '<|[~>' 'AOERS'_

- Após diversas tentativas de combinações diferentes e reformulações da chave, voltámos a confiar na nossa chave inicial `(AOE)` e decidimos substituir `~` por `I`. Dificilmente considerariamos esta opção pois isto significaria que até agora só teriam sido descobertas vogais. No entanto, esta adversividade serviu de aviso de que, entre o símbolo `~` e o símbolo `$` da tabela, as frequências são parecidas ao ponto de poderem corresponder a letras inesperadas.

- Foi nesta altura também que percebemos que nesta fase seria fulcral procurar pela presença de palavras que dificilmente existam ou façam sentido, ao invés de procurar e tentar formar palavras que existam, de modo a descartar possibilidades com mais facilidade. Essa maneira abstrata de pensar ajudou a avançar na tarefa de decifrção. No entanto, tivemos de ter precaução e cosniderar que, o que não fizer sentido pode ser, na verdade, mais que uma palavra junta, por não haver espaços no excerto cifrado.

- Assim, com alguma tentativa e erro, substituímos `>`, `%` e `*` por `S`, `N`, `R`, respetivamente. Finalmente começaram a aparecer algumas palavras, embora simples, como `anos`. Também nos saltou à vista uma secção do excerto: `A/ERI(ANOS` que poderia ser a palavra `americanos`. Conseguimos assim, finalmente, entrar no percurso correto da decifração, sabendo assim a letra seguinte a substituir.

![image](/screenshots/CTF10_6.png)

_tr '<|[~>%*' 'AOEISNR'_

- Substituimos, portanto, o símbolo `(` (que coincidentemente era o símbolo seguinte da nossa tabela 1-gram) por `C` e foi possível identifcar palavras como `corre` e `ação`. O símbolo `/` já tinhamos a certeza que corresponderia à letra `M` mas de modo a seguir a ordem da tabela deixamos essa substituição "em espera".

- Alterando cada vez mais letras usando a tabela de frequências e o sentido do excerto, conseguimos encontrar mais palavras como `sociedade`, `sindicato`, `corretora`, etc.

![image](/screenshots/CTF10_7.png)

_tr '<|[~>%*(.,/$' 'AOEISNRTDMP'_

- Por vezes, inserimos espaços no excerto para ter mais certeza de que a decifração estava correta

![image](/screenshots/CTF10_8.png)

- Nesta fase, para além de termos descoberto diversas palavras, descobrimos a correspondência correta dos símbolos mais frequentes no excerto. Isso facilitou a descoberta das restantes letras, pelo que havia cada vez menos espaço para ambiguidades, aproximando-nos assim do excerto completo e da flag correta.

- A tática usada para decifrar as restantes letras foi semelhante ao método usado para encontrar a palavra `americanos`, que posteriormente nos apercebemos que era apenas `americano` mas que não nos desviou da chave correta. Basicamente ao isolarmos a palavra torna-se fácil perceber o símbolo em falta, como por exemplo, a secção `IN]ESTIMENTOS` que seria a palavra `investimentos` e indicativo de que o símbolo `]` corresponde à letra `V`.

- Conseguimos então decifrar o excerto completo.

![image](/screenshots/CTF10_9.png)

_tr '<|[~>%*(.,/$-:_!?]=@&^' 'AOEISNRCTDMPLUFBGVHZQJ'_

- Ao inserirmos espaços percebemos que a nossa resposta estava correta, apesar de não ter sido um texto fácil de entender. 

![image](/screenshots/CTF10_10.png)

- Descobrimos então a flag sendo: `flag{zgdqsngpztmvcbct}`



