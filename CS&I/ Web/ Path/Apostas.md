#Path Traversal (Leitura de Arquivo)
📌 Informações Gerais

Categoria: Path Traversal

Dificuldade: Fácil

Flag: FLAG{p4th_tr4v3rs4l_f1l3_r34d}

🎯 Objetivo

Explorar uma vulnerabilidade de Path Traversal em um endpoint de download para acessar arquivos fora do diretório permitido e obter a flag armazenada no servidor.

🔍 Análise Inicial

Durante a enumeração inicial da aplicação, o arquivo robots.txt revelou o seguinte endpoint interessante:

/api/download/


Ao analisar seu comportamento, foi identificado que esse endpoint aceita um parâmetro file, responsável por definir qual arquivo será servido ao usuário.

Isso levanta uma suspeita clássica:
👉 o parâmetro pode não estar devidamente sanitizado, permitindo navegação arbitrária no sistema de arquivos.

🧪 Exploração
Passo 1 — Teste de funcionamento do endpoint

Primeiro, foi feito um teste básico para confirmar que o endpoint realmente tenta servir arquivos:

curl "http://localhost:8000/api/download/?file=default.png"


Resultado:

Retorno 404 — arquivo não encontrado

Confirma que o endpoint está funcional e busca arquivos em algum diretório interno (provavelmente media/)

Esse comportamento indica que o backend concatena diretamente o valor de file ao caminho base.

Passo 2 — Tentativa de Path Traversal

Sabendo que o endpoint serve arquivos a partir do diretório media/, foi testado o uso de ../ para escapar desse diretório e acessar pastas superiores.

curl "http://localhost:8000/api/download/?file=../flags/flag7.txt"

✅ Resultado

O servidor respondeu com o conteúdo do arquivo:

Parabens! Voce encontrou esta flag via Path Traversal!

FLAG{p4th_tr4v3rs4l_f1l3_r34d}
