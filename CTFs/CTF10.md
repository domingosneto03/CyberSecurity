# Classical Encryption

## Reconhecimento

- Efetuamos uma breve análise ao ficheiro cifrado que nos foi atribuido e levamos em conta a informação de que se tratava de um excerto de um jornal português cifrado com uma cifra clássica.

- Fiizemos a contagem de cada elemento para analiar as suas freqências.

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

