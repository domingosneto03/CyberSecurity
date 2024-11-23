# Secret-Key Encryption Lab

## Task 1

- Primeiro corremos o programa `freq.py` para obter a análise de frequências das letras presentes em `ciphertext.txt`

```bash
$ ./freq.py
-------------------------------------
1-gram (top 20):
n: 488
y: 373
v: 348
...
-------------------------------------
2-gram (top 20):
yt: 115
tn: 89
mu: 74
...
-------------------------------------
3-gram (top 20):
ytn: 78
vup: 30
mur: 20
...
```

- Após uma breve análise do output e do texto encriptado, acedemos as plataformas indicadas que mostravam a frequência de determinadas letras na língua inglesa.

- Aos poucos fomos desencriptando as letras por base do que saltava mais à vista como por exemplo a substituição do trigrama `ytn` por `THE`

| Trigram | Freq  |
|---------|-------|
| the     | 1.81% |
| and     | 0.73% |
| tha     | 0.33% |

- Depois de uma série de tentativas tornou-se mais fácil usar o próprio texto para decifrar as restantes letras, acabando por obter a seguinte chave:

```bash
$ tr 'vgapnbrtmosicuxejhqyzflkdw' 'ABCDEFGHIJKLMNOPQRSTUVWXYZ' < ciphertext.txt > plaintext.txt
```

```txt
THE OSCARS TURN  ON SUNDAY WHICH SEEMS ABOUT RIGHT AFTER THIS LONG STRANGE
AWARDS TRIP THE BAGGER FEELS LIKE A NONAGENARIAN TOO

THE AWARDS RACE WAS BOOKENDED BY THE DEMISE OF HARVEY WEINSTEIN AT ITS OUTSET
AND THE APPARENT IMPLOSION OF HIS FILM COMPANY AT THE END AND IT WAS SHAPED BY
THE EMERGENCE OF METOO TIMES UP BLACKGOWN POLITICS ARMCANDY ACTIVISM AND
A NATIONAL CONVERSATION AS BRIEF AND MAD AS A FEVER DREAM ABOUT WHETHER THERE
OUGHT TO BE A PRESIDENT WINFREY THE SEASON DIDNT JUST SEEM EXTRA LONG IT WAS
EXTRA LONG BECAUSE THE OSCARS WERE MOVED TO THE FIRST WEEKEND IN MARCH TO
AVOID CONFLICTING WITH THE CLOSING CEREMONY OF THE WINTER OLYMPICS THANKS
PYEONGCHANG

ONE BIG QUESTION SURROUNDING THIS YEARS ACADEMY AWARDS IS HOW OR IF THE
CEREMONY WILL ADDRESS METOO ESPECIALLY AFTER THE GOLDEN GLOBES WHICH BECAME
A JUBILANT COMINGOUT PARTY FOR TIMES UP THE MOVEMENT SPEARHEADED BY 
POWERFUL HOLLYWOOD WOMEN WHO HELPED RAISE MILLIONS OF DOLLARS TO FIGHT SEXUAL
HARASSMENT AROUND THE COUNTRY

...
```

## Task 2

- Gerámos o ficheiro `plaintext.txt` e cifrámo-lo com os modos seguintes:

    1. **Electronic-Code-Book Mode** (aes-128-ecb)

    ```bash
    $ openssl enc -aes-128-ecb -e -in plaintext.txt -out cipher-ecb.bin -K 00112233445566778899aabbccddeeff

    $ openssl enc -aes-128-ecb -d -in cipher-ecb.bin -out decrypted-ecb.txt -K 00112233445566778899aabbccddeeff
    ```

    2. **Cipher Block Chaining Mode** (aes-128-cbc)

    ```bash
    $ openssl enc -aes-128-cbc -e -in plaintext.txt -out cipher-cbc.bin -K 00112233445566778889aabbccddeeff -iv 01020304050607080102030405060708

    $ openssl enc -aes-128-cbc -d -in cipher-cbc.bin -out decrypted-cbc.txt -K 00112233445566778889aabbccddeeff -iv 01020304050607080102030405060708
    ```

    3. **Counter Block Mode** (aes-128-ctr)

    ```bash
    $ openssl enc -aes-128-ctr -e -in plaintext.txt -out cipher-ctr.bin -K 00112233445566778889aabbccddeeff -iv 01020304050607080102030405060708

    $ openssl enc -aes-128-ctr -d -in cipher-ctr.bin -out decrypted-ctr.txt -K 00112233445566778889aabbccddeeff -iv 01020304050607080102030405060708
    ```

#### Flags utilizadas

`-aes-128-ecb` `-aes-128-cbc` `-aes-128-ctr`: Especifica modo de operação usado para cifrar o ficheiro.

`-e` `-d`: Indica que o processo é de encriptar e desencriptar, respetivamente.

`-in`: Input de ficheiro a ser cifrado.

`-out`: Output de ficheiro cifrado.

`-K`: Chave usada para cifrar.

`-iv`: Vetor de inicialização (IV) usado na cifragem.

- O `iv` não foi necessário no modo `aes-128-ecb`

- A `key` e o `iv` utilizados precisaram de 16 bytes para corresponder à cifra de 128 bits

#### Diferenças entre os modos

1. **aes-128-ecb:**

    - Não utiliza `-iv` porque não usa vetor de inicialização.

    - Cada bloco de texto em claro é cifrado de forma independente com a mesma chave.

    - Blocos idênticos de texto em claro geram blocos idênticos de texto cifrado, tornando-o vulnerável a ataques.

2. **aes-128-cbc:**

    - Necessita do `-iv` para cifrar.

    - Cada bloco de texto é `XOR` com o bloco cifrado anterior antes de ser cifrado, e o primeiro bloco é `XOR` com o `iv`.

    - Garante que blocos idênticos de texto em claro gerem diferentes blocos de texto cifrado.

3. **aes-128-ctr:**

    - Também necessita do `-iv`, mas o `iv` funciona como contador inicial.

    - O contador é incrementado para cada bloco, garantindo que cada bloco seja cifrado de forma única.

    - Transforma o cifrador em um cifrador de fluxo (stream cipher), eficiente e flexível.

    - É altamente eficiente e adequado para transmissões contínuas de dados (streaming).

    - Não utiliza encadeamento entre blocos (como o `cbc`), o que permite cifragem e decifragem paralelas.

    - Desde que o `iv` seja único, oferece alta segurança.



