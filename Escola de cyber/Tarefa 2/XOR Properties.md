# XOR Properties
###### Solved by @Mayaracar2
> This is a CTF about cryptography, xor
## Desafio: XOR Properties (cryptography)
### Introdução
O desafio apresenta o seguinte enunciado:

"In the last challenge, you saw how XOR worked at the level of bits. In this one, we're going to cover the properties of the XOR operation and then use them to undo a chain of operations that have encrypted a flag. Gaining an intuition for how this works will help greatly when you come to attacking real cryptosystems later, especially in the block ciphers category.

There are four main properties we should consider when we solve challenges using the XOR operator
Let's break this down. Commutative means that the order of the XOR operations is not important. Associative means that a chain of operations can be carried out without order (we do not need to worry about brackets). The identity is 0, so XOR with 0 "does nothing", and lastly something XOR'd with itself returns zero.

Let's put this into practice! Below is a series of outputs where three random keys have been XOR'd together and with the flag. Use the above properties to undo the encryption in the final line to obtain the flag."


#### Tradução
"No último desafio, você viu como o XOR funcionava no nível de bits. Neste, abordaremos as propriedades da operação XOR e as usaremos para desfazer uma cadeia de operações que criptografou uma flag. Ter uma noção de como isso funciona ajudará muito quando você for atacar criptosistemas reais posteriormente, especialmente na categoria de cifras de bloco.

Há quatro propriedades principais que devemos considerar ao resolver desafios usando o operador XOR.
Vamos analisar isso. Comutativo significa que a ordem das operações XOR não é importante. Associativo significa que uma cadeia de operações pode ser realizada sem ordem (não precisamos nos preocupar com colchetes). A identidade é 0, então XOR com 0 "não faz nada" e, por fim, algo que é submetido a um XOR sobre si mesmo retorna zero.

Vamos colocar isso em prática! Abaixo está uma série de saídas onde três chaves aleatórias foram submetidas a um XOR em conjunto e com o sinalizador. Use as propriedades acima para desfazer a criptografia na linha final e obter o sinalizador."

- [Página do desafio](https://cryptohack.org/courses/intro/xor1/)

### Análise inicial
A proposta do desafio gira em torno da operação lógica XOR, também conhecida como "ou exclusivo" e apresenta a seguinte tabela:

![image](https://github.com/user-attachments/assets/ac4e9168-505a-4042-8f7f-21cecac192c1)

XOR (ou "ou exclusivo") é uma operação lógica que retorna 1 apenas quando os dois bits comparados são diferentes — ou seja, um é 1 e o outro é 0. Se ambos forem iguais (0 e 0 ou 1 e 1), o resultado é 0.

Essa operação é amplamente utilizada em áreas como eletrônica digital, criptografia e programação. Em linguagens como Python, o operador XOR é representado pelo símbolo `^`.

Por exemplo:

`1 ^ 0 = 1`

`1 ^ 1 = 0`

A operação XOR (⊕) tem algumas propriedades importantes que facilitam seu uso:

COMUTATIVA:

`A ⊕ B = B ⊕ A`

A ordem dos operandos não importa. Isso significa que você pode trocar a posição dos valores sem alterar o resultado.

ASSOCIATIVA:

`(A ⊕ B) ⊕ C = A ⊕ (B ⊕ C)`

Assim como na adição comum, é possível agrupar os valores de diferentes formas ao aplicar o XOR, e o resultado será o mesmo.

ELEMENTO NEUTRO:

`A ⊕ 0 = A`

Qualquer valor XOR com 0 continua sendo ele mesmo. Isso significa que 0 é o elemento neutro do XOR.

INVERSO:

`A ⊕ A = 0`

Um valor XOR com ele mesmo resulta sempre em 0. Isso é muito útil, por exemplo, para "desfazer" operações em algoritmos (como em criptografia ou troca de variáveis sem variável auxiliar).

Além disso, foi apresentado as seguintes informações:

![image](https://github.com/user-attachments/assets/c1a221b4-9e5a-4b7a-a4ca-384c195b7dcc)


## Solução
Para resolver o desafio, foi utilizado Python no terminal do Kali Linux. Primeiramente, ativou-se um ambiente virtual com o comando:

```bash.py
source venv/bin/activete
```

A ativação do ambiente virtual (`venv`) cria uma cópia isolada do interpretador Python que permite instalar pacotes sem afetar o ambiente global do seu sistema. Isso é útil principalmente quando você tem diferentes projetos que usam diferentes versões de bibliotecas.

Em seguida, foi executado:

```bash.py
python3
```
No interpretador, utilizou-se o seguinte código:

![image](https://github.com/user-attachments/assets/1be55d01-7b90-43f1-bd1a-ef3964d64e6e)


Explicação das funções utilizadas:

```bash.py
from pwn import xor
```

Importa a função xor da biblioteca pwntools, que facilita operações de XOR entre bytes.

```bash.py
key1 = bytes.fromhex("a6c8b6733c9b22de7bc0253266a3867df55acde8635e19c73313")
key2_xor_key1 = bytes.fromhex("37dbc292030faa90d07eec17e3b1c68daf94c354dc991a5e1e")
key2_xor_key3 = bytes.fromhex("c1547556687e7573db23aac13452a098b71a7bf0fddddde5fc1")
flag_xor_keys = bytes.fromhex("04ee9855208a2cd590910d0476a7e47963170d1660d7f56f5faf")
```

`key1` é conhecida diretamente.

`key2_xor_key1` representa `key2 ⊕ key1`

`key2_xor_key3` representa `key2 ⊕ key3`

`flag_xor_keys` representa `flag ⊕ key1 ⊕ key2 ⊕ key3`


```bash.py
key2 = xor(key1, key2_xor_key1)
```

Usa a propriedade:

A ⊕ B = C → C ⊕ A = B

Logo, `key2 = key1 ⊕ (key2 ⊕ key1)`


```bash.py
key3 = xor(key2, key2_xor_key3)
```

Utiliza a mesma propriedade anterior:

`key3 = key2 ⊕ (key2 ⊕ key3)`


```bash.py
flag = xor(flag_xor_keys, key1, key2, key3)
```

Aqui, temos:

`flag_xor_keys = flag ⊕ key1 ⊕ key2 ⊕ key3`

Usamos novamente a propriedade do XOR para "cancelar" os três valores:

`flag = flag_xor_keys ⊕ key1 ⊕ key2 ⊕ key3`

```bash.py
print("FLAG:", flag.decode())
```

Decodifica os bytes da flag para texto legível.

E por fim, printamos o resultado: 

>`crypto{x0r_i5_ass0c1at1v3}`

Este desafio mostra como a operação XOR, comum em criptografia simétrica, pode ser usada tanto para proteger quanto para quebrar sistemas quando mal aplicada. A partir de uma chave conhecida e de combinações XOR expostas, foi possível reconstruir outras chaves e decifrar a flag. Isso evidencia riscos como a reutilização de chaves e a exposição de dados intermediários. O problema ensina noções básicas de criptografia, criptanálise por reconstrução e reforça a importância de manter chaves sempre secretas para garantir a segurança da informação.

 
