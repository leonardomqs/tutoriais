---
titulo: "pyproject.toml: workflow completo, do projeto ao deploy"
tags: [python, pyproject, uv, docker, ci-cd]
nivel: avancado
atualizado: 2026-01-19
---

# `pyproject.toml`: workflow completo, do projeto ao deploy

Do arquivo de configuração até a imagem em produção: metadados PEP 621, configuração
centralizada de ferramentas, pipeline no GitHub Actions com `uv`, `Dockerfile` multi-stage
e orquestração local com Docker Compose.

> **Público-alvo:** quem está migrando de `setup.py` / `requirements.txt`.
> Para a fundamentação das PEPs e a comparação entre backends de build, veja
> [fundamentos e referência](pyproject-toml-fundamentos.md).

---

O `pyproject.toml` não é apenas mais um arquivo de configuração; é a **Single Source of Truth (SSOT)** do seu projeto. Se você está saindo do imperativo `setup.py` ou do frágil `requirements.txt`, bem-vindo à era declarativa.

## 1. Fundamentos Teóricos (O "Porquê")

Para entender a arquitetura moderna, você precisa entender o problema que as PEPs (Python Enhancement Proposals) resolveram:

- **PEP 518 (Build Dependencies):** Antes, para instalar um pacote, o `pip` precisava executar o `setup.py`. Mas o `setup.py` poderia importar `numpy`. A PEP 518 introduziu o `pyproject.toml` para definir o que é necessário _apenas para buildar_ o projeto, resolvendo o problema do "ovo e da galinha".
- **PEP 517 (Build Isolation):** Padronizou a interface entre o frontend de instalação (ex: `pip`, `uv`) e o backend de build (ex: `hatchling`, `setuptools`). Isso permitiu que ferramentas construíssem pacotes sem saber "como" o pacote é feito internamente.
- **PEP 621 (Standardized Metadata):** Esta é a virada de chave. Antes, cada ferramenta (`poetry`, `flit`, `setuptools`) tinha sua própria sintaxe. A PEP 621 padronizou tudo sob a tabela `[project]`. **Se uma ferramenta não suporta PEP 621 hoje, considere-a legada.**

## 2. O Build System: O Motor do Projeto

O Build System transforma seu código fonte em um artefato distribuível (`sdist` ou `wheel`).

### O Ecossistema Moderno

- **Hatchling (Recomendado):** O padrão moderno. Extremamente performático, suporta _plugins_ e adere estritamente à PEP 621. É o backend _default_ do `uv`.
- **Setuptools:** O "velho de guerra". Suporta PEP 621, mas carrega anos de débito técnico. Use apenas se precisar compilar extensões C legadas complexas.
- **Flit:** Minimalista, mas carece de recursos avançados de build hooks.
- **Poetry-core:** Bom, mas o cliente Poetry muitas vezes foge dos padrões da PEP 621, dificultando a interoperabilidade.

**Configuração do Backend (Exemplo com Hatchling):**

```toml
[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"
```

## 3. Anatomia do Arquivo: Metadados (PEP 621)

Esqueça `setup.cfg`. Tudo vive na tabela `[project]`.

### Versão Dinâmica vs. Estática

Em projetos profissionais, **nunca** hardcode a versão no TOML. A versão deve viver no código (`__init__.py`) ou vir de uma tag git.

```toml
[project]
name = "meu-sistema-enterprise"
dynamic = ["version"] # Indica que o backend vai descobrir a versão
description = "Backend de processamento de dados em alta performance."
readme = "README.md"
requires-python = ">=3.12"
license = { file = "LICENSE" }
authors = [
  { name = "Engenharia de Dados", email = "dev@empresa.com" }
]

# Dependências de PRODUÇÃO (O antigo requirements.txt)
dependencies = [
  "fastapi>=0.109.0",
  "pydantic>=2.6.0",
  "numpy>=1.26.0",
]

# Configuração do Hatchling para achar a versão
[tool.hatch.version]
path = "src/meu_sistema/__init__.py"
```

## 4. Centralização de Configurações (A "Killer Feature")

Limpamos a raiz do projeto movendo configs soltas para a tabela `[tool.NOME_DA_FERRAMENTA]`.

### Ruff (Linter & Formatter)

Substitui Flake8, Black, Isort e Pylint.

```toml
[tool.ruff]
line-length = 88
target-version = "py312"

[tool.ruff.lint]
select = [
    "E",   # pycodestyle errors
    "W",   # pycodestyle warnings
    "F",   # pyflakes
    "I",   # isort (ordenação de imports)
    "B",   # flake8-bugbear (bugs comuns)
    "UP",  # pyupgrade (sintaxe moderna)
]
ignore = ["E501"]
```

### Pytest

Elimine o `pytest.ini`.

```toml
[tool.pytest.ini_options]
minversion = "6.0"
addopts = "-ra -q --cov=src"
testpaths = ["tests"]
python_files = ["test_*.py"]
```

### Mypy (Tipagem Estática)

```toml
[tool.mypy]
python_version = "3.12"
strict = true
ignore_missing_imports = true

[[tool.mypy.overrides]]
module = "pandas.*"
ignore_missing_imports = true
```

## 5. Dependency Groups: Dev vs. Prod

**Abordagem Recomendada: Dependency Groups (PEP 735)** Suportado nativamente pelo uv. Separa totalmente o ambiente de dev do build final.

```toml
[dependency-groups]
dev = [
  "ruff>=0.1.0",
  "pytest>=8.0.0",
  "mypy>=1.8.0",
]
```

## 6. O Exemplo "Full-Stack"

Abaixo, um template robusto para um projeto Python 3.12+ usando **Hatchling** e focado em workflow com **uv**.

```toml
# pyproject.toml

# ------------------------------------------------------------------
# 1. BUILD SYSTEM
# ------------------------------------------------------------------
[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

# ------------------------------------------------------------------
# 2. PROJECT METADATA (PEP 621)
# ------------------------------------------------------------------
[project]
name = "core-analise-dados"
dynamic = ["version"]
description = "Sistema central de processamento e análise."
readme = "README.md"
requires-python = ">=3.12"
license = { text = "Proprietary" }
authors = [
    { name = "Time de Arquitetura", email = "arch@empresa.com" }
]
classifiers = [
    "Programming Language :: Python :: 3.12",
    "Operating System :: OS Independent",
]

dependencies = [
    "httpx>=0.26.0",
    "pydantic>=2.6.0",
    "pandas>=2.2.0",
    "typer>=0.9.0",
]

[project.scripts]
core-app = "core_analise.main:app"

# ------------------------------------------------------------------
# 3. DEPENDENCY GROUPS (PEP 735 / UV)
# ------------------------------------------------------------------
[dependency-groups]
dev = [
    "ruff>=0.3.0",
    "mypy>=1.9.0",
    "pytest>=8.1.0",
    "pytest-cov>=4.1.0",
    "ipykernel>=6.29.0",
]

# ------------------------------------------------------------------
# 4. TOOL CONFIGURATION
# ------------------------------------------------------------------

[tool.hatch.version]
path = "src/core_analise/__init__.py"

[tool.ruff]
line-length = 100
target-version = "py312"

[tool.ruff.lint]
select = ["E", "F", "W", "I", "B", "UP", "PL"]
ignore = []

[tool.ruff.format]
quote-style = "double"
indent-style = "space"

[tool.pytest.ini_options]
addopts = "--cov=src --cov-report=term-missing"
testpaths = ["tests"]
pythonpath = ["src"]

[tool.mypy]
python_version = "3.12"
warn_return_any = true
warn_unused_configs = true
disallow_untyped_defs = true

[[tool.mypy.overrides]]
module = ["pandas.*", "scipy.*"]
ignore_missing_imports = true
```

## 7. O Ciclo de Vida: Automação e CI/CD com GitHub Actions

O `pyproject.toml` define as regras, mas é o CI (Continuous Integration) que garante o cumprimento delas. Como estamos usando `uv`, nosso pipeline será drasticamente mais rápido e determinístico do que os pipelines tradicionais.

### O Prerrequisito: O Arquivo de Trava `uv.lock`

Antes de automatizar, você precisa garantir a reprodutibilidade. Enquanto o `pyproject.toml` define dependências abstratas (ex: `pandas>=2.0`), o `uv.lock` define o snapshot exato (ex: `pandas==2.2.1`).

**Regra de Ouro:** Em projetos de aplicação, o arquivo `uv.lock` **deve** ser comitado no Git.

```bash
uv lock
```

#### O Workflow (GitHub Actions)

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

#### Por que esta arquitetura é superior?

1. **Single Source of Truth:** O CI não tem flags mágicas. Ele apenas executa `uv run ruff check`. O Ruff, por sua vez, olha para o `pyproject.toml` para saber o que ignorar ou validar. Se você mudar a regra no TOML, o CI se atualiza automaticamente.
2. **Velocidade de Cache:** A action `setup-uv` com `enable-cache: true` evita baixar rodas (wheels) repetidamente, reduzindo o tempo de build de minutos para segundos.
3. **Segurança de Lockfile:** O comando `uv lock --check` falha o build se alguém alterar uma dependência no TOML mas esquecer de atualizar o lockfile, prevenindo divergências em produção.

## 8. Dockerfile Otimizado: Multi-stage & Cache Mounts

Seu pipeline de CI/CD termina aqui. Vamos criar uma imagem Docker de produção que é **leve** (apenas o runtime necessário) e **segura** (sem rodar como root).

Para isso, usamos duas técnicas avançadas:

1.  **Multi-stage Build:** Uma imagem "Builder" compila e instala tudo. Jogamos ela fora e copiamos apenas o ambiente virtual pronto para a imagem "Final".
2.  **Docker BuildKit Cache Mounts:** Configuramos o Docker para persistir o cache do `uv` entre builds diferentes. Se você mudar apenas uma lib, o `uv` não baixará as outras 50 novamente.

### O `Dockerfile`

Salve este arquivo na raiz do projeto.

```dockerfile
# ----------------------------------------------------------------------------------
# ESTÁGIO 1: BUILDER
# Objetivo: Instalar dependências e criar o ambiente virtual.
# Usamos uma imagem base completa para garantir que temos ferramentas de sistema.
# ----------------------------------------------------------------------------------
FROM python:3.12-slim AS builder

# Instala o 'uv' copiando o binário oficial da imagem da Astral
COPY --from=ghcr.io/astral-sh/uv:latest /uv /bin/uv

# Define variáveis de ambiente para otimizar o uv
ENV UV_COMPILE_BYTECODE=1
ENV UV_LINK_MODE=copy

WORKDIR /app

# OTIMIZAÇÃO DE CACHE DE CAMADA (LAYER CACHING):
# Copiamos APENAS os arquivos de definição primeiro.
# O Docker só invalidará esta etapa se o pyproject.toml ou uv.lock mudarem.
COPY pyproject.toml uv.lock ./

# INSTALAÇÃO COM CACHE MOUNT:
# 1. --mount=type=cache: O Docker cria um volume temporário para o cache do uv.
# 2. --frozen: Garante que instalaremos EXATAMENTE o que está no uv.lock (fail fast).
# 3. --no-dev: Não instala pytest, ruff, etc. em produção.
RUN --mount=type=cache,target=/root/.cache/uv \
    uv sync --frozen --no-dev

# Copia o restante do código fonte para dentro do ambiente (se necessário instalação via .pth)
# Ou prepara o código para cópia no próximo estágio.
COPY . .

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

#### Por que este Dockerfile é "Estado da Arte"?

1. `COPY --from=ghcr.io/astral-sh/uv`: Não instalamos o `uv` via pip ou curl. Copiamos o binário pré-compilado (Rust), o que é mais seguro e rápido.

2. `UV_COMPILE_BYTECODE=1`: O uv compila automaticamente os arquivos `.py` para `.pyc` durante a instalação. Isso melhora o tempo de inicialização (startup time) do container, pois o Python não precisa compilar na primeira execução.

3. `--mount=type=cache`: A mágica do BuildKit. Se você rodar `docker build` 10 vezes, o `uv` vai reutilizar os pacotes baixados anteriormente, fazendo builds subsequentes serem quase instantâneos.

4. `python:slim` **vs Alpine**: Evitamos Alpine (musl libc) para Python em produção devido a problemas de compatibilidade com rodas (wheels) binárias padrão (manylinux/glibc) e performance em runtime. A versão `slim`(Debian-based) é o equilíbrio ideal.

#### Comando de Build

Para garantir que o BuildKit ative os mounts de cache:

```bash
docker build -t meu-projeto:latest .
```

## 9. Orquestração Local: Docker Compose & Environment

Agora que temos uma imagem isolada, precisamos subir a aplicação conectada a um banco de dados real. Vamos usar o padrão moderno de especificação (Compose Specification).

### O Arquivo `.env` (Segurança)

Primeiro, **nunca** commite senhas no Git. Crie um arquivo `.env` na raiz (e adicione-o ao `.gitignore`).

```ini
# .env
POSTGRES_USER=admin
POSTGRES_PASSWORD=segredo_super_seguro
POSTGRES_DB=app_db
DATABASE_URL=postgresql://admin:segredo_super_seguro@db:5432/app_db
```

#### O Arquivo `compose.yml`

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
      test: ["CMD-SHELL", "pg_isready -U admin -d app_db"]
      interval: 5s
      timeout: 5s
      retries: 5
    ports:
      - "5432:5432" # Opcional: expõe para seu DBeaver/PgAdmin local

  # ----------------------------------------------------------------
  # SUA APLICAÇÃO PYTHON
  # ----------------------------------------------------------------
  app:
    build: .             # Usa o Dockerfile que criamos na Seção 8
    env_file:
      - .env
    ports:
      - "8000:8000"
    depends_on:
      db:
        condition: service_healthy # Só inicia o app quando o DB estiver VERDE

    # Dica Sênior: Para desenvolvimento local com "Hot Reload",
    # você pode sobrescrever o comando padrão.
    # command: uv run uvicorn core_analise.main:app --host 0.0.0.0 --reload

volumes:
  postgres_data:
```

#### Como Rodar

1. **Subir o ambiente:**

```bash
docker compose up --build
```

O `--build` força a recriação da imagem caso você tenha mudado dependências no `pyproject.toml`.

2. **Derrubar e limpar volumes (Reset total):**

```bash
docker compose down -v
```

## Conclusão do Tutorial

Parabéns. Você migrou de scripts frágeis para uma **Arquitetura de Software Profissional**:

1. **Definição:** `pyproject.toml` (PEP 621) como fonte da verdade.
2. **Gerenciamento:** uv para velocidade extrema e resolução de dependências.
3. **Qualidade:** Ferramentas (Ruff, Mypy) configuradas centralmente.
4. **Integração:** GitHub Actions com validação de lockfile.
5. **Entrega:** Docker Multi-stage builds leves e seguros.
6. **Orquestração:** Ambiente local reprodutível com Docker Compose.

Este é o padrão ouro de 2024/2025 para Python.
