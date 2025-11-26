# Análise Completa de Vulnerabilidade CSRF (Cross-Site Request Forgery)

## 1. Introdução e Definição
CSRF (Falsificação de Solicitação Entre Sites) é uma vulnerabilidade que induz o navegador de um usuário autenticado a executar ações indesejadas em uma aplicação confiável. O ataque explora o comportamento padrão dos navegadores de incluir automaticamente credenciais (como cookies de sessão) em requisições, permitindo que o atacante aja em nome da vítima sem o seu conhecimento.

## 2. Ciclo do Ataque e Impacto

### O Ciclo de Exploração
- Engenharia: O atacante estuda o formato da requisição da aplicação alvo.
- Armadilha: Cria um link ou script malicioso e o envia à vítima.
- Execução: A vítima, já logada no site alvo, interage com a armadilha (clique ou carregamento automático).
- Ação: O navegador envia a requisição forjada junto com os cookies de sessão, e o servidor processa a ação como legítima.

### Impacto (Severidade)
Embora geralmente não permita a leitura de dados (exfiltração), o CSRF é crítico para ações de mudança de estado:

- Transações financeiras não autorizadas.
- Alteração de credenciais (senhas, e-mails).
- Sequestro de conta (Account Takeover).

## 3. Análise da Técnica "Double Submit Cookies"
É uma defesa stateless (sem estado no servidor) popular.

- Funcionamento: O servidor envia um token pseudo-aleatório em um Cookie e o mesmo valor em um Input Hidden (ou cabeçalho).
- Validação: O servidor verifica: `Valor do Cookie == Valor do Corpo da Requisição`.

### Ponto Crítico de Falha:
Apesar de eficaz contra ataques simples, esta técnica possui vulnerabilidades fatais documentadas:

- XSS (Cross-Site Scripting): Se a aplicação tiver uma falha XSS, o atacante pode usar JavaScript para ler o token do cookie e injetá-lo na requisição forjada, anulando totalmente a proteção.
- Manipulação de Subdomínio: Se o atacante controlar um subdomínio, ele pode "setar" um cookie para o domínio pai, forçando um token que ele conhece para validar a requisição.

## 4. Vetores de Ataque

- CSRF Tradicional: Uso de formulários HTML ocultos que são submetidos automaticamente ou via engenharia social. Foca na mudança de estado (ex: POST).
- CSRF Assíncrono (XHR/Fetch): Uso de JavaScript (AJAX) para enviar requisições em segundo plano (sem recarregar a página). Comum em Single Page Applications (SPAs).
- CSRF Baseado em Flash (Legado): Explorava arquivos .swf maliciosos para enviar requisições. (Nota: Flash foi descontinuado em 2020, mas relevante para sistemas legados).

## 5. Mecanismos de Defesa e Mitigação

### A. Tokens Anti-CSRF (Padrão Ouro)
Um valor único, aleatório e imprevisível gerado pelo servidor e validado a cada requisição de mudança de estado.

### B. Double Submit Cookies (Cookies de Submissão Dupla)
Uma técnica "stateless" onde o servidor envia um valor pseudo-aleatório em um cookie e exige que esse mesmo valor seja enviado no corpo da requisição (ou cabeçalho).

- Validação: O servidor compara Cookie == Body Parameter.
- Fraquezas: Vulnerável se o atacante conseguir ler/escrever cookies (via XSS ou injeção de cookie por subdomínio) ou interceptar o tráfego (MitM).

### C. Atributo SameSite em Cookies
Instrui o navegador sobre quando enviar cookies em requisições cross-site:

- `Strict`: O cookie nunca é enviado em requisições cross-site (Proteção Máxima).
- `Lax`: Enviado apenas em navegações de topo (GET) e links, mas não em POSTs de terceiros (Equilíbrio entre segurança e usabilidade).
- `None`: Enviado em todas as requisições (Requer flag Secure).

### D. Outras Defesas

- Verificação de Cabeçalho Referer/Origin: Útil, mas pode ser burlado por ferramentas de privacidade ou spoofing.
- CAPTCHAs: Impede a automação em ações críticas.
- CSP (Content Security Policy): Mitiga a injeção de scripts maliciosos.

## 6. Estratégias de Mitigação (Red Team vs. Blue Team)
### Para Pentesters (Red Team)

- Testar endpoints: Verificar se ações sensíveis (POST/PUT/DELETE) exigem tokens.
- Analisar Tokens: Checar aleatoriedade, entropia e se o token é validado estritamente.
- Bypass: Tentar remover o token, usar token de outra sessão, ou explorar falhas de XSS para extrair tokens.
- Cenários: Testar injeção via imagens, iframes e XHR.

### Para Desenvolvedores (Blue Team / Codificação Segura)

- Tokens Obrigatórios: Implementar tokens Anti-CSRF em todos os formulários e requisições AJAX de mudança de estado.
- Configuração de Cookies: Usar SameSite=Strict ou Lax e flag Secure.
- Higiene de Sessão: Validar rigorosamente a origem (Origin/Referer) como defesa em profundidade.
- Não confie apenas no navegador: Cookies automáticos são a raiz do problema; exija prova de intencionalidade (Token).
