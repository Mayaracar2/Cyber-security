
# Crack
###### Solved by @Mayaracar2
> This is a CTF about dynamic analysis
## Desafio: Crack (Reverse engineering) 
### Análise inicial
A princípio, o desafio forneceu um arquivo nomeado como crack denominado como engenharia reversa, para entender melhor sobre o arquivo, foi executado o seguinte comando:

`file crack`

<img width="661" height="113" alt="image" src="https://github.com/user-attachments/assets/c6c0d4e8-adfc-499e-8055-73aabf2552a7" />

Com isso é possível ver que o arquivo é um executavel vinculado dinamicamente, isso significa que ele não contém todo o código de que precisa para rodar e que precisa de um input.

Para dar acesso para ele ser executado, utilize :

`chmod -X crack`

Em seguida rode o arquivo com:

`./crack`

Com isso, ele pede a entrada de uma senha e caso ela esteja incorreta, ele retorna "Bad Kitty!"

<img width="273" height="108" alt="image" src="https://github.com/user-attachments/assets/fb164130-d676-4386-b3fe-0c325a11643b" />

## Solução
Para prosseguir, utilize o gdb e utilize o seguinte comando:

`gdb ./crack`

Definir um breakpoint na main: Para garantir que o programa pare no início, antes de qualquer execução:

`(gdb) break main`

Executar o programa:

`(gdb) run`

Definir o breakpoint na função de impressão: Como não sabíamos onde a string "Bad kitty!" estava, definimos um breakpoint na função que a imprimiria, a puts.

`(gdb) break puts`

Continuar a execução e fornecer um input falso:

`(gdb) continue`

Quando o programa pediu a senha (enter the right password), digitamos uma senha falsa e apertamos Enter.

Sair da função puts: O GDB parou dentro da função puts. Para ver quem chamou a puts, usamos o comando finish.

`(gdb) finish`

Isso terminou a execução da puts e nos deixou parados na instrução exatamente seguinte à call puts, já no código "ruim" da main.

Com isso, precisamos achar onde faz a valição de senha

`(gdb) x/15i $rip - 40` ou Use `disassemble` para ver a função completa.

<img width="663" height="283" alt="image" src="https://github.com/user-attachments/assets/69d2bab8-a5f9-44ba-8161-90da210a0ab5" />

Isso nos disse que a verificação principal acontece no endereço `0x0000555555555555`.

Para extrair a Senha Correta,Descobrimos que o programa compara %bl (que continha o caractere correto) com 0x60(%rsp...) (que continha nosso input). No disassembly completo, vimos que o caractere correto era carregado para %ebx a partir de 0x10(%rsp...).

Reiniciar o programa:

`(gdb) run`

Definir o breakpoint no "Cofre": Removemos os breakpoints antigos e colocamos um novo, exatamente na instrução cmp.

`(gdb) delete`

`(gdb) break *0x0000555555555555`

Continuar e fornecer um input falso (de 8 caracteres): Pela análise anterior do código, sabíamos que a senha tinha 8 caracteres.

`(gdb) continue`

(O GDB parou na main)

`(gdb) continue`

Ler a Senha da Memória (A Chave Mestra): O programa parou no nosso breakpoint (cmp), pronto para comparar o primeiro caractere. Neste exato momento, a senha correta inteira estava na memória. Lemos os 8 bytes (/8cb) a partir do local de origem ($rsp+0x10).

`(gdb) x/8cb $rsp+0x10`

<img width="651" height="57" alt="image" src="https://github.com/user-attachments/assets/4ce8dec3-7432-4484-8f86-795a5580a3fd" />

O GDB imprimiu os 8 bytes da senha correta: `0x7fffffffdca0: 48 '0' 48 '0' 115 's' 71 'G' 111 'o' 52 '4' 77 'M' 48 '0'`

Transcrevendo os caracteres, encontramos a senha: `00sGo4M0`

Após executar o arquivo novamente e inserir a senha correta, irá retornar "Good Kitty!":

<img width="269" height="107" alt="image" src="https://github.com/user-attachments/assets/4cdfbdf9-3077-4dd3-88ff-c49f99dcac78" />

>`00sGo4M0`

### Conclusão
Em conclusão, este desafio de engenharia reversa foi resolvido através da análise dinâmica com o GDB. A estratégia principal foi identificar a lógica de verificação da senha, primeiro localizando a saída de erro e depois inspecionando o código que levava a ela. Ao definir um breakpoint na instrução de comparação, foi possível ler a senha correta diretamente da memória antes que ela fosse comparada, resultando na flag 00sGo4M0.

 
