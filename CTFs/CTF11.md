# Weak Encryption

## Reconhecimento

- Ao analisar o algoritmo de geração de geração de chaves, cifração e decifração dado na descrição do CTF descobrimos uma vulnerabilidade importante :

```py
from cryptography.hazmat.primitives.ciphers import Cipher, algorithms, modes
import os

KEYLEN = 16

def gen(): 
	offset = 3 # Hotfix to make Crypto blazing fast!!
	key = bytearray(b'\x00'*(KEYLEN-offset)) 
	key.extend(os.urandom(offset))
	return bytes(key)

def enc(k, m, nonce):
	cipher = Cipher(algorithms.AES(k), modes.CTR(nonce))
	encryptor = cipher.encryptor()
	cph = b""
	cph += encryptor.update(m)
	cph += encryptor.finalize()
	return cph

def dec(k, c, nonce):
	cipher = Cipher(algorithms.AES(k), modes.CTR(nonce))
	decryptor = cipher.decryptor()
	msg = b""
	msg += decryptor.update(c)
	msg += decryptor.finalize()
	return msg
```
- Na parte da geração da chave o código define que os primeiros KEYLEN - offset bytes da chave são preenchidos com ```b'\x00'``` (zero bytes).

- Apenas os últimos offset bytes são gerados de forma aleatória com os.urandom(offset).

- Ao alterar o código para aumentar a velocidade do algoritmo o programador tornou a chave muito previsível (neste caso, 13 bytes de zeros, se KEYLEN é 16 e offset é 3). Essa previsibilidade reduz significativamente o espaço de chave possível.

```py
offset = 3 # Hotfix to make Crypto blazing fast!!
key = bytearray(b'\x00'*(KEYLEN-offset)) 
key.extend(os.urandom(offset))
return bytes(key)
```

## Ataque

- Para explorar a vulnerabilidade criámos este código que usa brute force para tentar chegar aos três bytes aleatórios.

```py
from cryptography.hazmat.primitives.ciphers import Cipher, algorithms, modes
import string

def brute_force_key_no_plaintext(ciphertext, nonce):
    """
    Brute-forces the low-entropy key space to recover the plaintext by searching for
    meaningful patterns or readable text.

    Args:
        ciphertext (bytes): The encrypted message.
        nonce (bytes): The nonce used for encryption.

    Returns:
        bytes: The recovered key.
        bytes: The decrypted plaintext.
    """
    KEYLEN = 16
    offset = 3  # Exploit based on the low-entropy key

    readable_chars = set(string.printable.encode())  # Printable ASCII characters

    # Generate all possible low-entropy keys
    for i in range(256 ** offset):  # Only brute-force 3 random bytes (256^3 possibilities)
        low_entropy_key = bytearray(b'\x00' * (KEYLEN - offset))
        low_entropy_key.extend(i.to_bytes(offset, byteorder='big'))

        # Try decrypting with this key
        try:
            cipher = Cipher(algorithms.AES(bytes(low_entropy_key)), modes.CTR(nonce))
            decryptor = cipher.decryptor()
            decrypted_message = decryptor.update(ciphertext) + decryptor.finalize()

            # Check if decrypted message contains mostly readable characters
            if all(byte in readable_chars for byte in decrypted_message):
                print(f"Key found: {low_entropy_key.hex()}")
                print(f"Decrypted message: {decrypted_message.decode(errors='ignore')}")
                return bytes(low_entropy_key), decrypted_message
        except Exception:
            pass  # Skip invalid keys

    raise ValueError("Key not found")


# Known data
nonce = bytes.fromhex("605b402210ffdb1724252b844a72721d")
ciphertext = bytes.fromhex("92cf89e7ea754ad5060fa5e2db231d88b09ec6e08921")

# Attempt brute-force attack
try:
    recovered_key, recovered_plaintext = brute_force_key_no_plaintext(ciphertext, nonce)
except ValueError as e:
    print(e)
```

***Explicação do Código:***

1.	Exploração da baixa entropia da chave:
    -	O código sabe que a chave começa com uma parte fixa (b'\x00' * (KEYLEN - offset)) e termina com apenas 3 bytes aleatórios.

    -	Ele tenta todas as combinações possíveis para esses 3 bytes.
    
2.	Decriptação tentativa:
    -	Para cada chave gerada, tenta descriptografar o texto cifrado usando o algoritmo AES-CTR com o nonce conhecido.

3.	Verificação do texto descriptografado:
    -	Após descriptografar com uma chave candidata, o código verifica se o texto resultante contém caracteres legíveis (definidos em string.printable), assumindo que o texto original seja algo legível.

- Assim chegámos à flag após aproximadamente 2 minutos de execução do código:



![Flag](/screenshots/CTF11_1.png)
