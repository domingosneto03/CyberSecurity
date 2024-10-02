# Trabalho realizado nas Semanas #2 e #3

## Identificação

- CVE-2021-3156 é uma vulnerabilidade que pode ser usada por qualquer utilizador para ganhar root privileges.
- Não foi notada por 10 anos quando em 2021 foi descoberta por uma equipa de investigadores da Qualys.
- Eles descobriram um Heap-Based Buffer Overflow no Sudo que afetou todos os sistemas baseados em Unix.
- A 3 Feb, 2021 foi reportado que o macOS,AIX, e Solaris também eram vulneráveis.

## Catalogação

- A 13-01-2021 Todd.Miller@sudo foi avisado, 19-01-2021 patches foram enviados para distros@openwall e 26-01-21 foi corrigida.
- Notaram um potencial overflow no set_cmnd() apesar de parecer impossível pelo parse_args()’s escaping.
- Descobriram maneiras de contornar o escaping e chegar ao truque do sudoedit.

## Exploit

- Escrevendo um bruteforce script que cria variáveis LC aleatórias e o tamanho do buffer to overflow e ver onde o sudo falha.
- A maioria das exploits aproveita-se de um crash na função nss_lookup_function.
- Existem muitos scripts no GitHub para o Linux usando a exploit acima mas esse método não funciona tão bem no macOS.
- Outras exploits foram encontradas e existem multiplas formas de explorar esta vulnerabilidade.


## Ataques

- Várias exploits foram usadas para atacar sistemas Unix.
- Afeta todas as versões legacy from 1.8.2 to 1.8.31p2 e stable versions 1.9.0 to 1.9.5p1, in their default configuration do sudo.
- Sistemas que não tenha sido atualizados continuam em constante risco de exploração.
- Relatos em diversos sites conseguiram root privileges no Fedora,Debian e outras distros comuns.
