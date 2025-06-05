# You either know, XOR you don't
###### Solved by @Mayaracar2
> This is a CTF about cryptography, xor, hexadecimal
## Desafio: You either know, XOR you don't (cryptography)
### Introdução
O desafio apresenta o seguinte enunciado:

"I've encrypted the flag with my secret key, you'll never be able to guess it.

Remember the flag format and how it might help you in this challenge!"

#### Tradução
"Eu criptografei a bandeira com minha chave secreta, você nunca será capaz de adivinhá-la.

Lembre-se do formato da bandeira e como ele pode ajudar você neste desafio!"

- [Página do desafio](https://cryptohack.org/courses/intro/xorkey1/)

### Análise inicial
A proposta do desafio gira em torno da operação lógica XOR, também conhecida como "ou exclusivo":

XOR (ou "ou exclusivo") é uma operação lógica que retorna 1 apenas quando os dois bits comparados são diferentes — ou seja, um é 1 e o outro é 0. Se ambos forem iguais (0 e 0 ou 1 e 1), o resultado é 0.

Essa operação é amplamente utilizada em áreas como eletrônica digital, criptografia e programação. Em linguagens como Python, o operador XOR é representado pelo símbolo `^`.

Por exemplo:

`1 ^ 0 = 1`

`1 ^ 1 = 0`

Inicialmente, o desafio fornece uma sequência de dados em hexadecimal:

![image](https://github.com/user-attachments/assets/ae9a57fd-2470-4edd-ac89-46f8682259c4)

## Solução
Para resolver o desafio, foi utilizado o `CyberChef`, uma ferramenta que permite manipular dados binários e realizar diversas operações de criptografia e decodificação.

- [Cyberchef](https://gchq.github.io/CyberChef/)

A primeira etapa foi colar os dados fornecidos no CyberChef e converter a string representada em hexadecimal para seus valores reais em bytes (ou caracteres).

É necessário decodificar o hexadecimal porque a operação de XOR é geralmente aplicada a bytes. Quando você tem dados criptografados ou ocultos, eles frequentemente são representados em hexadecimal ou binário, que são formas compactas de armazenar e manipular dados binários.

![image](https://github.com/user-attachments/assets/22da4002-cfab-4c3c-9731-a227acd972da)

Em seguida, utilize o formato da flag na ferramenta `XOR` e adicione codificação `UTF-8`,que define como cada caractere do texto será transformado em bytes.

Ao usar esse formato conhecido como parte da chave, podemos aplicar XOR inverso à mensagem criptografada com o texto "crypto{" no início para tentar recuperar o restante da flag. Isso é possível porque a operação XOR tem a seguinte propriedade:

`A ^ B = C  →  C ^ A = B`

Ou seja, se você souber parte do resultado esperado (como "crypto{"), você pode começar a reverter o conteúdo original. 

![image](https://github.com/user-attachments/assets/90ac37e3-746b-4397-970c-5cc16b4a7f61)

Substituindo o conteúdo da flag pelos 8 primeiros caracteres esperados (impressos no resultado do XOR), foi possível revelar todo o conteúdo criptografado:

![image](https://github.com/user-attachments/assets/f1247b25-231c-4cb0-a912-35fd4da9b6ae)

E por fim, printamos o resultado: 

>`crypto{1f_y0u_Kn0w_En0uGH_y0u_Kn0w_1t_4ll}`

Este desafio ensina conceitos fundamentais de cibersegurança ao demonstrar como cifras simples, como o uso de XOR com uma chave curta e previsível, são vulneráveis a ataques. Ele destaca a importância de reconhecer padrões, como o formato típico de uma flag (crypto{}), para quebrar criptografias fracas. Além disso, mostra que aplicar criptografia sem cuidado pode comprometer a segurança, reforçando a necessidade de boas práticas e conhecimento técnico na implementação de sistemas seguros.
