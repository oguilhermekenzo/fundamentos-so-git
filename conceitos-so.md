# Explicando conceitos do sistema operacional

## Programas, processos e Threads

Programa é um arquivo parado no disco, como um executável de jogo/app ou um script em alguma linguagem. Neste momento, ele está apenas ocupando espaço do armazenamento. 

Quando falamos de Processo, estamos falando desse programa em estado de execução, executado pelo SO. O processo recebe um espaço na memória principal (RAM) só para ele, ele recebe também um PID (seu identificador) e os recursos que ele vai usar (arquivos, sockets).

Já uma Thread é uma linha de execução (sequência de instruções) dentro de um processo. O processo pode ter uma ou várias threads rodando em paralelo. Todas as threads dividem o mesmo espaço de memória do processo, e cada uma tem sua própria pilha de execução, a stack e registradores. 

É o SO que cria o identificador de cada processo, tendo um limite, o que faz acontecer que um PID pode ser reutilizado em um outro processo. Quando um programa roda duas vezes, são dois processos diferentes com suas threads e seus identificadores. 

Um exemplo prático seria um editor de texto, como o Word. O programa está parado na memória de armazenamento, mas quando seu executável é acionado então o SO coloca este programa para rodar e ele vira um processo dentro da memória principal. Este processo terá suas thread, como: escrever, corretor ortografico, salvamento automático e etc. 

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

## Concorrência x Paralelismo 

Quando falamos de concorrência, estamos falando de um único núcleo dividindo diversas tarefas, intercalando entre elas. E quando falamos de paralelismo, estamos falando de diversas tarefas, mas cada uma rodando em um núcleo diferente.

Dentro deste tema, podemos falar da context switching, que é quando o SO para uma thread e coloca outra para rodar. Antes disso, ele salva onde a thread parou e quando ela volta a rodar, volta exatamente do estado que parou. 

Neste tema, falamos também de race condition, que é quando duas tarefas ou mais, tentam acessar ou modificar o mesmo recurso. Ou seja, uma thread chega para modificar algo, mas a outra que acessou no mesmo tempo, sobrescreve o que a primeira fez, para isso usamos algumas técnicas para que quando a thread acessar, ela bloqueie o acesso ao recurso e libere apenas ao sair. 

Por fim, falamos de deadlocks, quando duas threads ficam esperando uma a outra para sempre. Ou seja, a thread A para ficar na condição X espera condição Y da thread B, mas para a thread B ficar na condição Y ela espera a thread A ficar na condição X. E vamos usar das técnicas na race condition para organizar e prevenir-se dos deadlocks. 

## Como serviços rodam no SO

Quando rodamos "node app.js" no terminal, o programa vai sobreviver enquanto o terminal está de pé, ao matar o terminal, o programa também morre. 

Em produção, ele vai precisar subir sozinho quando o servidor liga, permanecer de pé sem ninguém logado e reiniciar caso caia. E é por isso que existem os serviços.

O  serviço vai: 

1. Iniciar no boot;
2. Rodar desaclopado de qualquer terminal; 
3. É supervisionado (se cair, pode ser reiniciado automaticamente); 

Para o linux, temos os daemons e para o windows temos o windows services, que são processos que rodam no background sem terminal associado.  

