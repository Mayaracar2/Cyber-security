# RED
###### Solved by @Mayaracar2
> This is a CTF about Image metadata, Codification, File manipulation
## Desafio: RED (forensics)
### Introdução
O desafio apresenta o seguinte enunciado:VERMELHO, VERMELHO, VERMELHO, VERMELHO
Baixe a imagem: red.png

- [Página do desafio](https://play.picoctf.org/practice/challenge/460)

### Análise inicial
A princípio, nos deparamos com uma imagem de extensão `.png`.

![image](https://github.com/user-attachments/assets/7e7e2df1-6f39-4d0e-b74b-bc4c0744ce4f)

Em uma das dicas, apresenta as seguintes informações: Confira o nome atual do Facebook.

Logo, sabemos que o novo nome é "meta", insinuando que no arquivo havia `metadados`, que são dados sobre dados — ou seja, são informações que descrevem outros dados.

## Solução
No prompet de comando do kali linux, basta entrar no diretório downloads utlizando `cd Downloads/` e emitir o seguinte comando `strings red.py`, retornando o seguinte poema:

![image](https://github.com/user-attachments/assets/b18bed21-25a0-4030-af90-029500636b68)

Analisando a primeira letra de cada frase do poema, temos a seguinte frase: 
`It CHECK LSB`.

O LSB é muito usado para esconder informações secretas dentro de imagens sem que o olho humano perceba.

Além disso, usando o comando `exitftool red.py`, extraímos as seguintes informações:

![image](https://github.com/user-attachments/assets/78c41e2f-009c-4b73-98e0-f7753ff9a9f7)

Segundo essas informações, concluímos que o Color type da imagem, está em `RGB with Alpha`.
Portanto, basta usar um software pra extrair o lsb, decompondo em rgb alpha.

Para extrair as informações, basta adicionar o arquivo da imagem na página(cyberchef) e utilizar o "Extract LSB" e adicionar as cores R, G, B, A.

- [Link da página](https://gchq.github.io/CyberChef/)

![image](https://github.com/user-attachments/assets/2226b7b5-f292-4892-a031-17353569f4a1)

Portanto, é visível que as informações geradas, estão em um loop e na base 64.
Por fim, basta converter o resultado para a base 64, gerando a flag.

![image](https://github.com/user-attachments/assets/c72786ec-15f1-4f72-ab17-bf7d41d667d8)

>`picoCTF{r3d_1s_th3_ult1m4t3_cur3_f0r_54dn355_}`
 
