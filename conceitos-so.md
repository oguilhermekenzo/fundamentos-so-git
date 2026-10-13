# Conceitos de Sistemas Operacionais

## Programas, processos e threads

- **Programa:** um arquivo parado no disco, como o executável de um jogo/app ou um script. Nesse estado, ele só ocupa espaço no armazenamento.
- **Processo:** o programa **em execução**, colocado para rodar pelo SO. Ele recebe:
  - um espaço **só dele** na memória principal (RAM);
  - um **PID** (identificador do processo);
  - os **recursos** que vai usar (arquivos, sockets).
- **Thread:** uma **linha de execução** (sequência de instruções) dentro de um processo. Um processo pode ter uma ou várias threads.

Todas as threads de um processo **dividem o mesmo espaço de memória**, mas cada uma tem a sua **própria stack** e os seus **próprios registradores**. Com mais de um núcleo, elas podem rodar **em paralelo**.

### Sobre o PID

É o SO que cria o PID de cada processo. Como existe um limite de identificadores, um PID **pode ser reutilizado** por outro processo depois que o anterior termina. Rodar o mesmo programa duas vezes gera **dois processos diferentes**, cada um com seu PID e suas threads.

### Exemplo prático

Um editor de texto, como o Word:

```
Word.exe no disco          → programa (parado)
        │  duplo clique
        ▼
Word rodando na RAM        → processo (PID 4821, por exemplo)
   ├── thread: digitação
   ├── thread: corretor ortográfico
   └── thread: salvamento automático
```

### Estados de um processo

Durante a vida, um processo (ou thread) passa por estes estados:

- **Novo:** está sendo criado pelo SO.
- **Pronto:** já pode rodar e só espera a sua vez na CPU.
- **Executando:** está usando a CPU neste momento.
- **Bloqueado:** está esperando algo (leitura de disco, rede, um lock) e não adianta dar CPU para ele.
- **Finalizado:** terminou e o SO vai liberar os recursos.

```
Novo ──→ Pronto ──────────→ Executando ──→ Finalizado
           ↑   ←─ (vez acabou) ─┘  │
           │                       │ (espera E/S ou lock)
           └──── Bloqueado ←───────┘
              (o que esperava chegou)
```

Num núcleo, **só um** processo está "executando" por vez. Os outros ficam em "pronto", esperando, ou em "bloqueado". É o **escalonador** do SO que decide quem sai de "pronto" para "executando".

### Um processo na memória

```
endereços altos
┌──────────────────────┐
│  Stack               │  ← cresce "para baixo"
│  (frames das funções)│
│          ↓           │
│                      │
│   (espaço livre)     │
│                      │
│          ↑           │
│  Heap                │  ← cresce "para cima"
│  (alocação dinâmica) │
├──────────────────────┤
│  Dados globais       │  variáveis globais/estáticas
├──────────────────────┤
│  Código (text)       │  instruções do programa (somente leitura)
└──────────────────────┘
endereços baixos
```

## Heap x Stack

### Stack (pilha)

Funciona como **LIFO** (_Last In, First Out_): o último a entrar é o primeiro a sair. A cada **chamada de função**, um novo _frame_ é empilhado com:

- os **parâmetros** da função;
- o **endereço de retorno**;
- as **variáveis locais**;
- os **registradores salvos**.

Quando a função termina, o frame é desempilhado. A stack é **pequena** e muito rápida.

### Heap

Região **grande e flexível** para **alocação dinâmica**: você pede um bloco do tamanho que precisar. Os dados aqui geralmente **vivem além da função** que os criou.

As duas se conectam: variáveis na stack (ponteiros/referências) **guardam endereços** de objetos que estão na heap.

| | Stack | Heap |
|---|---|---|
| Tamanho | Pequena | Grande |
| Organização | LIFO, automática | Blocos sob demanda |
| Tempo de vida | Até a função terminar | Até ser liberado |
| Guarda | Parâmetros, locais, retorno | Dados dinâmicos |

## Concorrência x Paralelismo

- **Concorrência:** várias tarefas **em andamento** ao mesmo tempo. Mesmo com um único núcleo, o SO **intercala** entre elas.
- **Paralelismo:** várias tarefas **executando no mesmo instante**, cada uma em um núcleo diferente.

```
Concorrência (1 núcleo):   núcleo 1: A ─ B ─ A ─ C ─ B ─ A

Paralelismo (3 núcleos):   núcleo 1: A ─ A ─ A
                           núcleo 2: B ─ B ─ B
                           núcleo 3: C ─ C ─ C
```

**Todo paralelismo é concorrente, mas nem toda concorrência é paralela.**

### Context switching (troca de contexto)

É quando o SO **pausa uma thread e coloca outra para rodar**. Antes da troca, ele **salva o estado** da thread (onde ela parou, registradores). Quando ela volta, continua **exatamente de onde parou**. Nos estados vistos acima, é a passagem de "executando" para "pronto" (ou "bloqueado") de uma thread e de "pronto" para "executando" da outra.

### Race condition (condição de corrida)

Acontece quando duas ou mais threads acessam o **mesmo recurso** ao mesmo tempo e **pelo menos uma delas escreve**. O resultado passa a depender da **ordem** em que elas executam.

```
saldo = 100

Thread A: lê saldo (100)
Thread B: lê saldo (100)
Thread A: grava 100 + 50  → 150
Thread B: grava 100 - 30  → 70     ← a escrita da A se perdeu
```

**Solução:** proteger o recurso com um **lock** (ou _mutex_). A thread **bloqueia** o acesso ao entrar e **libera** ao sair. Assim, só uma thread mexe no recurso por vez.

### Deadlock (impasse)

Duas threads ficam **esperando uma pela outra para sempre**:

```
Thread A: tem o lock 1 → espera o lock 2
Thread B: tem o lock 2 → espera o lock 1
```

Repare que são os próprios **locks** que causam o deadlock. Para prevenir:

- pegar os locks **sempre na mesma ordem** em todas as threads;
- usar **timeout** ao tentar pegar um lock.

Enquanto espera o lock, cada thread fica no estado **bloqueado**. Por isso, num deadlock, nenhuma das duas volta a rodar.

## Kernel x espaço de usuário

A memória e a CPU são divididas em dois "modos":

- **Kernel (modo núcleo):** onde roda o núcleo do SO. Tem acesso **total** ao hardware (disco, rede, memória de todos os processos).
- **Espaço de usuário:** onde rodam os programas comuns (navegador, Node, Word). Eles **não acessam o hardware diretamente**.

Essa separação protege o sistema: se um programa travar ou tiver um bug, ele não derruba o SO nem mexe na memória de outro processo.

### Chamadas de sistema (syscalls)

Se o programa não pode tocar no hardware, como ele lê um arquivo? Ele **pede ao kernel** por meio de uma **syscall**:

```
Programa (espaço de usuário)
   │  "quero ler notas.txt"   → syscall read()
   ▼
Kernel
   │  verifica permissão, fala com o disco
   ▼
Programa recebe os dados
```

Exemplos de syscalls:

- **abrir/ler/escrever arquivos:** `open`, `read`, `write`;
- **criar processos:** `fork`/`exec` no Linux, `CreateProcess` no Windows;
- **pedir memória:** `mmap`, `brk` (é assim que a heap cresce).

Na prática, você quase nunca chama uma syscall direto: funções como `fs.readFile` do Node chamam por você. Enquanto espera a resposta (do disco, por exemplo), a thread costuma ficar **bloqueada**.

## Como serviços rodam no SO

Quando rodamos `node app.js` no terminal, o programa **vive enquanto o terminal estiver aberto**. Fechou o terminal, o programa morre junto.

Em produção, ele precisa **subir sozinho** quando o servidor liga, **ficar de pé** sem ninguém logado e **reiniciar** se cair. É para isso que existem os **serviços**. Um serviço:

1. **Inicia no boot**;
2. **Roda desacoplado** de qualquer terminal (em background);
3. **É supervisionado**: se cair, pode ser reiniciado automaticamente.

| SO | Nome do processo | Quem gerencia |
|---|---|---|
| Linux | _daemon_ | `systemd` |
| Windows | _Windows Service_ | Gerenciador de Serviços (`services.msc`) |

## Comandos básicos do CMD

### Arquivos e pastas

```cmd
dir                                # lista o conteúdo da pasta
dir /b                             # lista de forma mais limpa
dir /a                             # inclui arquivos ocultos
dir /s                             # inclui subpastas
cd pasta                           # entra na pasta
mkdir pasta                        # cria uma pasta
copy nul arq.txt                   # cria um arquivo vazio
copy arq.txt c:\diretorio          # copia um arquivo
move arq.txt c:\diretorio          # move um arquivo
ren novoArq.txt novoArqRename.txt  # renomeia um arquivo
del novoArqRename.txt              # apaga um arquivo
type notas.txt                     # mostra o conteúdo de um arquivo
rmdir pasta                        # apaga uma pasta vazia
rmdir /s pasta                     # apaga a pasta com tudo dentro
```

### Processos

```cmd
tasklist                           # lista os processos em execução
tasklist | findstr opera           # filtra só os processos do programa informado
taskkill /PID 4821                 # pede para encerrar o processo pelo PID
taskkill /IM opera.exe /F          # força o encerramento pelo nome
```

### Serviços

```cmd
sc query                           # lista os serviços ativos
sc query nome-do-servico           # mostra o estado de um serviço
net start nome-do-servico          # inicia um serviço (precisa de admin)
net stop nome-do-servico           # para um serviço (precisa de admin)
```

## Equivalentes no Linux

| Ação | Windows (CMD) | Linux (bash) |
|---|---|---|
| Listar arquivos | `dir` | `ls` |
| Listar com ocultos | `dir /a` | `ls -a` |
| Entrar em pasta | `cd pasta` | `cd pasta` |
| Pasta atual | `cd` | `pwd` |
| Criar pasta | `mkdir pasta` | `mkdir pasta` |
| Criar arquivo vazio | `copy nul arq.txt` | `touch arq.txt` |
| Copiar | `copy` | `cp` |
| Mover / renomear | `move` / `ren` | `mv` |
| Apagar arquivo | `del` | `rm` |
| Apagar pasta com tudo | `rmdir /s pasta` | `rm -r pasta` |
| Ver conteúdo | `type` | `cat` |
| Filtrar texto | `findstr` | `grep` |
| Listar processos | `tasklist` | `ps aux` |
| Encerrar processo | `taskkill /PID 4821` | `kill 4821` |

### Serviços com systemd

```bash
systemctl status nginx             # mostra se o serviço está rodando
sudo systemctl start nginx         # inicia o serviço
sudo systemctl stop nginx          # para o serviço
sudo systemctl restart nginx       # reinicia o serviço
sudo systemctl enable nginx        # faz o serviço subir no boot
journalctl -u nginx                # mostra os logs do serviço
```
