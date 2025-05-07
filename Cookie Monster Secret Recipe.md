# Cookie Monster Secret Recipe
###### Solved by @Mayaracar2
> This is a CTF about cookies,Web Exploitation
## Desafio: Cookie Monster Secret Recipe (Exploração Web) 
### Introdução
O desafio apresenta o seguinte enunciado: "O Monstro dos Come-Come escondeu sua receita secreta de biscoitos em algum lugar do seu site. Como um aspirante a detetive de biscoitos, sua missão é desvendar esse segredo delicioso. Você consegue enganar o Monstro dos Come-Come e encontrar a receita escondida?"

- [Página do desafio](https://play.picoctf.org/practice/challenge/469)
### Análise inicial
A princípio, nos deparamos com um link que nos direciona para o seguinte site:

- [Link do site](http://verbal-sleep.picoctf.net:51173/)

Inicialmente, qualquer usuário ou senha que inserirmos, retornaria um acesso negado.

![image](https://github.com/user-attachments/assets/1fb4ea4b-6605-40bd-ad1d-a4c02cb38115)

Portanto, ao gerar um acesso negado, ele nos da uma dica como: 
"O Monstro dos Come-Come diz: 'Eu não preciso de senha. Eu só preciso de cookies!'
Dica: Você verificou seus cookies ultimamente?".
Com isso, ele nos direciona a inspecionar os cookies da página.
## Solução
Para concluir o desafio, devemos clicar com o botão direito do mouse e selecionar "Inspecionar", direcionando para a seguinte tela:

![image](https://github.com/user-attachments/assets/593feb3b-1371-4f54-9b46-6b8d15e009d6)

Logo, devemos clicar na função "Application", depois "Cookies" para que ele exiba o URL que apresenta o cookie:

![image](https://github.com/user-attachments/assets/c02079e8-4e37-4270-a30e-d08b237ed8c4)

Ao clicar em "secret_recip", ele irá mostrar na parte inferior a seguinte mensagem encriptografada `cGljb0NURntjMDBrMWVfbTBuc3Rlcl9sMHZlc19jMDBraWVzXzA1N0JDQjUxfQ==`

Logo, iremos usar o seguinte site para poder indentificar como a mensagem foi encriptografada:

- [Cipher Identifier](https://www.dcode.fr/cipher-identifier)

Ao ser analisada, ela retorna os seguintes resultados: 

![image](https://github.com/user-attachments/assets/e06718cd-033a-4c25-a5fc-8b93a62e7690)

Deixando claro que a `base64` é predominante e ao decriptografar, gerou por fim a flag do desafio.

>`picoCTF{c00k1e_m0nster_l0ves_c00kies_057BCB51}`
 
