# RSA

## Etapas

### Função que testa a primalidade de números

- Assim como aconcelhado nas tarefas do CTF começámos por criar uma função que usa o algoritmo Miller-Rabin para encontrar uma lista de primos próximos dos números submetidos :

```py
def miller_rabin(n, k=5):
    if n <= 1:
        return False
    if n <= 3:
        return True
    if n % 2 == 0:
        return False

    # Escreve n-1 como 2^r * d, com d ímpar
    r, d = 0, n - 1
    while d % 2 == 0:
        r += 1
        d //= 2

    # Testa k vezes
    for _ in range(k):
        a = random.randint(2, n - 2)  # Escolhe um número aleatório em [2, n-2]
        x = pow(a, d, n)  # Calcula (a^d) % n
        if x == 1 or x == n - 1:
            continue

        for _ in range(r - 1):
            x = pow(x, 2, n)  # Calcula (x^2) % n
            if x == n - 1:
                break
        else:
            return False  # Composto

    return True  # Provavelmente primo

def find_nearest_prime(num, direction="both", max_search=1000):
    primes = []
    
    if direction in ("both", "up"):
        for i in range(num + 1, num + max_search):
            if miller_rabin(i):
                primes.append(i)
                if direction != "both":
                    break

    if direction in ("both", "down"):
        for i in range(num - 1, num - max_search, -1):
            if i > 0 and miller_rabin(i):
                primes.append(i)
                if direction != "both":
                    break

    return sorted(primes, key=lambda x: abs(x - num)) if direction == "both" else primes
```

### Função extendedEuclidean(a,b)


A função implementa o **Algoritmo Estendido de Euclides** para calcular o inverso modular de `a` em relação a `b`, assumindo que \( \text{gcd}(a, b) = 1 \).

```py
def extendedEuclidean(a, b):
    x0, x1, y0, y1 = 1, 0, 0, 1
    original_b = b
    while b != 0:
        q, a, b = a // b, b, a % b
        x0, x1 = x1, x0 - q * x1
        y0, y1 = y1, y0 - q * y1
    if a != 1:
        raise ValueError("O inverso modular não existe, gcd ≠ 1")
    return x0 % original_b
```

#### Passos:
1. **Inicialização**:
   - Variáveis \( x_0, x_1, y_0, y_1 \) representam os coeficientes da equação de Bézout (\( a \cdot x + b \cdot y = \text{gcd}(a, b) \)).
   - Armazena o valor original de `b` para calcular o módulo no final.

2. **Loop do Algoritmo de Euclides**:
   - Atualiza \( a \) e \( b \) com o quociente (\( q \)) e o resto da divisão inteira.
   - Ajusta os coeficientes \( x \) e \( y \) para refletir as mudanças.

3. **Verificação**:
   - Se o \( \text{gcd}(a, b) \neq 1 \), não há inverso modular, e um erro é lançado.

4. **Retorno**:
   - Retorna \( x_0 \mod b \), que é o inverso modular de `a` em relação a `b`.

### Criar a função dec(ciphertext_hex, d, n)

```py
def dec(ciphertext_hex, d, n):
    c_bytes = bytes.fromhex(ciphertext_hex)
    c_int = int.from_bytes(c_bytes, "little")
    m_int = pow(c_int, d, n)
    m_bytes = m_int.to_bytes((m_int.bit_length() + 7) // 8, 'little')
    m_bytes = m_bytes.rstrip(b'\x00')
    try:
        return m_bytes.decode('utf-8')
    except UnicodeDecodeError:
        return m_bytes.decode('latin-1', errors='ignore')
```
#### Passos:
1. **Hexadecimal para bytes**:
   - Converte o texto cifrado em hexadecimal para bytes:
     ```python
     c_bytes = bytes.fromhex(ciphertext_hex)
     ```

2. **Bytes para inteiro**:
   - Interpreta os bytes como um número inteiro (little-endian):
     ```python
     c_int = int.from_bytes(c_bytes, "little")
     ```

3. **Exponenciação modular**:
   - Calcula \( m = c^d \mod n \):
     ```python
     m_int = pow(c_int, d, n)
     ```

4. **Inteiro para bytes**:
   - Converte o resultado de volta para bytes no formato little-endian:
     ```python
     m_bytes = m_int.to_bytes((m_int.bit_length() + 7) // 8, 'little')
     ```

5. **Remover padding**:
   - Remove zeros à direita (padding):
     ```python
     m_bytes = m_bytes.rstrip(b'\x00')
     ```

6. **Decodificar texto**:
   - Converte os bytes para texto legível (UTF-8 ou latin-1):
     ```python
     return m_bytes.decode('utf-8')

### Criar a função para iterar por vários primos e descobrir a flag

```py

def brute_force_dec(ciphertext_hex, i, j, e=0x10001, max_search=10000):
    p_candidates = find_nearest_prime(2**i, direction="both", max_search=max_search)
    q_candidates = find_nearest_prime(2**j, direction="both", max_search=max_search)
    
    for p in p_candidates:
        for q in q_candidates:
            n = p * q
            phi = (p - 1) * (q - 1)
            try:
                d = extendedEuclidean(e, phi)
            except ValueError:
                # Se não for inversível, gcd != 1
                continue
            plaintext = dec(ciphertext_hex, d, n)
            if "flag{" in plaintext:  # ou qualquer outro critério
                return plaintext
    
    return None
```

Esta função utiliza as funções anteriormente criadas para descobrir o texto encriptado, tivémos que aumentar a max_search para 10000 porque com o parâmetro a 1000 não chegava aos primos corretos.

A função itera pelos resultados retornados pelo find_nearest_prime
e calcula o private exponent depois executa a função dec com o ciphertext dado , o private exponent calculado e o módulo. Se o resultado conter ```"flag{"```ele retorna o valor.

Assim encontrámos a flag :

![Flag](/screenshots/CTF12_1.png)
