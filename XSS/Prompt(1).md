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

