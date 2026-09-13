---
titulo: "Revisar e mergear um Pull Request com o GitHub CLI"
tags: [git, github, gh, pull-request, code-review]
nivel: intermediario
atualizado: 2026-09-13
---

# Revisar e mergear um Pull Request com o GitHub CLI

Alguém abriu um Pull Request no seu repositório. E agora?

Este tutorial cobre o fluxo completo: entender o que foi proposto, **verificar se funciona
de verdade**, dar um veredito e mergear — tudo pelo terminal, com o `gh` (GitHub CLI).

O foco não são os comandos. Comando você encontra em qualquer lugar. O foco é a sequência
de decisões, e principalmente as armadilhas que fazem alguém aprovar um PR quebrado achando
que revisou.

> **Pré-requisitos:** Git instalado e configurado — veja
> [Configurar Git no Linux com SSH](configuracao-linux-ssh.md) — e um repositório no GitHub
> com um PR aberto.

---

## 1. Instalar e autenticar o gh

**Linux (Debian/Ubuntu):**

O `gh` **não está** nos repositórios padrão do Debian e do Ubuntu. Onde um pacote comunitário
existe, a própria documentação do projeto avisa que versões antigas quebram por usarem APIs
do GitHub já descontinuadas. Então adicione o repositório oficial antes de instalar:

```bash
# 1. Chave de assinatura do repositório oficial
sudo mkdir -p -m 755 /etc/apt/keyrings
wget -qO- https://cli.github.com/packages/githubcli-archive-keyring.gpg \
  | sudo tee /etc/apt/keyrings/githubcli-archive-keyring.gpg > /dev/null
sudo chmod go+r /etc/apt/keyrings/githubcli-archive-keyring.gpg

# 2. Repositório na lista de fontes do apt
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" \
  | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null

# 3. Agora sim
sudo apt update
sudo apt install gh -y
```

**Windows:**

```powershell
winget install GitHub.cli
```

Autentique — abre o navegador e pede confirmação de um código:

```bash
gh auth login
```

Confirme:

```bash
gh auth status
```

Você deve ver `Logged in to github.com account <seu-usuario>` e a lista de escopos do token.
Precisa ter `repo` ali.

> **No Windows:** depois de instalar, abra um terminal **novo**. Um terminal já aberto guarda
> o `PATH` de antes da instalação e vai insistir que o `gh` não existe.

## 2. As cinco decisões

Revisar um PR não são cinco comandos. São cinco perguntas:

| # | A pergunta | O comando que ajuda |
|---|---|---|
| 1 | O que está aberto? | `gh pr list` |
| 2 | O que ele diz que fez, e **por quê**? | `gh pr view <n>` |
| 3 | O que ele mudou de fato? | `gh pr checkout <n>` |
| 4 | **Funciona?** | rodar o código |
| 5 | Qual o veredito? | `gh pr comment` / `merge` |

A pergunta 4 é a única que a interface web do GitHub **não consegue** responder. O navegador
mostra o diff; ele não roda nada. É por isso que vale aprender o `gh` — ele reduz a quase
zero o custo de trazer o código para a sua máquina.

## 3. Decisão 1 — o que está aberto

```bash
gh pr list
```

```text
#1  feat: adiciona transcrição parcial  eduardomizael:feat/transcricao-parcial
```

Duas informações importam aqui.

O **número** (`#1`) é o que você usa em todos os comandos seguintes.

O **branch de origem** diz de onde veio. No formato `usuario:branch`, aquele `usuario:` antes
dos dois-pontos significa que o PR veio de um **fork** — uma cópia do seu repositório na
conta de outra pessoa. É assim que contribuições externas funcionam: quem não tem permissão
de escrita no seu repo forka, commita na cópia dele, e abre um PR pedindo que você puxe as
mudanças.

Sem o `usuario:`, o branch é interno e o autor já tem acesso de escrita.

> A saída do `gh` muda conforme o destino: no seu terminal vem em tabela; dentro de um script
> vem separada por tabs. Para automatizar, use `--json` em vez de tentar interpretar a tabela.

## 4. Decisão 2 — o que ele diz que fez, e por quê

```bash
gh pr view 1
```

Você recebe o cabeçalho (autor, estado, linhas alteradas, status de CI) e o texto que o autor
escreveu.

Leia com uma separação na cabeça:

- **O que ele afirma ter feito** é uma alegação. Você vai verificar nos passos seguintes. Não
  aceite ainda.
- **O porquê** é diferente: é o que define seu critério de aprovação. Sem entender qual
  problema o PR resolve, você não tem como julgar se a solução é proporcional, nem o que
  testar.

No cabeçalho, procure a linha de checks:

```text
+382 -28 • No checks
```

`No checks` significa que **não existe CI neste repositório**. Nada verificou esse código
automaticamente. Num projeto com CI, parte da verificação está terceirizada; sem CI, se você
não rodar, ninguém rodou — e o passo 4 deixa de ser boa prática para virar obrigação.

## 5. Decisões 3 e 4 — trazer o código e rodar

Para um diff de mais de umas 50 linhas, ler no terminal é pior do que ler no editor. Então os
dois passos se resolvem no mesmo comando:

```bash
gh pr checkout 1
```

Isso substitui três comandos que você faria na mão:

```bash
git ls-remote origin 'refs/pull/*/head'
git fetch origin refs/pull/1/head:algum-nome
git checkout algum-nome
```

E faz uma coisa a mais: configura o branch local para acompanhar o fork do autor. Se ele
publicar um commit novo no PR, um `git pull` seu já traz.

O branch local recebe o nome do branch **do autor**, não um genérico. Com vários PRs abertos,
você sabe o que é cada um sem consultar nada.

### Uma pegadinha do checkout em fork

Você pode querer conferir esse rastreamento com:

```bash
git branch -vv
```

E vai ver o branch do PR **sem** o `[origin/...]` que os outros branches têm. Parece que não
funcionou. Funcionou:

```bash
git config --get-regexp '^branch\.'
```

```text
branch.feat-xyz.remote  https://github.com/autor/repo.git
branch.feat-xyz.merge   refs/heads/feat-xyz
```

O `gh` apontou o upstream direto para a **URL** do fork, sem criar um remote nomeado. E o
`git branch -vv` só exibe upstream quando ele resolve para um ref local de rastreamento
(`refs/remotes/algo/...`), o que exige um remote com nome. Sem nome, não há o que exibir.

O teste que vale é o prático:

```bash
git pull
```

Se responder `Already up to date` em vez de reclamar que não sabe de onde puxar, está
configurado.

### Agora rode

Instale as dependências e rode a suíte de testes do projeto:

```bash
# Python com uv
uv sync
uv run python -m unittest discover -s tests

# Node
npm install && npm test
```

Depois **use o programa**. Testes automatizados verificam o que alguém pensou em verificar;
usar o programa encontra o que ninguém pensou.

## 6. A armadilha: "os testes passam"

Esta seção é a mais importante do tutorial.

Um PR chegou com nove testes, todos verdes. Dentro deles:

```python
with patch.object(transcribe, "decode_audio_range", return_value=("audio", 100.0, 10.0, 30.0)):
```

`decode_audio_range` era a função que fazia o trabalho central do PR. E em todos os testes que
a envolviam ela estava **mockada** — substituída por um valor fixo. A suíte inteira nunca
executava uma linha dela.

Isso não é desonestidade: mockar I/O em teste unitário é prática normal, senão o teste
precisaria de um arquivo de mídia real. Mas a consequência é concreta: os nove testes verdes
não diziam nada sobre a alegação principal do PR.

> **"Os testes passam" nunca é a pergunta. A pergunta é o que eles testam.**

Antes de confiar numa suíte, procure `mock`, `patch`, `stub` ou `fake` e veja o que foi
substituído. Se a função central do PR está entre elas, você ainda não verificou nada — e
precisa testar na mão.

## 7. Monte um teste que se verifica sozinho

Tendo que testar na mão, existe um jeito ruim e um bom.

**Ruim:** rodar o programa e julgar se a saída "parece certa". Depende da sua atenção, não
escala, e você vai aprovar coisa errada num dia cansado.

**Bom:** montar o teste de forma que a resposta seja comparável a algo que você já tem. Alguns
padrões:

- **Contra uma saída anterior.** Se o projeto já gerou resultados antes da mudança, eles são o
  gabarito. Rode a versão nova e compare.
- **Contra outro caminho do próprio código.** Se o PR adiciona um atalho rápido, compare o
  resultado dele com o caminho lento que já existia.
- **Contra uma propriedade que tem de valer sempre.** Soma que fecha, total que bate, arquivo
  que reabre sem erro.

No exemplo real: o PR adicionava transcrição de um trecho de um vídeo. Havia a transcrição
completa do mesmo vídeo, feita antes. Bastou transcrever o intervalo `00:30:00`–`00:35:00` e
comparar contra o mesmo intervalo do gabarito. O texto bateu em 97,8%, e uma frase aparecia em
`1801.28` nos dois arquivos — prova de que a conversão de tempo estava correta.

A pergunta saiu de "você acha que está certo?" para `1801.28 == 1801.28`.

> **Cuidado ao comparar:** confira se as condições batem. Se o gabarito foi gerado com uma
> configuração e você roda com outra, os resultados vão divergir e o PR leva a culpa por um
> erro seu.

## 8. Decisão 5 — o veredito

Três caminhos, com consequências diferentes.

### a) Pedir mudanças

```bash
gh pr review 1 --request-changes -b "Descrição do que precisa mudar"
```

Padrão da indústria. O autor faz a correção, aprende seu critério, e a autoria fica com ele.
Para sugerir código linha a linha, use a aba **Files changed** no navegador: o recurso de
*suggested change* deixa o autor aceitar sua correção com um clique. Isso o `gh` não faz bem —
é o caso em que a web ganha.

### b) Corrigir você mesmo

Se o PR veio de fork, verifique:

```bash
gh pr view 1 --json maintainerCanModify
```

`true` significa que o autor deixou marcada a opção *"Allow edits by maintainers"* (vem
marcada por padrão). Você pode commitar **direto no branch dele**:

```bash
# você já está no branch do PR, via gh pr checkout
git commit -am "fix: descrição do ajuste"
git push
```

Confira na saída do push que o destino é o **repositório do autor**, não o seu. O commit
aparece no PR automaticamente, com você como autor.

É mais rápido, mas tem custo: você tira do autor a chance de fazer a correção, e ele vê um
commit que não escreveu. **Comente no PR explicando o que mudou e por quê** — não é opcional:

```bash
gh pr comment 1 --body-file comentario.md
```

O `--body-file` evita o inferno de aspas, acento e quebra de linha de um texto longo na linha
de comando.

### c) Aprovar

```bash
gh pr review 1 --approve
```

Se você é dono do repositório e vai mergear em seguida, o `--approve` formal é cerimônia — um
`gh pr comment` entrega a informação. Ele é necessário quando o repositório exige aprovação
para mergear.

## 9. Mergear

Três estratégias, e a escolha é convenção do projeto:

```bash
gh pr merge 1 --squash    # colapsa todos os commits do PR em um só
gh pr merge 1 --merge     # cria commit de merge, preserva tudo
gh pr merge 1 --rebase    # empilha os commits linearmente, sem merge commit
```

**`--squash`** é uma boa escolha quando os commits do PR não fazem sentido isolados — um
`"fix: corrige typo"` sozinho não significa nada para quem ler daqui a um ano. Um commit por
funcionalidade completa lê melhor.

**`--merge`** e **`--rebase`** preservam os commits individuais e seus autores. Prefira quando
cada commit conta uma etapa que vale manter separada.

> **Sobre autoria:** ao fazer squash de commits de pessoas diferentes, o GitHub mantém o autor
> do PR como autor e adiciona os demais como `Co-authored-by:` na mensagem. O crédito fica
> correto sozinho.

### Evite o --delete-branch em PR de fork

A opção existe e você tem permissão, mas num PR de fork ela apaga o branch **no repositório do
autor**. Deixe que ele apague o dele. Para limpar o seu lado:

```bash
git checkout main
git pull
git branch -d nome-do-branch
```

Se você usou `--squash`, esse último comando vai **recusar**:

```text
error: the branch 'xyz' is not fully merged
```

Não está quebrado. O squash não incorporou seus commits — criou um commit novo, com conteúdo
equivalente e hash diferente. Para o Git, os commits originais só existem no seu branch local.
O `-d` é a trava que protege contra apagar trabalho não incorporado; aqui é um falso positivo,
porque você sabe que o conteúdo entrou.

> ⚠️ O `-D` a seguir apaga o branch local **sem verificar** se o trabalho foi incorporado. Só
> rode depois de confirmar que o merge entrou na `main` — o que estiver apenas nesse branch se
> perde.

```bash
git branch -D nome-do-branch
```

Esse é o preço do squash: histórico limpo, mas o Git perde o rastro entre o branch e o que
entrou na `main`. O `--rebase` não tem esse efeito.

## 10. Os hábitos que ficam

Os comandos você esquece e reconsulta. Estes quatro hábitos são o que transformam leitura de
diff em revisão de verdade:

1. **"Os testes passam" não é a pergunta — a pergunta é o que eles testam.** Procure os mocks
   antes de confiar numa suíte verde.

2. **Toda afirmação tem um comando de verificação de um segundo.** Vale para o autor do PR,
   para tutoriais, e para este texto. Rode e confira.

3. **Monte o teste para que ele se verifique sozinho.** Um gabarito transforma "parece certo"
   em uma comparação objetiva.

4. **Confira as condições antes de comparar.** Versão, modelo, configuração e dados de entrada
   iguais dos dois lados — senão você mede o seu próprio erro.

## Referência rápida

```bash
gh pr list                          # PRs abertos
gh pr view 1                        # detalhes e descrição
gh pr view 1 --web                  # abre no navegador
gh pr view 1 --json <campos>        # saída estruturada
gh pr diff 1                        # diff no terminal
gh pr checkout 1                    # traz o branch para a máquina
gh pr checks 1                      # status de CI
gh pr comment 1 --body-file x.md    # comentário simples
gh pr review 1 --approve            # veredito formal
gh pr merge 1 --squash              # mergeia
```

## Ver também

- [Comandos essenciais do Git](comandos-essenciais.md) — a base do dia a dia: branches,
  sincronização e como desfazer o que deu errado.
- [Configurar Git no Linux com SSH](configuracao-linux-ssh.md) — identidade dos commits e
  conexão com o GitHub, pré-requisito deste tutorial.

## Referências

- [cli.github.com](https://cli.github.com) — documentação oficial do GitHub CLI
- [Instalação no Linux](https://github.com/cli/cli/blob/trunk/docs/install_linux.md) — o
  procedimento do repositório oficial, por distribuição
