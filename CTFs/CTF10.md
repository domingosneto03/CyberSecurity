# Classical Encryption

## Reconhecimento

- Efetuamos uma breve análise ao ficheiro cifrado que nos foi atribuido e levamos em conta a informação de que se tratava de um excerto de um jornal português cifrado com uma cifra clássica.

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

- Como se trata de um excerto sem espaços apenas consideramos a frequência de um elemento (1-grama).

## Comparação

- Visto ser uma cifra clássica explorámos a previsibilidade da frequência de letras esperada. Sendo o excerto de um jornal português, pesquisamos pela freqência de letras mais utiilizadas na língua portuguesa para servir de nosso guia inicial.

- Segundo o Wikipedia, as letras mais frequentes na llíngua portuguesa são as seguintes:

![image](/screenshots/CTF10_2.png)

- Devido ao facto da frequência nos primeiros símbos ser unanimamente alta e diminuir drasticamente a partir do 13º elemento da tabela, previmos que a probabilidade da chave divergir bastante dos dados da tabela de frequência de letras mais usadas seria alta.

- No excerto dado, seguindo a ordem da tabela 1-gram, os 3 primeiros símbolos têm uma frequência que se destaca, os 10 seguintes têm uma frequência muito parecida que torna dificil determinar a letra correspondente, os restantes símbolos têm uma frequência bastante reduzida em comparação aos anteriores e são os que estão mariotariamente presentes na flag que queremos descobrir por estarem entre `{}` (os únicos símbolos que não são cifrados).

## Decifração

