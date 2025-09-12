

# Forbidden Paths
###### Solved by @Mayaracar2
> This is a CTF about Path Traversal,Web Exploitation
## Desafio: Forbidden Paths (Exploração Web) 
### Introdução
O desafio apresenta o seguinte enunciado: 

"Can you get the flag?
We know that the website files live in /usr/share/nginx/html/ and the flag is at /flag.txt but the website is filtering absolute file paths. Can you get past the filter to read the flag?
Here's the website."

#### Tradução
"Você consegue pegar a bandeira?
Sabemos que os arquivos do site estão em /usr/share/nginx/html/[local /flag.txt / ...
Aqui está o site."

- [Página do desafio](https://play.picoctf.org/practice/challenge/270)
### Análise inicial
A princípio, o desafio forneceu um link que nos direciona para o seguinte site:

- [Link do site](http://saturn.picoctf.net:60543/)

 Ao analisar o código-fonte da página, não foram encontrados elementos que indicassem diretamente a localização da flag. Entretanto, o enunciado mencionava o diretório `/usr/share/nginx/html/`, indicando que o acesso a arquivos do servidor seria necessário para resolver o desafio.
## Solução
A primeira tentativa foi acessar diretamente o diretório fornecido na URL:

```bash
/usr/share/nginx/html/
```
Ao fazer isso, percebeu-se que a cor da página mudou, mas nenhum conteúdo específico foi retornado. Ao adicionar arquivos .txt conhecidos na URL, foi possível visualizar o conteúdo dos arquivos de texto, porém não era possível acessar diretamente os livros completos.

Posteriormente, ao digitar os nomes dos livros no input disponibilizado pelo site, os arquivos correspondentes eram retornados corretamente.

<img width="237" height="312" alt="image" src="https://github.com/user-attachments/assets/df9f5b29-66dc-4af9-a4c7-60587f575b07" />

Dado o nome da questão `Forbidden Paths` ficou claro que técnicas de [Path transversal](https://portswigger.net/web-security/file-path-traversal) poderiam ser aplicadas. Para isso, no campo de input, utilizou-se:

```bash
../../../flag
```
Ao clicar em `read`, a flag foi revelada:

<img width="343" height="88" alt="image" src="https://github.com/user-attachments/assets/b342d66b-d90b-45f0-b174-84d43b806c45" />

>`picoCTF{7h3_p47h_70_5ucc355_e5fe3d4d}`

## Conclusão
O desafio foi resolvido explorando a vulnerabilidade de Path Traversal, permitindo acessar o arquivo da flag fora do diretório web permitido.
