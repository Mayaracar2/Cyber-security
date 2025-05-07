# Cookie Monster Secret Recipe
###### Solved by @Mayaracar2
> This is a CTF about cookies,Web Exploitation
## Desafio: Cookie Monster Secret Recipe (Exploração Web) 
### Introdução
O desafio apresenta o seguinte enunciado: 

"O Monstro dos Come-Come escondeu sua receita secreta de biscoitos em algum lugar do seu site. Como um aspirante a detetive de biscoitos, sua missão é desvendar esse segredo delicioso. Você consegue enganar o Monstro dos Come-Come e encontrar a receita escondida?"

- [Página do desafio](https://play.picoctf.org/practice/challenge/469)
### Análise inicial
A princípio, o desafio forneceu um link que nos direciona para o seguinte site:

- [Link do site](http://verbal-sleep.picoctf.net:51173/)

Na página inicial, qualquer combinação de nome de usuário e senha resultava em um acesso negado.

![image](https://github.com/user-attachments/assets/1fb4ea4b-6605-40bd-ad1d-a4c02cb38115)

Ao negar o acesso, o próprio site fornecia uma pista:
"O Monstro dos Come-Come diz: 'Eu não preciso de senha. Eu só preciso de cookies!'
Dica: Você verificou seus cookies ultimamente?"

Com essa dica, tornou-se evidente que a análise deveria se concentrar nos cookies da página.
## Solução
Para prosseguir, abra as ferramentas de desenvolvedor no navegador com um clique no botão direito do mouse e selecione a opção `"Inspecionar"`.

![image](https://github.com/user-attachments/assets/593feb3b-1371-4f54-9b46-6b8d15e009d6)

Na aba `"Application"`, é possível acessar a seção Cookies, onde aparece o domínio da página com um cookie chamado `secret_recipe`.

![image](https://github.com/user-attachments/assets/c02079e8-4e37-4270-a30e-d08b237ed8c4)

Ao clicar em "secret_recip", ele irá mostrar na parte inferior a seguinte mensagem encriptografada:

```bash
cGljb0NURntjMDBrMWVfbTBuc3Rlcl9sMHZlc19jMDBraWVzXzA1N0JDQjUxfQ==
```

Para descobrir o tipo de codificação utilizada, utilize o `cipher-identifier`

- [Cipher Identifier](https://www.dcode.fr/cipher-identifier)

A análise apontou com clareza que se tratava de uma codificação Base64.

![image](https://github.com/user-attachments/assets/e06718cd-033a-4c25-a5fc-8b93a62e7690)

Após a decodificação, a flag foi revelada com sucesso:

>`picoCTF{c00k1e_m0nster_l0ves_c00kies_057BCB51}`
 
