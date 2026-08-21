---
titulo: "Do pyproject.toml ao deploy: CI e Docker com uv"
tags: [python, uv, docker, ci-cd, github-actions]
nivel: avancado
atualizado: 2026-08-21
---

# Do `pyproject.toml` ao deploy: CI e Docker com `uv`

O `pyproject.toml` define as regras do projeto. Este guia cobre o que vem depois: garantir
que essas regras sejam cumpridas automaticamente e que o resultado chegue a produção como
uma imagem enxuta e segura.

> **Pré-requisito:** um projeto já estruturado com `pyproject.toml`. Se ainda não é o caso,
> comece por [fundamentos e referência](pyproject-toml-fundamentos.md), que cobre as PEPs,
> a escolha do backend de build, os metadados de `[project]` e a configuração centralizada
> de Ruff, pytest e mypy.

Os três estágios, nesta ordem:

1. **Lockfile** — congelar as versões exatas com `uv.lock`.
2. **CI** — validar lint, tipagem e testes a cada push, com GitHub Actions.
3. **Entrega** — imagem Docker multi-stage e orquestração local com Compose.

---

## 1. Reprodutibilidade: o lockfile e o CI

O `pyproject.toml` define as regras, mas é o CI (Continuous Integration) que garante o cumprimento delas. Como estamos usando `uv`, nosso pipeline será drasticamente mais rápido e determinístico do que os pipelines tradicionais.

### 1.1. O pré-requisito: o arquivo de trava `uv.lock`

Antes de automatizar, você precisa garantir a reprodutibilidade. Enquanto o `pyproject.toml` define dependências abstratas (ex: `pandas>=2.0`), o `uv.lock` define o snapshot exato (ex: `pandas==2.2.1`).

**Regra de Ouro:** Em projetos de aplicação, o arquivo `uv.lock` **deve** ser comitado no Git.

```bash
uv lock
```

### 1.2. O workflow do GitHub Actions

Crie o arquivo `.github/workflows/ci.yml`. Este pipeline usa a action oficial da Astral para instalar o `uv`, gerenciar cache e executar as ferramentas configuradas no seu `pyproject.toml`.

```yaml
name: Quality Gate

on:
  push:
    branches: [ main, master ]
  pull_request:

jobs:
  quality-check:
    name: Build & Test (Python 3.12)
    runs-on: ubuntu-latest

    steps:
      - name: Checkout do código
        uses: actions/checkout@v4

      # Instala uv + Python + Cache global automático
      - name: Configurar uv e Python
        uses: astral-sh/setup-uv@v5
        with:
          python-version: "3.12"
          enable-cache: true
          cache-dependency-glob: "uv.lock"

      # Garante que o lockfile corresponde ao pyproject.toml
      - name: Validar Lockfile
        run: uv lock --check

      # Instala dependências do grupo 'dev' (Ruff, Mypy, Pytest)
      - name: Instalar Dependências
        run: uv sync --dev

      # Executa ferramentas usando as configs do pyproject.toml
      # 'uv run' usa o ambiente isolado automaticamente
      - name: Linter & Formatter (Ruff)
        run: |
          uv run ruff format --check
          uv run ruff check

      - name: Tipagem Estática (Mypy)
        run: uv run mypy .

      - name: Testes (Pytest)
        run: uv run pytest
```

### 1.3. Por que esta arquitetura funciona

1. **Single Source of Truth:** O CI não tem flags mágicas. Ele apenas executa `uv run ruff check`. O Ruff, por sua vez, olha para o `pyproject.toml` para saber o que ignorar ou validar. Se você mudar a regra no TOML, o CI se atualiza automaticamente.
2. **Velocidade de Cache:** A action `setup-uv` com `enable-cache: true` evita baixar rodas (wheels) repetidamente, reduzindo o tempo de build de minutos para segundos.
3. **Segurança de Lockfile:** O comando `uv lock --check` falha o build se alguém alterar uma dependência no TOML mas esquecer de atualizar o lockfile, prevenindo divergências em produção.

## 2. Dockerfile otimizado: multi-stage e cache mounts

Seu pipeline de CI/CD termina aqui. Vamos criar uma imagem Docker de produção que é **leve** (apenas o runtime necessário) e **segura** (sem rodar como root).

Para isso, usamos duas técnicas avançadas:

1.  **Multi-stage Build:** Uma imagem "Builder" compila e instala tudo. Jogamos ela fora e copiamos apenas o ambiente virtual pronto para a imagem "Final".
2.  **Docker BuildKit Cache Mounts:** Configuramos o Docker para persistir o cache do `uv` entre builds diferentes. Se você mudar apenas uma lib, o `uv` não baixará as outras 50 novamente.

### 2.1. O `Dockerfile`

Salve este arquivo na raiz do projeto.

```dockerfile
# ----------------------------------------------------------------------------------
# ESTÁGIO 1: BUILDER
# Objetivo: Instalar dependências e criar o ambiente virtual.
# Usamos uma imagem base completa para garantir que temos ferramentas de sistema.
# ----------------------------------------------------------------------------------
FROM python:3.12-slim AS builder

# Instala o 'uv' copiando o binário oficial da imagem da Astral.
# Fixe a versão: ':latest' quebra a reprodutibilidade que o uv.lock garante.
COPY --from=ghcr.io/astral-sh/uv:0.5.11 /uv /bin/uv

# Define variáveis de ambiente para otimizar o uv
ENV UV_COMPILE_BYTECODE=1
ENV UV_LINK_MODE=copy

WORKDIR /app

# ETAPA 1 — SÓ AS DEPENDÊNCIAS (LAYER CACHING):
# Copiamos apenas os arquivos de definição. O Docker só invalida esta camada
# se o pyproject.toml ou o uv.lock mudarem — o código-fonte não a afeta.
COPY pyproject.toml uv.lock ./

# --mount=type=cache:      volume persistente para o cache de downloads do uv
# --frozen:                instala exatamente o que está no uv.lock (fail fast)
# --no-install-project:    ainda não copiamos o código, então o projeto em si
#                          não pode ser instalado nesta etapa
# --no-dev:                sem pytest, ruff etc. em produção
RUN --mount=type=cache,target=/root/.cache/uv \
    uv sync --frozen --no-install-project --no-dev

# ETAPA 2 — O CÓDIGO E A INSTALAÇÃO DO PRÓPRIO PROJETO.
# Esta camada muda a cada commit, por isso vem depois das dependências.
COPY . .
RUN --mount=type=cache,target=/root/.cache/uv \
    uv sync --frozen --no-dev

# ----------------------------------------------------------------------------------
# ESTÁGIO 2: RUNNER (FINAL)
# Objetivo: Imagem final enxuta, segura e pronta para execução.
# ----------------------------------------------------------------------------------
FROM python:3.12-slim AS final

# Configuração de Segurança: Criação de usuário não-privilegiado
# Nunca rode containers como root em produção.
RUN groupadd -r appuser && useradd -r -g appuser appuser

WORKDIR /app

# Copia APENAS o ambiente virtual (.venv) e o código do estágio 'builder'
# Isso elimina cache do uv, compiladores e arquivos temporários da imagem final.
COPY --from=builder --chown=appuser:appuser /app/.venv /app/.venv
COPY --from=builder --chown=appuser:appuser /app/src /app/src

# Adiciona o venv ao PATH.
# Agora, 'python' e 'uvicorn' referem-se aos executáveis dentro do venv.
ENV PATH="/app/.venv/bin:$PATH"

# Troca para o usuário seguro
USER appuser

# Exemplo de comando de entrada (para uma API FastAPI/Typer)
# Como o PATH está setado, não precisa ativar venv explicitamente.
CMD ["python", "-m", "core_analise.main"]
# Ou se for web:
# CMD ["uvicorn", "core_analise.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### 2.2. As decisões por trás do Dockerfile

1. `COPY --from=ghcr.io/astral-sh/uv`: Não instalamos o `uv` via pip ou curl. Copiamos o binário pré-compilado (Rust), o que é mais seguro e rápido.

2. `UV_COMPILE_BYTECODE=1`: O uv compila automaticamente os arquivos `.py` para `.pyc` durante a instalação. Isso melhora o tempo de inicialização (startup time) do container, pois o Python não precisa compilar na primeira execução.

3. `--mount=type=cache`: A mágica do BuildKit. Se você rodar `docker build` 10 vezes, o `uv` vai reutilizar os pacotes baixados anteriormente, fazendo builds subsequentes serem quase instantâneos.

4. **`uv sync` em duas etapas**: a primeira, com `--no-install-project`, instala só as
   dependências e fica em cache; a segunda instala o projeto em si, depois do `COPY . .`.
   Rodar um `uv sync` único antes de copiar o código falha, porque o backend de build não
   encontra os fontes do pacote. Rodar um único depois de copiar funciona, mas invalida o
   cache das dependências a cada commit.

5. **Um `.dockerignore` é obrigatório** ao lado do `Dockerfile`, senão o `COPY . .` leva o
   `.venv` local, o `.git` e os caches para dentro da imagem:

   ```text
   .venv
   .git
   .github
   .mypy_cache
   .ruff_cache
   .pytest_cache
   __pycache__
   *.pyc
   dist
   build
   .env
   ```

4. `python:slim` **vs Alpine**: Evitamos Alpine (musl libc) para Python em produção devido a problemas de compatibilidade com rodas (wheels) binárias padrão (manylinux/glibc) e performance em runtime. A versão `slim`(Debian-based) é o equilíbrio ideal.

### 2.3. Comando de build

O `--mount=type=cache` do estágio builder só funciona com o **BuildKit** ativo. No Docker
Desktop e no Docker Engine moderno ele já é o padrão, então basta:

```bash
docker build -t meu-projeto:latest .
```

Se você estiver num ambiente antigo — ou num CI que ainda use o builder legado — o build
falha com `the --mount option requires BuildKit`. Nesse caso, ative explicitamente:

```bash
DOCKER_BUILDKIT=1 docker build -t meu-projeto:latest .
```

Para confirmar qual builder está em uso:

```bash
docker buildx version
```

## 3. Orquestração local: Docker Compose

Agora que temos uma imagem isolada, precisamos subir a aplicação conectada a um banco de dados real. Vamos usar o padrão moderno de especificação (Compose Specification).

### 3.1. O arquivo `.env`

Primeiro, **nunca** commite senhas no Git. Crie um arquivo `.env` na raiz (e adicione-o ao `.gitignore`).

```ini
# .env
POSTGRES_USER=admin
POSTGRES_PASSWORD=segredo_super_seguro
POSTGRES_DB=app_db
DATABASE_URL=postgresql://admin:segredo_super_seguro@db:5432/app_db
```

### 3.2. O arquivo `compose.yml`

Crie o arquivo `compose.yml` na raiz. Note o uso de **Healthchecks**. Isso resolve aquele problema clássico da aplicação tentar conectar no banco antes dele estar pronto e "quebrar" na inicialização.

```yaml
services:
  # ----------------------------------------------------------------
  # SERVIÇO DE BANCO DE DADOS (Postgres)
  # ----------------------------------------------------------------
  db:
    image: postgres:15-alpine
    restart: always
    volumes:
      - postgres_data:/var/lib/postgresql/data
    env_file:
      - .env
    healthcheck:
      # Usa as variáveis do .env — não repita usuário e banco aqui, senão o
      # healthcheck passa a mentir assim que você mudar o .env.
      test: ["CMD-SHELL", "pg_isready -U $$POSTGRES_USER -d $$POSTGRES_DB"]
      interval: 5s
      timeout: 5s
      retries: 5
    ports:
      - "5432:5432" # Opcional: expõe para seu DBeaver/PgAdmin local

  # ----------------------------------------------------------------
  # SUA APLICAÇÃO PYTHON
  # ----------------------------------------------------------------
  app:
    build: .             # Usa o Dockerfile da seção 2
    env_file:
      - .env
    ports:
      - "8000:8000"
    depends_on:
      db:
        condition: service_healthy # Só inicia o app quando o DB estiver VERDE

    # Para desenvolvimento local com hot reload,
    # você pode sobrescrever o comando padrão.
    # command: uv run uvicorn core_analise.main:app --host 0.0.0.0 --reload

volumes:
  postgres_data:
```

### 3.3. Como rodar

1. **Subir o ambiente:**

```bash
docker compose up --build
```

O `--build` força a recriação da imagem caso você tenha mudado dependências no `pyproject.toml`.

2. **Derrubar e limpar volumes (Reset total):**

```bash
docker compose down -v
```

## Conclusão

O que a cadeia completa entrega:

| Camada | Ferramenta | Garantia |
| --- | --- | --- |
| Definição | `pyproject.toml` (PEP 621) | Fonte única da verdade |
| Gerenciamento | `uv` + `uv.lock` | Resolução rápida e versões congeladas |
| Qualidade | Ruff e mypy, configurados no TOML | Mesmas regras local e no CI |
| Integração | GitHub Actions com `uv lock --check` | Lockfile não diverge do TOML |
| Entrega | Docker multi-stage | Imagem enxuta, sem compiladores nem cache |
| Orquestração | Docker Compose com healthcheck | Ambiente local reprodutível |

O ponto que sustenta tudo isso é o CI não ter nenhuma configuração própria: ele só executa
`uv run ruff check`, `uv run mypy` e `uv run pytest`. As regras vivem no `pyproject.toml`,
então mudar uma regra lá atualiza o pipeline automaticamente.

## Ver também

- [`pyproject.toml`: fundamentos e referência](pyproject-toml-fundamentos.md) — as PEPs, os
  backends de build e a anatomia completa do arquivo.
- [Monorepo Python com `uv` e Ruff](uv-workspaces-monorepo.md) — quando o projeto vira
  vários `apps/` e `libs/` sob um único lockfile.
