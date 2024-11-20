# Secret-Key Encryption Lab

## Task 1

- Primeiro corremos o programa `freq.py` para obter a análose de frequências das letras presentes em `ciphertext.txt`

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