# XOR Starter
###### Solved by @Mayaracar2
> This is a CTF about cryptography, xor
## Desafio: XOR Starter (cryptography)
### Introdução
O desafio apresenta o seguinte enunciado:

"XOR is a bitwise operator which returns 0 if the bits are the same, and 1 otherwise. In textbooks the XOR operator is denoted by ⊕, but in most challenges and programming languages you will see the caret ^ used instead.
For longer binary numbers we XOR bit by bit: 0110 ^ 1010 = 1100. We can XOR integers by first converting the integer from decimal to binary. We can XOR strings by first converting each character to the integer representing the Unicode character.

Given the string label, XOR each character with the integer 13. Convert these integers back to a string and submit the flag as crypto{new_string}."


#### Tradução
"XOR é um operador bit a bit que retorna 0 se os bits forem iguais e 1 caso contrário. Em livros didáticos, o operador XOR é denotado por ⊕, mas na maioria dos desafios e linguagens de programação, você verá o acento circunflexo ^usado em seu lugar.
Para números binários maiores, realizamos XOR bit a bit: 0110 ^ 1010 = 1100. Podemos realizar XOR em números inteiros convertendo primeiro o inteiro de decimal para binário. Podemos realizar XOR em strings convertendo primeiro cada caractere no inteiro que representa o caractere Unicode.

Dada a string label, realizamos XOR em cada caractere com o inteiro 13. Convertemos esses inteiros de volta para uma string e enviamos o sinalizador como crypto{new_string}."

- [Página do desafio](https://cryptohack.org/courses/intro/xor0/)

### Análise inicial
A proposta do desafio gira em torno da operação lógica XOR, também conhecida como "ou exclusivo" e apresenta a seguinte tabela:

![image](https://github.com/user-attachments/assets/2d15c678-fa8f-44ba-af85-35219bb6970e)


XOR (ou "ou exclusivo") é uma operação lógica que retorna 1 apenas quando os dois bits comparados são diferentes — ou seja, um é 1 e o outro é 0. Se ambos forem iguais (0 e 0 ou 1 e 1), o resultado é 0.

Essa operação é amplamente utilizada em áreas como eletrônica digital, criptografia e programação. Em linguagens como Python, o operador XOR é representado pelo símbolo `^`.

Por exemplo:

`1 ^ 0 = 1`

`1 ^ 1 = 0`


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

![image](https://github.com/user-attachments/assets/c8d58068-d8b4-409d-84bd-48bb210e6c6d)

Explicação das funções utilizadas:

`ord(c)`: Converte um caractere em seu valor inteiro (ASCII).

`^ 13`: Aplica a operação XOR com o número 13.

`chr(...)`: Converte o número resultante de volta em caractere.

`''.join([...])`: Junta os caracteres gerados em uma nova string.

E por fim, printamos o resultado: 

>`crypto{aloha}`

Com esta atividade, foi possível adquirir conhecimentos fundamentais na área de cibersegurança, especialmente no que diz respeito à codificação e manipulação de dados em baixo nível. Aprendemos o funcionamento da operação XOR, muito utilizada em algoritmos de criptografia simples e em técnicas de ofuscação de dados. Compreendemos como textos podem ser representados em seus valores binários ou inteiros (ASCII), e como a operação XOR pode ser aplicada para transformar ou ocultar mensagens. Além disso, foi possível entender a importância da reversibilidade em certas operações criptográficas, característica essencial em métodos de cifra simétrica. Também tivemos contato com ferramentas básicas da linguagem Python que são amplamente utilizadas em scripts de análise e quebra de criptografia, como ord(), chr() e operadores bit a bit. Esses conceitos são fundamentais para áreas como engenharia reversa, análise de malware e desenvolvimento de algoritmos de criptografia.

 
