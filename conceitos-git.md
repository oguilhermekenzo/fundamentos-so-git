# Controle de versão com Git e GitHub

## Modelo mental do Git

- **Diretório de trabalho:** os arquivos que você edita.
- **Staging area:** uma "sala de espera": você escolhe quais mudanças entram no próximo commit.
- **Repositório local:** o histórico guardado na pasta oculta `.git`.
- **Repositório remoto:** a cópia no GitHub (por convenção chamada de `origin`).

### O que é um commit

Um **commit** é uma **foto (snapshot)** do projeto naquele momento, com:

- um **hash** único (identificador de 40 caracteres, normalmente exibido com 7, como `a3f9c2d`);
- **autor**, **data** e uma **mensagem**;
- uma referência ao(s) commit(s) **pai(s)**.

Como cada commit aponta para o anterior, o histórico forma uma **cadeia** (na verdade, um grafo). Commits são **imutáveis**: você não "edita" um commit, você cria outro (ou reescreve o histórico, o que veremos no rebase).

### Bons commits

- **Pequenos e com um propósito só** (uma mudança lógica por commit).
- **Mensagem clara**, que diz **o que** e **por quê**.
- O roadmap pede o padrão **Conventional Commits**: `tipo(escopo): descrição`.

```bash
  feat(auth): adiciona rota de login
  fix(pedidos): corrige cálculo do frete
  docs: atualiza README com instruções de execução
```

## Configuração inicial

```bash
git --version
git config --global user.name "Seu Nome"
git config --global user.email "seu@email.com"
git config --global init.defaultBranch main
```

O nome e o e-mail aparecem em cada commit seu. Use no e-mail da config o mesmo cadastrado no GitHub para que seus commits sejam associados ao seu perfil.

Para criar ou copiar um repositório:

```bash
git init               # transforma a pasta atual em um repositório
git clone <url>        # copia um repositório do GitHub para o seu computador
```

**Autenticação com o GitHub:** ele **não aceita mais senha** para operações Git por HTTPS. As opções são **chave SSH** (recomendada), _Personal Access Token_ ou a ferramenta `gh auth login`. Com SSH, você gera um par de chaves (`ssh-keygen -t ed25519`) e cadastra **apenas a chave pública** (`.pub`) no GitHub. A chave privada nunca sai da sua máquina.

## Branches (ramificações)

Uma branch é só um **ponteiro móvel para um commit**. Criar uma branch **não copia nada**, é praticamente instantâneo e custa quase nada. O **HEAD** indica em qual branch (e commit) você está agora.

```
A ─── B ─── C        ← main
\
D ─── E     ← feature/login   (HEAD)
```

Para que serve: você desenvolve uma funcionalidade ou corrige um bug **isolado**, sem mexer na `main` (que deve estar sempre funcionando).

```bash
git branch                       # lista as branches
git switch -c feature/login      # cria e entra na branch
git switch main                  # volta para a main
git branch -d feature/login      # apaga a branch (depois de mesclada)
```

### Convenção de nomes

`feature/login`, `fix/calculo-frete`, `docs/readme`. Nomes curtos e descritivos.

#### Estratégias de trabalho com branches

- **GitHub Flow (o mais usado hoje e o melhor para você agora):** `main` sempre estável, uma branch curta por tarefa, Pull Request, revisão e merge.
- **Git Flow:** mais estruturado, com `develop`, `release` e `hotfix`. Mais pesado, comum em times com versões fixas.
- **Trunk-based:** todos integram na `main` com muita frequência, em branches de vida bem curta.

## Merge (mesclar)

**Merge** junta o histórico de uma branch em outra. Você vai para a branch de **destino** e mescla a de **origem**:

```bash
git switch main
git merge feature/login
```

Há dois tipos de resultado:

#### Fast-forward (avanço rápido)

Quando a `main` **não andou** desde que a branch nasceu, o Git só move o ponteiro para frente. Não cria commit novo e o histórico fica em linha reta.

```
Antes:   A ─ B ─ C           ← main
                  \
                   D ─ E     ← feature

Depois:  A ─ B ─ C ─ D ─ E   ← main, feature
```

#### Merge commit (merge "de três vias")

Quando as duas branches **seguiram caminhos diferentes**, o Git cria um **commit de merge**, com **dois pais**.

```
Antes:   A ─ B ─ C ─ F        ← main
                  \
                   D ─ E       ← feature

Depois:  A ─ B ─ C ─ F ─ M     ← main      (M tem dois pais: F e E)
                  \     /
                   D ─ E
```

#### Conflitos de merge

Se as duas branches **alteraram as mesmas linhas** do mesmo arquivo, o Git não sabe qual versão manter e para, pedindo que você decida. O arquivo fica assim:

```
<<<<<<< HEAD
versão da branch em que você está
=======
versão que veio da outra branch
>>>>>>> feature/login
```

Para resolver:

1. Abra o arquivo e **edite** deixando o resultado final correto (apague os marcadores `<<<<<<<`, `=======`, `>>>>>>>`).
2. `git add arquivo`
3. `git commit` para concluir o merge.

Se quiser desistir: `git merge --abort`. O VS Code mostra botões ("Aceitar atual", "Aceitar entrada", "Aceitar ambos") que ajudam. Conflito não é erro: é o Git pedindo uma decisão humana. A próxima aula detalha isso.

## Rebase

#### O que faz

O **rebase** pega os commits da sua branch e os **reaplica sobre outra base**, como se você tivesse começado seu trabalho a partir do ponto mais recente. O resultado é um **histórico linear**.

```
Antes:   A ─ B ─ C ─ F        ← main
                  \
                   D ─ E       ← feature

Depois do rebase da feature sobre a main:

         A ─ B ─ C ─ F ─ D' ─ E'    ← feature (D' e E' são cópias novas de D e E)
                       ↑
                      main
```

bash

```bash
git switch feature/logingit rebase main
```

**Importante:** D' e E' são commits **novos**, com **hashes diferentes**. O rebase **reescreve o histórico**.

## Pull Requests (PR)

#### O que é

Um Pull Request é um recurso do **GitHub** (no GitLab chama-se _Merge Request_), **não do Git**. É um pedido formal: _"Quero mesclar esta branch na main, por favor revisem."_ Ele reúne o código alterado, a discussão, os comentários e os testes automáticos em um só lugar.

#### O fluxo completo

```
1. Cria uma branch a partir da main atualizada
2. Faz os commits
3. Envia a branch para o GitHub (push)
4. Abre o Pull Request
5. Revisores comentam, aprovam ou pedem mudanças
6. Os testes automáticos (CI) rodam
7. Você ajusta (novos commits na mesma branch atualizam o PR)
8. Alguém faz o merge
9. A branch é apagada
```

#### Revisão de código (code review)

É a parte mais valiosa. O revisor lê as mudanças, comenta em linhas específicas e escolhe: **Comment**, **Approve** ou **Request changes**. Isso melhora a qualidade, espalha conhecimento no time e pega erros cedo. No seu roadmap, é exatamente o papel do mentor em cada entrega.
