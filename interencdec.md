# Interencdec
###### Solved by @Mayaracar2
> This is a CTF about Cryptography
## Desafio:Interencdec (Cryptography)
### Análise inicial 
O desafio apresenta o seguinte enunciado: 

"Can you get the real meaning from this file."

#### Tradução
"Você consegue entender o significado real deste arquivo?"

- [Página do desafio](https://play.picoctf.org/practice/challenge/418)

Inicialmente, foi disponibilizado um arquivo sem extensão com o nome `enc_flag`, levando à dedução de que seu conteúdo está criptografado.

Ao converter o arquivo para `.txt` e abri-lo em um editor de texto, foi exibida a seguinte informação:

 ```bash
 YidkM0JxZGtwQlRYdHFhR3g2YUhsZmF6TnFlVGwzWVROclgya3lNRFJvYTJvMmZRPT0nCg==
 ```  
## Solução
Para identificar o método de criptografia utilizado, foi usado o Cipher Identifier:

-[Cipher Identifier](https://www.dcode.fr/cipher-identifier)

O resultado indicou que a mensagem está codificada em `base64`.

![image](https://github.com/user-attachments/assets/3fdd828b-75c4-4a98-a6ef-b55819572fd5)

Utilizando o terminal do Kali Linux, foi executado o seguinte comando para decodificar a mensagem:

```bash
echo "YidkM0JxZGtwQlRYdHFhR3g2YUhsZmF6TnFlVGwzWVROclgya3lNRFJvYTJvMmZRPT0nCg==
" | base64 -d`
```
Como a flag ainda não apareceu, concluiu-se que o desafio envolve múltiplos níveis de codificação. Por isso, o processo foi repetido com o novo conteúdo gerado, substituindo a string entre aspas a cada passo.

Na terceira decodificação com base64, observou-se que ainda não foi possível revelar a flag. Portanto, identificou-se a necessidade de mais um processo de decodificação.


![image](https://github.com/user-attachments/assets/ce90752e-a952-461b-8a4d-44861f57afa2)

O Cipher Identifier foi usado novamente para identificar a próxima técnica a ser utilizada:

![image](https://github.com/user-attachments/assets/61b6c490-d641-49c1-9be5-198aee872885)

Conforme mostrado, o método seguinte é a `Caesar Cipher`. Aplicando essa técnica, obteve-se a seguinte flag:

![image](https://github.com/user-attachments/assets/191c9967-253d-474a-8273-1214e4145ddc)

>`picoCTF{caesar_d3cr9pt3d_b204adc6}`
 
