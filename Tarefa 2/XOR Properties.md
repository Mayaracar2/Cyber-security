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

A operação XOR (⊕) tem algumas propriedades importantes que facilitam seu uso em computação. Primeiro, ela é comutativa, ou seja, a ordem dos valores não altera o resultado (A ⊕ B = B ⊕ A). Também é associativa, o que significa que a forma de agrupar as operações não importa (A ⊕ (B ⊕ C) = (A ⊕ B) ⊕ C). O número zero funciona como elemento neutro, pois qualquer valor XOR com zero permanece igual (A ⊕ 0 = A). Por fim, XOR é seu próprio inverso, já que qualquer valor XOR consigo mesmo resulta em zero (A ⊕ A = 0). Essas propriedades tornam o XOR útil em várias aplicações, como criptografia e manipulação de dados.

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

![image](https://github.com/user-attachments/assets/c8d58068-d8b4-409d-84bd-48bb210e6c6d)

Explicação das funções utilizadas:

`ord(c)`: Converte um caractere em seu valor inteiro (ASCII).

`^ 13`: Aplica a operação XOR com o número 13.

`chr(...)`: Converte o número resultante de volta em caractere.

`''.join([...])`: Junta os caracteres gerados em uma nova string.

E por fim, printamos o resultado: 

>`crypto{aloha}`

Com esta atividade, foi possível adquirir conhecimentos fundamentais na área de cibersegurança, especialmente no que diz respeito à codificação e manipulação de dados em baixo nível. Aprendemos o funcionamento da operação XOR, muito utilizada em algoritmos de criptografia simples e em técnicas de ofuscação de dados. Compreendemos como textos podem ser representados em seus valores binários ou inteiros (ASCII), e como a operação XOR pode ser aplicada para transformar ou ocultar mensagens. Além disso, foi possível entender a importância da reversibilidade em certas operações criptográficas, característica essencial em métodos de cifra simétrica. Também tivemos contato com ferramentas básicas da linguagem Python que são amplamente utilizadas em scripts de análise e quebra de criptografia, como ord(), chr() e operadores bit a bit. Esses conceitos são fundamentais para áreas como engenharia reversa, análise de malware e desenvolvimento de algoritmos de criptografia.

 
