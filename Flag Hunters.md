# Flag Hunters
###### Solved by @Mayaracar2
> This is a CTF about Source code analysis,Hidden functions,reverse engineering
## Desafio: Flag Hunters (Reverse engineering)
### Introdução
O desafio apresenta o seguinte enunciado:

"Lyrics jump from verses to the refrain kind of like a subroutine call. There's a hidden refrain this program doesn't print by default. Can you get it to print it? There might be something in it for you.
The program's source code can be downloaded `here`.
Connect to the program with netcat:
$ nc verbal-sleep.picoctf.net 56688"

#### Tradução
"As letras saltam dos versos para o refrão, como uma chamada de subrotina. Há um refrão oculto que este programa não imprime por padrão. Você consegue fazer com que ele o imprima? Pode haver algo nele para você.
O código fonte do programa pode ser baixado `aqui` .
Conecte-se ao programa com o netcat:
$ nc verbal-sleep.picoctf.net 56688"

- [Página do desafio](https://play.picoctf.org/practice/challenge/472)

### Análise inicial
Inicialmente, foi disponibilizado um arquivo no formato `.py` e um netcat `nc verbal-sleep.picoctf.net 56688`.

O Netcat (ou nc) é uma ferramenta de linha de comando bastante versátil, usada para se comunicar diretamente com outros computadores pela rede, usando os protocolos TCP ou UDP.

## Solução
Utilizando o terminal do Kali Linux, acesse o diretório Downloads com o comando:

```bash
cd ~/Downloads
```
Em seguida, foi executado:

```bash
cat lyric-reader.py
```
Esse comando retornou o seguinte código:

```
import re
import time


# Read in flag from file
flag = open('flag.txt', 'r').read()

secret_intro = \
'''Pico warriors rising, puzzles laid bare,
Solving each challenge with precision and flair.
With unity and skill, flags we deliver,
The ether’s ours to conquer, '''\
+ flag + '\n'


song_flag_hunters = secret_intro +\
'''

[REFRAIN]
We’re flag hunters in the ether, lighting up the grid,
No puzzle too dark, no challenge too hid.
With every exploit we trigger, every byte we decrypt,
We’re chasing that victory, and we’ll never quit.
CROWD (Singalong here!);
RETURN

[VERSE1]
Command line wizards, we’re starting it right,
Spawning shells in the terminal, hacking all night.
Scripts and searches, grep through the void,
Every keystroke, we're a cypher's envoy.
Brute force the lock or craft that regex,
Flag on the horizon, what challenge is next?

REFRAIN;

Echoes in memory, packets in trace,
Digging through the remnants to uncover with haste.
Hex and headers, carving out clues,
Resurrect the hidden, it's forensics we choose.
Disk dumps and packet dumps, follow the trail,
Buried deep in the noise, but we will prevail.

REFRAIN;

Binary sorcerers, let’s tear it apart,
Disassemble the code to reveal the dark heart.
From opcode to logic, tracing each line,
Emulate and break it, this key will be mine.
Debugging the maze, and I see through the deceit,
Patch it up right, and watch the lock release.

REFRAIN;

Ciphertext tumbling, breaking the spin,
Feistel or AES, we’re destined to win.
Frequency, padding, primes on the run,
Vigenère, RSA, cracking them for fun.
Shift the letters, matrices fall,
Decrypt that flag and hear the ether call.

REFRAIN;

SQL injection, XSS flow,
Map the backend out, let the database show.
Inspecting each cookie, fiddler in the fight,
Capturing requests, push the payload just right.
HTML's secrets, backdoors unlocked,
In the world wide labyrinth, we’re never lost.

REFRAIN;

Stack's overflowing, breaking the chain,
ROP gadget wizardry, ride it to fame.
Heap spray in silence, memory's plight,
Race the condition, crash it just right.
Shellcode ready, smashing the frame,
Control the instruction, flags call my name.

REFRAIN;

END;
'''

MAX_LINES = 100

def reader(song, startLabel):
  lip = 0
  start = 0
  refrain = 0
  refrain_return = 0
  finished = False

  # Get list of lyric lines
  song_lines = song.splitlines()
  
  # Find startLabel, refrain and refrain return
  for i in range(0, len(song_lines)):
    if song_lines[i] == startLabel:
      start = i + 1
    elif song_lines[i] == '[REFRAIN]':
      refrain = i + 1
    elif song_lines[i] == 'RETURN':
      refrain_return = i

  # Print lyrics
  line_count = 0
  lip = start
  while not finished and line_count < MAX_LINES:
    line_count += 1
    for line in song_lines[lip].split(';'):
      if line == '' and song_lines[lip] != '':
        continue
      if line == 'REFRAIN':
        song_lines[refrain_return] = 'RETURN ' + str(lip + 1)
        lip = refrain
      elif re.match(r"CROWD.*", line):
        crowd = input('Crowd: ')
        song_lines[lip] = 'Crowd: ' + crowd
        lip += 1
      elif re.match(r"RETURN [0-9]+", line):
        lip = int(line.split()[1])
      elif line == 'END':
        finished = True
      else:
        print(line, flush=True)
        time.sleep(0.5)
        lip += 1



reader(song_flag_hunters, '[VERSE1]')
```
O script simula uma "leitura musical interativa", onde uma música é executada linha por linha, com controle de fluxo semelhante ao de um programa (como GOTO, RETURN, END). A flag do desafio está escondida no início do texto (em secret_intro), e só aparece se a pessoa ler o código-fonte, na qual não é impressa automaticamente.

### Partes mais importantes do código:
- `flag = open('flag.txt').read()`

Lê a flag de um arquivo. Essa flag é incluída no começo da música, mas não é exibida no terminal.

- `secret_intro + song_flag_hunters`

Cria a música completa, que tem versos, refrões e comandos especiais como:

`[REFRAIN]` → onde o refrão começa.

`REFRAIN;` → pulo para o refrão.

`RETURN` → volta ao ponto após o REFRAIN;.

`CROWD (...)` → pede input do usuário.

`END;` → encerra a execução.

- `reader(song, '[VERSE1]')`

Função que começa a “ler” a música a partir do marcador `[VERSE1]`.
Controla a execução como um interpretador: imprime linhas, lida com comandos e para no END.

Com base nessas informações sobre o código é possível prosseguir com o desafio.

Utilizando o terminal do Kali Linux, basta digitar o seguinte comando:

```bash
nc verbal-sleep.picoctf.net 56688
```
Com isso, a música começará rodar até emitir uma entrada `Crowd`

![image](https://github.com/user-attachments/assets/66baac9e-c257-4e43-8d35-3d7d2541f563)

Utilize a seguinte entrada: 
```bash
REFRAIN;RETURN 0
```
Com isso, o comando fará o `REFRAIN` da música ficar repetindo e fazendo o programa continuar sua execução de uma maneira que ele possa eventualmente alcançar a parte final onde a flag é exibida.

![image](https://github.com/user-attachments/assets/39d91534-803a-447b-805d-8f95cf85023b)

>`picoCTF{70637h3r_f0r3v3r_75053bc3}`
 
