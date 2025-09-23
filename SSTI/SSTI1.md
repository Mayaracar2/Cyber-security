

# SSTI1
###### Solved by @Mayaracar2
> This is a CTF about SSTI, Web Exploitation
## Desafio: SSTI1 (Exploração Web) 
### Introdução
O desafio apresenta o seguinte enunciado: 

"I made a cool website where you can announce whatever you want! Try it out!
I heard templating is a cool and modular way to build web apps! Check out my website here!"

#### Tradução
"Criei um site bacana onde você pode anunciar o que quiser! Experimente!
Ouvi dizer que criar templates é uma maneira legal e modular de criar aplicativos web! Confira meu site aqui !"

- [Página do desafio](https://play.picoctf.org/practice/challenge/492)
### Análise inicial
O link do desafio direciona para uma aplicação web simples onde o usuário pode submeter um texto de anúncio que é renderizado pelo servidor:

- [Link do site](http://rescued-float.picoctf.net:64720/)

A tarefa consiste em verificar se a entrada do usuário é avaliada por um motor de templates (SSTI) e, se positivo, explorar essa avaliação para extrair a flag presente no sistema.

## Solução
Ao acessar a página, foi observado um formulário de input para os anúncios:

<img width="508" height="230" alt="image" src="https://github.com/user-attachments/assets/b16c95bb-8ec0-441d-9b5e-0d3c54c00c3f" />

Para testar se o campo era interpretado pelo motor de templates, foi enviada uma expressão aritmética típica de templates:

```bash
{{7*7}}
```

A aplicação retornou `49`, confirmando que o conteúdo entre `{{ ... }}` é avaliado no servidor, indício claro de Server-Side Template Injection (SSTI).

<img width="924" height="350" alt="image" src="https://github.com/user-attachments/assets/e66cb5c0-cd17-4cf1-b463-cf18d51ad1f2" />

Com a confirmação de SSTI, o atacante pode navegar pelos objetos expostos ao template para alcançar funções perigosas do ambiente Python. O payload utilizado explora a introspecção do objeto `request` para acessar `__builtins__`, importar o módulo os e executar um comando do sistema para ler o arquivo da flag:

```bash
{{request.application.__globals__.__builtins__.__import__('os').popen('cat flag').read()}}
```

Funcionou porque o motor de templates avalia expressões Python dentro de `{{ ... }}` e expõe objetos ricos ao contexto (como request). Através de `request.application.__globals__.__builtins__` o payload alcança o `builtin __import__`, importa o módulo os e usa `os.popen('cat flag').read()` para executar o comando `cat flag` no servidor e retornar o conteúdo da flag, que é então renderado na página.

Ao submeter o payload, a aplicação exibiu o conteúdo da flag:

<img width="1363" height="230" alt="image" src="https://github.com/user-attachments/assets/28ceee4e-ba0e-4eae-916f-628ac970a7f6" />


>`picoCTF{s4rv3r_s1d3_t3mp14t3_1nj3ct10n5_4r3_c001_bdc95c1a}`

## Conclusão
O desafio demonstrou uma exploração direta de SSTI: a aplicação avaliava entradas controladas pelo usuário no contexto do template e expunha objetos que permitiram escalada até execução de comandos do sistema. Isso permitiu a leitura do arquivo flag e a recuperação da bandeira.
