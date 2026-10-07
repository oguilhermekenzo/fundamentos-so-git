# Explicando conceitos do sistema operacional

## Programas, processos e Threads

Programa é um arquivo parado no disco, como um executavel de jogo/app ou um script em alguma linguagem. Neste momento, ele está apenas ocupando espaço do armazenamento. 

Quando falamos de Processo, estamos falando desse programa em estado de execução, executado pelo SO. O processo recebe um espaço na memória principal (RAM) só para ele, ele recebe também um PID (seu identificador) e os recursos que ele vai usar (arquivos, sockets).

Já uma Thread é uma linha de execução (sequência de instruções) dentro de um processo. O processo pode ter uma ou várias threads rodando em paralelo. Todas as threads dividem o mesmom espaço de memória do processo, e cada uma tem sua própria pilha de execução, a stack e registradores. 

É o SO que cria o identificador de cada processo, tendo um limite, o que faz acontecer que um PID pode ser reutilizado em um outro processo. Quando um programa roda duas vezes, são dois processos diferentes com suas threads e seus identificadores. 

Um exemplo prático seria um editor de texto, como o Word. O programa está parado na memória de armazenamento, mas quando seu executável é acionado então o SO coloca este programa para rodar e ele vira um processo dentro da memória principal. Este processo terá suas theread, como: escrever, corretor ortografico, salvamento automático e etc. 

Trago abaixo, um desenho feito por AI, de como enxergar um processo na memória. 

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

## Heap x Stack

Stack é um local na memória que funciona como LIFO. Ela aloca, parâmetros da função, endereço de retorno, variáveis locais e registradores salvos. Sempre que uma chamada de função for realizada ela faz o empilhamento e sempre a última a chegar, é a primeira a sair. Ela é bem menor que a Heap. 
Heap é uma região na memória grande e flexível para alocação dinâmica: podendo alocar-se um bloco do tamanho necessário. Os dados aqui geralmente vivem além da função. O stack aponta endereços daqui. 

