
# Crack
###### Solved by @Mayaracar2
> This is a CTF about dynamic analysis
## Desafio: Crack (Reverse engineering) 
### Análise inicial
AO desafio fornecia um arquivo executável denominado crack, classificado como um exercício de engenharia reversa. Para identificar o tipo de arquivo, foi utilizado o comando:

```bash
file crack
```

<img width="661" height="113" alt="image" src="https://github.com/user-attachments/assets/c6c0d4e8-adfc-499e-8055-73aabf2552a7" />

A saída revelou que se tratava de um executável dinamicamente vinculado, o que indica que ele depende de bibliotecas externas e requer um input para sua execução correta.

Para conceder permissão de execução, foi executado:

```bash
chmod +x crack
```

Em seguida, o binário foi iniciado com:

```bash 
./crack
```
O programa solicitava a entrada de uma senha e, quando uma senha incorreta era fornecida, exibia a mensagem `"Bad Kitty!"`.

<img width="273" height="108" alt="image" src="https://github.com/user-attachments/assets/fb164130-d676-4386-b3fe-0c325a11643b" />

## Solução
Para prosseguir com a análise, o binário foi aberto no GDB com o comando:

```bash
gdb ./crack
```

Primeiramente, foi definido um breakpoint na `função main`, garantindo que a execução fosse interrompida logo no início:

```bash
(gdb) break main
(gdb) run
```
Como a localização da string "Bad Kitty!" ainda era desconhecida, definiu-se um novo breakpoint na função puts, responsável por imprimir mensagens na tela:

```break
(gdb) break puts
(gdb) continue
```
Após fornecer uma senha incorreta e pressionar Enter, o programa parou dentro da `função puts`. Para verificar quem havia chamado essa função, foi utilizado o comando:

```bash
(gdb) finish
```
Esse comando concluiu a execução de puts e retornou o controle ao ponto seguinte à sua chamada na função main, onde ocorria a validação da senha.

Para inspecionar o código responsável pela comparação da senha, foi utilizado:

```bash
(gdb) x/15i $rip - 40
```
ou, alternativamente,

```bash
(gdb) disassemble
```

<img width="663" height="283" alt="image" src="https://github.com/user-attachments/assets/69d2bab8-a5f9-44ba-8161-90da210a0ab5" />

A análise revelou que a instrução de comparação principal estava localizada no endereço `0x0000555555555555`.

O binário carrega o caractere esperado em `%ebx` e compara apenas o byte baixo `(%bl)` com o byte correspondente do input localizado na pilha `(0x60(%rsp))`. O valor do caractere correto estava armazenado previamente em `0x10(%rsp)` e foi carregado para `%ebx` antes da comparação. A instrução cmp entre %bl e 0x60(%rsp) é seguida por um salto condicional (jmp para o caminho de erro): se os bytes diferirem, o programa salta para a rotina que imprime "Bad Kitty!"; se forem iguais, continua para a próxima comparação.

Para extrair a senha correta, o programa foi reiniciado:

```bash
(gdb) run
```

Os breakpoints anteriores foram removidos e um novo foi definido exatamente na instrução de comparação (cmp):

```bash
(gdb) delete
(gdb) break *0x0000555555555555
```

Em seguida, o programa foi executado novamente com uma senha falsa de 8 caracteres, já que a análise do código indicava que a senha possuía esse tamanho:

```bash
(gdb) continue
```

No momento em que o programa parou no breakpoint, a senha correta estava armazenada na memória. Para visualizá-la, foi utilizado:

```bash
(gdb) x/8cb $rsp+0x10
```

<img width="651" height="57" alt="image" src="https://github.com/user-attachments/assets/4ce8dec3-7432-4484-8f86-795a5580a3fd" />

O comando exibiu os `8 bytes` correspondentes à senha correta:

 ```bash
 0x7fffffffdca0: 48 '0' 48 '0' 115 's' 71 'G' 111 'o' 52 '4' 77 'M' 48 '0'
```

Convertendo os valores, obteve-se a senha:

```bash
00sGo4M0
```

Ao executar novamente o binário e inserir a senha correta, o programa exibiu a mensagem `"Good Kitty!"`, confirmando o sucesso da validação.

<img width="269" height="107" alt="image" src="https://github.com/user-attachments/assets/4cdfbdf9-3077-4dd3-88ff-c49f99dcac78" />

>`00sGo4M0`

### Conclusão
O desafio foi resolvido por meio de análise dinâmica utilizando o GDB. A abordagem adotada consistiu em rastrear o fluxo do programa até a rotina de verificação da senha, iniciando pela interceptação da função puts e avançando até o ponto de comparação dos caracteres.

Ao identificar a instrução responsável pela validação, foi possível ler diretamente da memória o conteúdo correspondente à senha correta, resultando na flag `00sGo4M0`.

 
