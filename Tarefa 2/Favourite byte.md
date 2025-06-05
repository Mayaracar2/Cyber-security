# Favourite byte
###### Solved by @Mayaracar2
> This is a CTF about cryptography, xor, hexadecimal
## Desafio: Favourite byte (cryptography)
### Introdução
O desafio apresenta o seguinte enunciado:

"I've hidden some data using XOR with a single byte, but that byte is a secret. Don't forget to decode from hex first."

#### Tradução
"Escondi alguns dados usando XOR com um único byte, mas esse byte é secreto. Não se esqueça de decodificar a partir do hexadecimal primeiro."

- [Página do desafio](https://cryptohack.org/courses/intro/xorkey0/)

### Análise inicial
A proposta do desafio gira em torno da operação lógica XOR, também conhecida como "ou exclusivo":

XOR (ou "ou exclusivo") é uma operação lógica que retorna 1 apenas quando os dois bits comparados são diferentes — ou seja, um é 1 e o outro é 0. Se ambos forem iguais (0 e 0 ou 1 e 1), o resultado é 0.

Essa operação é amplamente utilizada em áreas como eletrônica digital, criptografia e programação. Em linguagens como Python, o operador XOR é representado pelo símbolo `^`.

Por exemplo:

`1 ^ 0 = 1`

`1 ^ 1 = 0`

A princípio, foi apresentado a seguinte informação:

![image](https://github.com/user-attachments/assets/e165e1a4-d2de-46a3-9f62-09796a356fc5)

## Solução
Para resolver o desafio, utilize o `cyberchef`.

- [Cyberchef](https://gchq.github.io/CyberChef/)

Primeiro cole a informação que foi fornecida na página e converta para `hexadecimal`.

É necessário converter para hexadecimal porque a operação de XOR é geralmente aplicada a bytes. Quando você tem dados criptografados ou ocultos, eles frequentemente são representados em hexadecimal ou binário, que são formas compactas de armazenar e manipular dados binários.
  
![image](https://github.com/user-attachments/assets/b102644c-e43a-4d93-bb09-7a0b9fcdaeb5)

Em seguida, utilize o `xor brute force` para encontrar qual é o byte secreto.

Quando você usa XOR brute force, você está tentando descobrir a chave que foi usada para codificar os dados com a operação XOR, testando todas as possibilidades possíveis de chave.

No caso de um XOR com um único byte, existem apenas 256 possibilidades (de 0 a 255), então é viável testar todas elas.

![image](https://github.com/user-attachments/assets/71cc4fa5-85de-482e-994c-27ed9693b983)

Analiando as chaves que foram geradas, é possivel encontrar a flag na `chave 10`.

![image](https://github.com/user-attachments/assets/8d6d6a26-5f72-4aed-aaee-28695defccd4)

E por fim, printamos o resultado: 

>`crypto{0x10_15_my_f4v0ur173_by7e}`

Essa questão ensina lições valiosas sobre cibersegurança. Mostra que usar XOR com uma chave curta (como um único byte) é inseguro, pois pode ser quebrado facilmente por força bruta, também destaca como padrões previsíveis como o formato de uma flag ajudam na quebra de cifras fracas. Além disso, reforça que não basta usar técnicas de criptografia: a forma como elas são implementadas faz toda a diferença. Por fim, demonstra a importância da prática e da automação em segurança, habilidades essenciais para quem deseja atuar como analista, pentester ou profissional de defesa digital.










