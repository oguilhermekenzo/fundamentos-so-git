# Controle de versão com Git e GitHub

## Modelo mental do Git

- **Diretório de trabalho:** os arquivos que você edita.
- **Staging area:** uma "sala de espera": você escolhe quais mudanças entram no próximo commit.
- **Repositório local:** o histórico guardado na pasta oculta `.git`.
- **Repositório remoto:** a cópia no GitHub (por convenção chamada de `origin`).

#### O que é um commit

Um **commit** é uma **foto (snapshot)** do projeto naquele momento, com:

- um **hash** único (identificador de 40 caracteres, normalmente exibido com 7, como `a3f9c2d`);
- **autor**, **data** e uma **mensagem**;
- uma referência ao(s) commit(s) **pai(s)**.

Como cada commit aponta para o anterior, o histórico forma uma **cadeia** (na verdade, um grafo). Commits são **imutáveis**: você não "edita" um commit, você cria outro (ou reescreve o histórico, o que veremos no rebase).

#### Bons commits

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

#### Convenção de nomes

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

## O fluxo básico, comando por comando

#### `git init` e `git clone`: ter um repositório

```bash
git init                         # transforma a pasta atual em repositório (cria a pasta .git)
git clone git@github.com:usuario/repo.git          # copia um repositório do GitHub
git clone <url> nome-da-pasta    # clona escolhendo o nome da pasta
git remote -v                    # mostra os remotos (o "origin")
git remote add origin <url>      # liga um repositório local a um remoto
```

O `clone` já traz todo o histórico e já configura o `origin`.

#### `git status`: o seu melhor amigo

Mostra o que mudou, o que está na staging e em qual branch você está. **Rode antes de qualquer outro comando** até virar hábito.

#### `git add`: escolher o que entra no próximo commit

```bash
git add arquivo.js          # um arquivo
git add src/                # uma pasta
git add .                   # tudo da pasta atual para baixo
git add -p                  # escolhe PEDAÇOS de cada arquivo, um por um
git restore --staged arq    # tira da staging (não perde a alteração)
```

O `git add .` é prático, mas pode levar lixo junto (um `.env`, um log). Confira antes de commitar: `git status` e `git diff --staged`. O `git add -p` ajuda a fazer **commits pequenos e focados**, mesmo quando você mexeu em coisas diferentes no mesmo arquivo.

#### `git commit`: gravar o snapshot

```bash
git commit -m "feat(auth): adiciona rota de login"
git commit -am "fix: corrige typo"          # add + commit, mas SÓ de arquivos já rastreados
git commit --amend -m "nova mensagem"       # corrige o ÚLTIMO commit
git commit --amend --no-edit                # inclui mais mudanças no último commit, mesma mensagem
```

Use o `--amend` **só se o commit ainda não foi enviado** (`push`), porque ele reescreve o commit (novo hash).

#### `git push` e `git pull`: conversar com o GitHub

```bash
git push                              # envia seus commits
git push -u origin feature/login      # 1ª vez de uma branch (liga ao remoto)
git pull                              # traz as novidades (fetch + merge)
git pull --rebase                     # traz as novidades usando rebase
git fetch                             # só baixa, sem mexer nos seus arquivos
```

- Se o `push` for **rejeitado** (*non-fast-forward*), alguém enviou commits antes de você. Faça `git pull`, resolva o que for preciso e envie de novo. **Não** force.
- Ao fazer `git pull` com históricos que divergiram, o Git pode pedir que você escolha entre merge e rebase (`git config --global pull.rebase false` para merge, `true` para rebase). Siga o padrão do seu time.

#### `git log`: ler o histórico

```bash
git log --oneline                    # um commit por linha
git log --oneline --graph --all      # com o desenho das branches
git log -5                           # só os 5 últimos
git log --author="Kenzo"
git log -p                           # mostra as mudanças de cada commit
git log --stat                       # arquivos alterados por commit
git log -- caminho/arquivo           # histórico de um arquivo
git show a3f9c2d                     # detalhes de um commit
git blame arquivo                    # quem mexeu em cada linha
```

#### `git diff`: ver o que mudou

Cada versão do `diff` compara duas áreas diferentes:

| Comando | Compara |
| --- | --- |
| `git diff` | Diretório de trabalho **x** staging (o que você mudou e ainda não adicionou) |
| `git diff --staged` | Staging **x** último commit (o que vai entrar no próximo commit) |
| `git diff HEAD` | Diretório de trabalho **x** último commit (tudo que está pendente) |
| `git diff main..feature` | Duas branches |
| `git diff abc123 def456` | Dois commits |

Extras úteis: `--stat` (resumo) e `--name-only` (só os nomes dos arquivos).

### `git stash`: guardar o trabalho pela metade

**Problema:** você está no meio de uma alteração (sem condições de commitar) e surge um bug urgente em outra branch. O `stash` guarda suas mudanças numa "gaveta" e deixa a pasta limpa.

```bash
git stash                        # guarda as mudanças rastreadas
git stash -u                     # inclui também arquivos novos (não rastreados)
git stash push -m "wip login"    # com uma descrição
git stash list                   # lista a gaveta
git stash pop                    # recupera a mais recente e REMOVE da gaveta
git stash apply                  # recupera, mas MANTÉM na gaveta
git stash drop                   # joga fora o stash mais recente
git stash show -p                # vê o que tem lá dentro
```

Fluxo típico:

```bash
git stash -u
git switch -c fix/bug-urgente main
# corrige, commita, abre PR...
git switch feature/login
git stash pop
```

Cuidados: o stash é **local** (não vai para o GitHub) e não deve ser seu "backup". Não deixe stashes esquecidos por semanas. O `pop` pode gerar **conflito** se o código mudou nesse meio-tempo.

### `git cherry-pick`: pegar um commit específico

Aplica **um commit de outra branch** na branch atual, criando uma **cópia com hash novo**.

```bash
git switch main
git cherry-pick a3f9c2d              # traz esse commit para a main
git cherry-pick abc123 def456        # vários
git cherry-pick -x a3f9c2d           # registra na mensagem de onde veio
git cherry-pick --continue           # após resolver um conflito
git cherry-pick --abort              # desiste
```

Quando usar:

- Um **hotfix** feito numa branch que também precisa ir para outra (por exemplo, para a `main` e para uma release).
- Você **commitou na branch errada**.
- Você quer aproveitar **um** commit de uma branch grande, sem trazer o resto.

Cuidado: cria commits **duplicados** (mesmo conteúdo, hashes diferentes) e pode causar conflitos mais tarde. Use com parcimônia. Para trazer várias mudanças, prefira merge ou rebase.

#### `git reset`: mover a branch para trás

O `reset` move o ponteiro da branch (e o HEAD) para outro commit, "esquecendo" os commits que ficaram depois. Ele tem **três modos**:

| Modo | Move o HEAD | Limpa a staging | Altera seus arquivos |
| --- | --- | --- | --- |
| `--soft` | Sim | Não (mudanças ficam **staged**) | Não |
| `--mixed` (padrão) | Sim | Sim (mudanças ficam **não staged**) | Não |
| `--hard` | Sim | Sim | **Sim, apaga as mudanças** |

```bash
git reset --soft HEAD~1      # desfaz o commit, mantém tudo pronto para recommitar
git reset HEAD~1             # desfaz o commit, mantém as mudanças nos arquivos
git reset --hard HEAD~1      # desfaz o commit E joga o conteúdo fora
```

`HEAD~1` significa "o pai do commit atual" (um passo atrás); `HEAD~3` são três passos.

#### `git revert`: desfazer criando um commit novo

O `revert` **não apaga nada do histórico**. Ele cria um **novo commit que faz o oposto** de um commit anterior.

bash

```bash
git revert a3f9c2d
git revert HEAD              # desfaz o último commit
```

Para desfazer um **merge commit**, é preciso indicar qual pai manter: `git revert -m 1 <hash-do-merge>`.

#### Revert x reset

|  | `reset` | `revert` |
| --- | --- | --- |
| O que faz | Move o ponteiro para trás, **esquece** commits | Cria um commit novo que **anula** o anterior |
| Reescreve o histórico? | **Sim** | **Não** |
| Seguro depois do push / em branch compartilhada? | **Não** | **Sim** |
| Quando usar | Corrigir commits **locais** ainda não publicados | Desfazer algo que **já está no GitHub** |