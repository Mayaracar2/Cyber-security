# x86 Assembly Crash Course
###### Written by @Mayaracar2
>This is a crash course in x86 Assembly for training in malware reverse engineering.
## Propósito deste Guia
A linguagem Assembly representa a forma mais fundamental de comunicação com o hardware de um computador. Embora seja uma linguagem de baixo nível, ela é, paradoxalmente, a representação de mais alto nível que podemos obter de forma confiável ao descompilar um arquivo binário.

Para a análise de malware, lidamos quase exclusivamente com binários compilados. Entender o seu funcionamento exige reverter esse processo. Embora a descompilação possa perder informações cruciais, a análise do código Assembly nos dá a visão mais precisa do que o programa realmente faz.

Este material abordará os seguintes pilares:

* Opcodes e Operandos
* Instruções Comuns de Montagem
* Operações Aritméticas e Lógicas
* Comparações e Condicionais
* Controle de Fluxo e Ramificações

## Opcodes e Operandos
A CPU executa instruções em formato binário (sequências de 1s e 0s). Para facilitar a leitura humana, agrupamos esses bits em bytes, comumente representados em hexadecimal.

Para um analista, as instruções que o computador executa apareceriam como uma sequência de números hexadecimais, e essas sequências são compostas por opcodes e operandos.

### Opcodes
Cada arquitetura de processador possui um conjunto de instruções único. A linguagem Assembly é a representação textual e legível dessas instruções. Um desmontador (disassembler) traduz os opcodes (códigos de operação) binários de volta para instruções Assembly legíveis.

Considere a instrução mov eax, 0x5f, em um desmontador, ela pode aparecer assim: 40000: b8 5f 00 00 00  mov eax, 0x5f

* 040000: é o endereço de memória da instrução.
* b8 é o opcode hexadecimal que significa "mover um valor imediato para o registrador EAX".
* 5F 00 00 00 é a representação em little-endian do valor 0x5f que serve como o outro operando.

### Tipos de Operandos
Existem três categorias principais de operandos em Assembly:

* Operandos Imediatos: Valores fixos e constantes definidos diretamente na instrução (ex: 0x5f).
* Operandos de Registrador: Utilizam os registradores da CPU (ex: eax, ebx) como fonte ou destino de dados.
* Operandos de Memória: Referenciam um local na memória. São identificados pelo uso de colchetes [].

## Instruções Gerais de Montagem
Instruções são os comandos que a CPU executa. Elas manipulam dados (operandos) de registradores, memória ou valores imediatos, e armazenam o resultado em um registrador ou na memória.

### Instrução MOV
A instrução MOV (move) copia dados de uma origem para um destino. 

Sintaxe: mov destino, origem

* mov eax, 0x5f: Coloca o valor imediato 0x5f dentro do registrador eax.
* mov ebx, eax: Copia o conteúdo de eax para dentro de ebx.
* mov eax, [0x5fc53e]: Copia o valor que está armazenado no endereço de memória 0x5fc53e para dentro de eax.

### Instrução LEA
A instrução LEA (Load Effective Address) calcula um endereço de memória e o armazena no destino, sem acessar o conteúdo desse endereço. 

Exemplo: lea eax, [ebp+4]

Neste caso, eax receberá o resultado da soma ebp+4, e não o dado que está guardado no endereço de memória resultante. LEA é frequentemente usada como uma forma rápida de realizar operações aritméticas.

### Instrução NOP
A NOP (No Operation) é uma instrução nula, no contexto de malware, ajudam a garantir que um shellcode seja executado mesmo que o controle de fluxo não salte exatamente para o seu início.

### Instrução Shift (Deslocamento)
As instruções de deslocamento (shift) movem os bits em um registrador para a esquerda (shl) ou direita (shr). São uma forma eficiente de multiplicar ou dividir por potências de 2.

* shr destino, count ; desloca bits para a direita
* shl destino, count ; desloca bits para a esquerda

Count define o número de posições a deslocar. Zeros são inseridos no lado oposto. O último bit que "cai" para fora do registrador é armazenado na Carry Flag (CF).

### Instrução Rotate (Rotação)
Similar ao shift, mas os bits que "caem" de um lado são reinseridos no outro, em vez de serem descartados.

* ror destino, count ; rotaciona para a direita
* rol destino, count ; rotaciona para a esquerda

## Bandeiras
A CPU possui um registrador de status, o EFLAGS, que armazena o estado atual do processador. Ele é composto por bits individuais (flags) que são atualizados após operações lógicas ou aritméticas.

As flags são essenciais para o controle de fluxo (como saltos condicionais).

Flags Comuns:

<img width="550" height="317" alt="image" src="https://github.com/user-attachments/assets/cb776d07-21a4-41be-b06b-a459e8fff65a" />

## Operações Aritméticas
Instruções que realizam cálculos matemáticos.

### Adição e Subtração

* add destino, valor: Soma valor ao destino. Resultado fica em destino.
* sub destino, valor: Subtrai valor de destino. Resultado fica em destino.

Estas instruções afetam flags importantes. ZF é ativada se sub resultar em zero. CF é ativada se sub precisar "emprestar" (destino < valor).

### Multiplicação e Divisão
Estas operações são mais complexas, pois o resultado pode exceder 32 bits, usando registradores edx e eax em conjunto.

* mul valor: Multiplica eax por valor. O resultado de 64 bits é armazenado em edx:eax (a parte alta em edx, a baixa em eax).
* div valor: Divide o valor de 64 bits em edx:eax por valor. O quociente fica em eax e o resto (módulo) fica em edx.

### ncremento e Decremento
Formas rápidas de somar ou subtrair 1.

* inc eax: Equivalente a add eax, 1.
* dec eax: Equivalente a sub eax, 1.

## Instruções Lógicas
Realizam operações booleanas em nível de bit (bit-a-bit).

* AND: Retorna 1 apenas se ambos os bits de entrada forem 1.
* OR: Retorna 1 se pelo menos um dos bits de entrada for 1.
* NOT: Inverte todos os bits do operando (0 vira 1, 1 vira 0).
* XOR: Retorna 1 apenas se os bits de entrada forem diferentes.

## Condicionais e Ramificações
Para tomar decisões, a CPU usa instruções que comparam valores e alteram o fluxo de execução.

### Instrução TEST
Executa uma operação AND lógica entre os operandos, mas não armazena o resultado. Seu único propósito é atualizar as flags (especialmente a ZF).

* test destino, fonte: Se o resultado do AND for zero, ZF é definida como 1. Muito usado para verificar se um bit específico está ativado.

### Instrução CMP
Compara (compare) dois valores, também sem alterar os operandos. Funciona internamente como uma subtração (destino - fonte) cujo resultado é descartado, atualizando apenas as flags.

`cmp destino, fonte`

* Se destino == fonte, o resultado da subtração é 0, e ZF = 1.
* Se destino < fonte, ocorre um "empréstimo", e CF = 1.
* Se destino > fonte, ZF = 0 e CF = 0.

### Ramificação (Branching)
Normalmente, o ponteiro de instrução (EIP) avança linearmente pela memória. Uma ramificação altera o EIP, fazendo o fluxo de execução "saltar" para outro local.

### Instrução JMP
O JMP (jump) é um salto incondicional. Ele simplesmente move o fluxo de controle para o endereço de destino especificado.

Sintaxe: jmp destino

### Saltos Condicionais
Estes são os equivalentes a if/else em Assembly. Eles executam o salto apenas se uma condição, baseada nas flags, for verdadeira. Quase sempre são usados após uma instrução CMP ou TEST.

<img width="480" height="371" alt="image" src="https://github.com/user-attachments/assets/51012411-041e-42e3-9450-f09de342a123" />

## A Pilha (Stack)
A pilha é uma estrutura de dados fundamental do tipo LIFO (Last-In, First-Out). O último item colocado na pilha é o primeiro a ser removido. O registrador ESP (Extended Stack Pointer) aponta para o topo atual da pilha.

### Instrução PUSH
A instrução push "empilha" um valor. 

Sintaxe: push fonte

Esta operação faz duas coisas:

* O valor da fonte é escrito no endereço de memória apontado por ESP.
* O ESP é decrementado (pois a pilha em x86 cresce "para baixo", em direção a endereços de memória menores).


