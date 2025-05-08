# RED
###### Solved by @Mayaracar2
> This is a CTF about Image metadata, Codification, File manipulation
## Desafio: RED (forensics)
### Introdução
O desafio apresenta o seguinte enunciado:

"RED, RED, RED, RED

Download the image: `red.png`"

#### Tradução
"VERMELHO, VERMELHO, VERMELHO, VERMELHO

Baixe a imagem: `red.png`"

- [Página do desafio](https://play.picoctf.org/practice/challenge/460)

### Análise inicial
Inicialmente, foi disponibilizada uma imagem no formato `.png`:

![image](https://github.com/user-attachments/assets/7e7e2df1-6f39-4d0e-b74b-bc4c0744ce4f)

## Solução
Como boa prática em desafios forenses, o primeiro passo foi verificar a presença de metadados ocultos na imagem.

Utilizando o terminal do Kali Linux, acesse o diretório Downloads com o comando:

```bash
cd ~/Downloads
```
Em seguida, foi executado:

```bash
strings red.py
```
Esse comando retornou o seguinte poema:

![image](https://github.com/user-attachments/assets/b18bed21-25a0-4030-af90-029500636b68)

Ao analisar a primeira letra de cada verso, formou-se a mensagem:
`It CHECK LSB`.

A sigla LSB (Least Significant Bit) refere-se a uma técnica comumente usada para esconder informações dentro de imagens, de forma que o olho humano não perceba.

Adicionalmente, com o comando:

```bash
exiftool red.py
```

Foram extraídas informações dos metadados da imagem:

![image](https://github.com/user-attachments/assets/78c41e2f-009c-4b73-98e0-f7753ff9a9f7)

Conforme indicado, o Color Type da imagem é `RGB with Alpha`, o que sugere a decomposição em quatro canais: Vermelho, Verde, Azul e Alfa.

Para extrair as informações escondidas, utilize o site CyberChef, adicione o arquivo da imagem e execute com a função "Extract LSB", selecionando os canais R, G, B e A:

- [Link da página](https://gchq.github.io/CyberChef/)

![image](https://github.com/user-attachments/assets/2226b7b5-f292-4892-a031-17353569f4a1)

As informações extraídas estavam em formato `base64` e repetidas em um loop.

Decodificando esse conteúdo base64, foi possível obter a flag final:

![image](https://github.com/user-attachments/assets/c72786ec-15f1-4f72-ab17-bf7d41d667d8)

>`picoCTF{r3d_1s_th3_ult1m4t3_cur3_f0r_54dn355_}`
 
