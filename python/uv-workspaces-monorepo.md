---
titulo: "Monorepo Python com uv e Ruff"
tags: [python, uv, ruff, monorepo, fastapi]
nivel: avancado
atualizado: 2026-01-16
---

# Monorepo Python com `uv` e Ruff

Este guia documenta a criação de uma arquitetura Monorepo escalável para Python, abandonando o gerenciamento tradicional (Conda/Pip) em favor da velocidade e dos recursos de **Workspaces** do `uv`.

## 1. O Conceito

Em vez de gerenciar múltiplos repositórios e sofrer com versões conflitantes de bibliotecas compartilhadas, utilizamos um **Monorepo**.

* **Single Source of Truth:** Um único arquivo de trava (`uv.lock`) para todos os projetos.
* **Workspaces:** Projetos isolados logicamente (`apps`, `libs`), mas que compartilham o mesmo ambiente virtual raiz.
* **Performance:** Uso do `uv` (escrito em Rust) para resolução de dependências instantânea.

## 2. Estrutura de Pastas

O objetivo final é esta estrutura:

```text
meu-monorepo/
├── pyproject.toml        # (Raiz) Orquestrador do Workspace
├── uv.lock               # (Raiz) Trava de versões global
├── .venv/                # (Raiz) Ambiente virtual unificado
├── .vscode/              # (Raiz) Configurações do editor
│   └── settings.json
├── apps/
│   └── app-server/       # Aplicação FastAPI
│       ├── pyproject.toml
│       └── src/
├── libs/
│   └── shared-lib/       # Biblioteca de utilitários
│       ├── pyproject.toml
│       └── src/
```

---

## 3. Passo a Passo: Configuração Inicial

### 3.1. Inicializar o Workspace

No terminal (WSL/Linux/Windows):

```bash
# Criar pasta e iniciar
mkdir meu-monorepo
cd meu-monorepo
uv init
```

Edite o arquivo `pyproject.toml` gerado na raiz para definir os membros:

```toml
[project]
name = "meu-monorepo"
version = "0.1.0"
description = "Monorepo raiz com uv"
requires-python = ">=3.12"
dependencies = []

[tool.uv.workspace]
members = ["apps/*", "libs/*"]
```

### 3.2. Criar os Membros (Sub-projetos)

```bash
# Criar estrutura de diretórios
mkdir -p libs apps

# Criar a biblioteca compartilhada
uv init --lib libs/shared-lib --name shared-lib

# Criar a aplicação (API)
uv init --app apps/app-server --name app-server
```

### 3.3. Vincular Dependências

Dizemos ao `app-server` que ele depende da `shared-lib` (o `uv` detecta automaticamente pelo nome do membro no workspace) e do `fastapi`. Também instalamos o `uvicorn` explicitamente no projeto para evitar conflitos com instalações globais (como Anaconda).

```bash
# Adiciona a lib local ao app (O uv resolve isso internamente sem --path)
uv add --package app-server shared-lib

# Adiciona dependências externas ao app
uv add --package app-server fastapi uvicorn
```

---

## 4. Implementação do Código

Para que os imports funcionem corretamente, precisamos respeitar a estrutura de pacotes.

### 4.1. A Biblioteca Compartilhada (`libs/shared-lib`)

Crie o arquivo: `libs/shared-lib/src/shared_lib/hello.py`
*(Nota: Crie o arquivo `hello.py`, evite usar o `__init__.py` para lógica principal)*

```python
def dizer_ola():
    return "Olá! Esta mensagem veio da shared-lib dentro do Monorepo!"
```

### 4.2. A Aplicação (`apps/app-server`)

Edite o arquivo: `apps/app-server/src/main.py`

```python
from fastapi import FastAPI
# Importa do pacote local (hífens viram underlines no import)
from shared_lib import hello 

# A variável deve se chamar 'app' por convenção do uvicorn
app = FastAPI()

@app.get("/")
def read_root():
    # Consome a função da lib
    mensagem_lib = hello.dizer_ola() 
    return {"mensagem": "API rodando!", "lib_compartilhada": mensagem_lib}
```

---

## 5. Qualidade de Código (Ruff)

Substituímos Flake8, Black e Isort pelo **Ruff**.

### 5.1. Instalação na Raiz

Ferramentas de dev ficam na raiz para serem usadas por todos.

```bash
uv add --dev ruff
```

### 5.2. Configuração Global

Adicione ao final do `pyproject.toml` da **raiz**:

```toml
[tool.ruff]
line-length = 88
target-version = "py312"
exclude = [".venv", "venv", ".git", "dist"]

[tool.ruff.lint]
select = ["E", "F", "I"] # Erros, Falhas Lógicas, Import Sort
fixable = ["ALL"]

[tool.ruff.format]
quote-style = "double"
indent-style = "space"
```

---

## 6. Automação no VS Code

Para garantir que o código seja formatado e corrigido ao salvar (`Ctrl+S`), configure o ambiente de trabalho.

1.  Instale a extensão **Ruff** (ID: `charliermarsh.ruff`) no VS Code.
2.  Crie a pasta `.vscode` na raiz e dentro dela o arquivo `settings.json`:

```json
{
  "python.defaultInterpreterPath": ".venv/bin/python",
  "python.analysis.extraPaths": [
    "${workspaceFolder}/libs/shared-lib/src",
    "${workspaceFolder}/apps/app-server/src"
  ],
  "[python]": {
    "editor.defaultFormatter": "charliermarsh.ruff",
    "editor.formatOnSave": true,
    "editor.codeActionsOnSave": {
      "source.fixAll": "explicit",
      "source.organizeImports": "explicit"
    }
  },
  "ruff.interpreter": [".venv/bin/python"], 
  "ruff.enable": true
}
```

---

## 7. Executando o Projeto

O comando deve ser executado apontando para o módulo Python, não para o arquivo físico, para garantir que os imports funcionem.

**Comando Correto (da raiz ou da pasta do app):**

```bash
# Navegue até o app
cd apps/app-server

# Rode apontando para o arquivo 'main' e a variável 'app'
# O --reload permite ver as mudanças em tempo real
uv run uvicorn main:app --reload
```

Acesse no navegador: `http://127.0.0.1:8000`

Você deverá ver o JSON:
```json
{
  "mensagem": "API rodando!",
  "lib_compartilhada": "Olá! Esta mensagem veio da shared-lib dentro do Monorepo!"
}
```