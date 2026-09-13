---
titulo: "Comandos essenciais do Git"
tags: [git, referencia]
nivel: iniciante
atualizado: 2026-09-13
---

# Comandos essenciais do Git

Referência rápida dos comandos do dia a dia. Para instalar e configurar o Git do zero
(identidade, chave SSH, conexão com o GitHub), veja
[Configuração do Git em ambiente Linux](configuracao-linux-ssh.md).

---

## Básicos

### Criar um novo repositório

```bash
git init
```

### Copiar um repositório existente

```bash
git clone <url>
```

### Ver alterações pendentes

```bash
git status
```

### Adicionar alterações à área de preparação

```bash
git add <arquivo>
```

Para adicionar tudo que mudou:

```bash
git add .
```

### Salvar as alterações preparadas

```bash
git commit -m "mensagem"
```

> A mensagem descreve **o que mudou**, no imperativo: `"Adiciona validação de CPF"`,
> não `"mudanças"`.

---

## Sincronização

### Trazer alterações do repositório remoto

```bash
git pull
```

### Subir alterações ao repositório remoto

```bash
git push
```

### Conectar seu repositório local com um remoto

```bash
git remote add <nome> <url>
```

Por convenção, o remoto principal se chama `origin`:

```bash
git remote add origin git@github.com:usuario/repositorio.git
```

### Baixar alterações, mas sem mesclá-las

```bash
git fetch
```

> `git fetch` atualiza sua cópia do remoto sem tocar nos seus arquivos.
> `git pull` é o `fetch` seguido de um `merge` automático.

---

## Branches

### Listar, criar ou excluir branches

```bash
git branch
```

```bash
git branch <nome-da-branch>
```

### Mudar de branch

```bash
git switch <nome-da-branch>
```

Para criar e já mudar para ela:

```bash
git switch -c <nome-da-branch>
```

### Mesclar alterações de outra branch

```bash
git merge <nome-da-branch>
```

### Excluir uma branch

```bash
git branch -d <nome-da-branch>
```

> ⚠️ `-d` só apaga a branch se ela já tiver sido mesclada. O `-D` (maiúsculo) apaga de
> qualquer jeito — e os commits exclusivos daquela branch ficam órfãos.

---

## Desfazer alterações

### Desfazer alterações em um arquivo

```bash
git restore <arquivo>
```

> ⚠️ Isso descarta as edições não commitadas do arquivo, sem possibilidade de recuperação.

### Remover um arquivo da área de preparação

```bash
git restore --staged <arquivo>
```

A forma antiga, ainda muito usada:

```bash
git reset HEAD <arquivo>
```

### Criar um commit que reverte outro commit

```bash
git revert <id-commit>
```

> É a forma segura de desfazer algo que já foi para o remoto: em vez de reescrever o
> histórico, adiciona um commit novo que anula o anterior.

---

## Avançado

### Ver histórico de commits resumido e visual

```bash
git log --oneline --graph --all
```

### Salvar alterações temporariamente sem fazer um commit

```bash
git stash
```

### Restaurar alterações salvas

```bash
git stash pop
```

> Útil quando você precisa trocar de branch no meio de uma edição: guarda o trabalho em
> andamento, troca, e depois recupera com `git stash pop`.

### Reaplicar commits de uma branch em outra, para um histórico limpo

```bash
git rebase <branch>
```

> ⚠️ Rebase reescreve o histórico. Nunca faça rebase de commits que já foram enviados ao
> repositório remoto e que outras pessoas possam ter baixado.

### Aplicar um commit específico em outra branch

```bash
git cherry-pick <id-commit>
```

---

## Ajuda

### Ver todas as opções do manual de ajuda

```bash
git help --all
```

Ou a ajuda de um comando específico:

```bash
git <comando> --help
```

---

## Resumo

Todos os comandos acima em uma tela, para consulta rápida:

| Grupo | Comando | O que faz |
| --- | --- | --- |
| **Básicos** | `git init` | Cria um novo repositório |
| | `git clone <url>` | Copia um repositório existente |
| | `git status` | Mostra as alterações pendentes |
| | `git add <arquivo>` | Adiciona alterações à área de preparação |
| | `git commit -m "mensagem"` | Salva as alterações preparadas |
| **Sincronização** | `git pull` | Traz alterações do repositório remoto |
| | `git push` | Envia alterações ao repositório remoto |
| | `git remote add <nome> <url>` | Conecta o repositório local a um remoto |
| | `git fetch` | Baixa alterações sem mesclá-las |
| **Branches** | `git branch` | Lista as branches |
| | `git branch <nome>` | Cria uma branch |
| | `git switch <nome>` | Muda de branch |
| | `git switch -c <nome>` | Cria a branch e já muda para ela |
| | `git merge <nome>` | Mescla as alterações de outra branch |
| | `git branch -d <nome>` | Exclui uma branch já mesclada |
| **Desfazer** | `git restore <arquivo>` | Descarta as edições não commitadas |
| | `git restore --staged <arquivo>` | Tira o arquivo da área de preparação |
| | `git revert <id-commit>` | Cria um commit que anula outro |
| **Avançado** | `git log --oneline --graph --all` | Histórico resumido e visual |
| | `git stash` | Guarda o trabalho em andamento sem commitar |
| | `git stash pop` | Restaura o que foi guardado |
| | `git rebase <branch>` | Reaplica os commits sobre outra base |
| | `git cherry-pick <id-commit>` | Aplica um commit específico em outra branch |
| **Ajuda** | `git help --all` | Lista todas as opções do manual |
| | `git <comando> --help` | Abre a ajuda de um comando específico |
