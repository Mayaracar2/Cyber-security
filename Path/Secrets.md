
# Secrets
###### Solved by @Mayaracar2
> This is a CTF about Path Traversal,Web Exploitation
## Desafio: Secrets (Exploração Web) 
### Introdução
O desafio apresenta o seguinte enunciado: 

"We have several pages hidden. Can you find the one with the flag?
The website is running here."

#### Tradução
"Temos várias páginas escondidas. Você consegue encontrar aquela com a bandeira?
O site está funcionando aqui."

- [Página do desafio](https://play.picoctf.org/practice/challenge/296,)
### Análise inicial
A princípio, o desafio forneceu um link que nos direciona para o seguinte site:

- [Link do site](http://saturn.picoctf.net:62540/)

Ao acessar a página inicial, não há informações diretas sobre a flag. Portanto, foi necessário inspecionar o código-fonte da aplicação para buscar pistas.

## Solução
Para prosseguir, inspecione o código-fonte da página principal utilizando `Ctrl + U`.

```bash
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8" />
    <meta
      name="viewport"
      content="width=device-width, initial-scale=1, shrink-to-fit=no"
    />
    <meta name="description" content="" />
    <!-- Bootstrap core CSS -->
    <link href="vendor/bootstrap/css/bootstrap.min.css" rel="stylesheet" />
    <!-- title -->
    <title>home</title>
    <!-- css -->
    <link href="secret/assets/index.css" rel="stylesheet" />
  </head>
  <body>
    <!-- ***** Header Area Start ***** -->
    <div class="topnav">
      <a class="active" href="#home">Home</a>
      <a href="about.html">About</a>
      <a href="contact.html">Contact</a>
    </div>

    <div class="imgcontainer">
      <img
        src="secret/assets/DX1KYM.jpg"
        alt="https://www.alamy.com/security-safety-word-cloud-concept-image-image67649784.html"
        class="responsive"
      />
      <div class="top-left">
        <h1>If security wasn't your job, would you do it as a hobby?</h1>
      </div>
    </div>
  </body>
</html>
```

Ao analisar, observou-se que havia uma referência a um diretório chamado `secret`:

```bash
  <link href="secret/assets/index.css" rel="stylesheet" />
```

Ao acessar o diretório `/secret/`, foi exibida a seguinte imagem:

<img width="488" height="604" alt="image" src="https://github.com/user-attachments/assets/0db9b7d2-76ad-464b-9338-731520befaff" />

Ao inspecionar o novo diretório, foi descoberto um novo diretório com uma nova referência para `hidden`:

```bash
 <link rel="stylesheet" href="hidden/file.css" />
```
 Acessando `/hidden/`, a página retornou um formulário de login:

<img width="1365" height="548" alt="image" src="https://github.com/user-attachments/assets/7270a7e4-6e64-45a3-b58c-bc588e6804ac" />

Ao repetir o processo de análise, foi identificado mais um caminho chamado `superhidden`:

```bash
<link href="superhidden/login.css" rel="stylesheet" />
```
Ao visitar `/superhidden/`, a página exibiu a seguinte frase:

<img width="1022" height="302" alt="image" src="https://github.com/user-attachments/assets/48caf913-b710-401d-94dc-e7b553ed8bd5" />

Isso sugeriu que o conteúdo verdadeiro não estava visível na interface, mas oculto no código.

Inspecionando o código-fonte dessa última página, foi possível identificar a flag dentro de uma tag HTML:

```bash
<h3 class="flag">picoCTF{succ3ss_@h3n1c@10n_790d2615}</h3>
```

>`picoCTF{succ3ss_@h3n1c@10n_790d2615}`

## Conclusão
A questão demonstrou a importância de analisar cuidadosamente os recursos externos referenciados em páginas web, já que muitas vezes os diretórios e arquivos escondidos podem conter informações críticas ou a própria solução do desafio.
 
