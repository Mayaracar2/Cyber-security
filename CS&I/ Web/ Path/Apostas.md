# Path Traversal (Leitura de Arquivo)
###### Solved by @Mayaracar2
> This is a CTF about Path Traversal,Web Exploitation
## Path Traversal (Exploração Web) 
### Introdução
Explorar uma vulnerabilidade de Path Traversal em um endpoint de download para acessar arquivos fora do diretório permitido e obter a flag armazenada no servidor.

### Análise Inicial
Durante a enumeração inicial da aplicação, o arquivo `robots.txt`:

<img width="412" height="267" alt="image" src="https://github.com/user-attachments/assets/ab33b0fc-1b31-4b63-8eb7-21a90aea4acb" />

Esse arquivo revelou o seguinte endpoint interessante:

```bash
/api/download/
```

Ao investigar seu comportamento, observou-se que o endpoint aceita um parâmetro file, responsável por definir qual arquivo será retornado ao usuário.

<img width="440" height="288" alt="image" src="https://github.com/user-attachments/assets/16c874b3-269f-45a7-994b-2670957bf365" />

Isso levantou a hipótese de que o parâmetro poderia não estar devidamente sanitizado, permitindo navegação arbitrária pelo sistema de arquivos do servidor.

### Exploração
#### Passo 1 — Teste de funcionamento do endpoint

Inicialmente, foi realizado um teste simples para confirmar que o endpoint realmente serve arquivos:

```bash
api/download/?file=default.png
```

Resultado:

<img width="1035" height="399" alt="image" src="https://github.com/user-attachments/assets/b3099f43-13bd-4c78-b188-b3995998b1bd" />

Esse comportamento confirma que o endpoint está funcional e busca arquivos em algum diretório interno (provavelmente `media/`).

Isso sugere que o backend concatena diretamente o valor fornecido em `file` ao caminho base, um padrão comum em implementações vulneráveis a Path Traversal.

#### Passo 2 — Tentativa de Path Traversal

Sabendo que o endpoint serve arquivos a partir do diretório `media/`, foi testado o uso de `../` para escapar desse diretório e acessar pastas superiores.

Após algumas tentativas (incluindo brute force para identificar possíveis nomes de arquivos), foi realizada a seguinte requisição:

```bash
api/download/?file=../flags/flag7.txt
```

<img width="512" height="396" alt="image" src="https://github.com/user-attachments/assets/fdd5ce38-f599-4df0-b716-a48f55a64b6d" />

O servidor respondeu com sucesso, retornando o conteúdo do arquivo:

`FLAG{p4th_tr4v3rs4l_f1l3_r34d}`

### Conclusão
Este desafio evidenciou como a falta de validação adequada de entradas pode resultar em uma vulnerabilidade de Path Traversal, permitindo acessar arquivos fora do diretório autorizado. Ao explorar o parâmetro file, foi possível navegar pelo sistema de arquivos e obter a flag. A atividade reforça a importância de sanitização de entradas e controle de caminhos no desenvolvimento seguro de aplicações web.
