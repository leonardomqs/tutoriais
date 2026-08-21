---
titulo: "pyproject.toml: fundamentos e referência"
tags: [python, pyproject, empacotamento, pep]
nivel: intermediario
atualizado: 2026-01-19
---

# `pyproject.toml`: fundamentos e referência

Como o empacotamento moderno de Python funciona por baixo — as PEPs que definem o formato,
a escolha do backend de build, os metadados de `[project]` e a centralização das configs
de ferramentas. Para o ciclo de entrega (CI, Docker, Compose), veja
[workflow completo](pyproject-toml-workflow-completo.md).

---

## 0) Modelo mental: o que é “pyproject.toml” hoje

`pyproject.toml` virou o **ponto central** porque separa claramente:

- **Frontend de build/instalação** (pip, `python -m build`, `uv`)
- **Backend de build** (hatchling, setuptools, flit_core, pdm-backend, `uv_build`)
- **Metadados padronizados do projeto** (`[project]`)
- **Config de ferramentas** (`[tool.*]`)

Essa divisão foi formalizada por PEPs: **517/518** (interface + deps mínimas do build) e **621** (metadados estáticos em `[project]`).

---

## 1) Fundamentos teóricos (o “porquê”)

### PEP 517: interface padrão de build (wheel/sdist)

Define uma **API mínima** para qualquer ferramenta (pip/build/uv) conseguir “perguntar” a um backend como gerar **wheel** e **sdist**, sem depender de executar `setup.py` arbitrário.

### PEP 518: `[build-system]` e build isolado

Define como declarar as **dependências mínimas de build** em `[build-system]`, permitindo build isolado e reprodutível (sem “funciona na minha máquina porque eu tinha X instalado”).

### PEP 621: metadados em `[project]`

Padroniza **metadados de pacote** (nome, versão, deps, autores etc.) para serem consumidos por ferramentas, reduzindo dependência de formatos específicos de cada tool.

Resultado prático: menos “código executável” (setup.py), mais **config declarativa** + **reprodutibilidade**.

---

## 2) Build system: backends modernos e o papel do `uv`

### Backends (resumo pragmático)

- **Hatchling**: backend moderno, extensível, bem “standards-first”.
- **Setuptools**: o legado ainda muito usado, útil quando você precisa de compat/complexidade (extensões, casos antigos).
- **Flit (`flit_core`)**: excelente para pacotes simples/puro-Python, pouca cerimônia.
- **PDM-Backend**: backend moderno alinhado com PEPs (517/621/660).
- **`uv_build`**: backend de build do ecossistema Astral/uv (com recomendação de upper-bound).

A **sintaxe de `[project]` é agnóstica** (PEP 621). O que muda entre backends é principalmente **config extra em `[tool.<backend>]`** e detalhes de build (inclusão de arquivos, versioning, hooks etc.).

### Onde o `uv` entra

`uv` é mais “DevEx” do que “backend”: ele atua como **gerenciador de projeto/deps/lock/sync** e também pode ser **frontend de build** — e, se você quiser, também **backend (`uv_build`)**. O `uv init` hoje já pode criar projeto usando `uv_build` por padrão e permite trocar via `--build-backend`.

Neste tutorial, vou usar **Hatchling** como backend “principal” (muito comum na indústria). Onde fizer sentido, mostro o equivalente no `uv_build`.

---

## 3) Anatomia do arquivo: `[project]` (PEP 621)

### Esqueleto mínimo saudável

```toml
[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[project]
name = "meu_pacote"
version = "0.1.0"
description = "Uma frase clara do que isso faz"
readme = "README.md"
requires-python = ">=3.11"
dependencies = [
  "httpx>=0.27",
]
```

Hatchling exige esse `[build-system]` e o valor de `build-backend`.

### Dependências obrigatórias vs opcionais (extras)

- **Obrigatórias**: `project.dependencies`
- **Opcionais** (extras publicáveis): `project.optional-dependencies`

```toml
[project]
dependencies = [
  "pydantic>=2.7",
  "typing-extensions>=4.10; python_version<'3.11'",
]

[project.optional-dependencies]
postgres = ["asyncpg>=0.29"]
redis    = ["redis>=5"]
cli      = ["typer>=0.12"]
```

Extras viram metadado do pacote (para consumidor instalar `meu_pacote[postgres]`). Isso é diferente de “deps de dev”.

### Versão dinâmica (do jeito certo)

Você tem 3 estratégias comuns:

1) **Versão estática** (`version = "1.2.3"`) — OK para internos simples
2) **Single-source** (arquivo no repo com a versão) — funciona, mas precisa disciplina
3) **VCS tags** (recomendado para libs): versão vem do Git tag e você evita drift

Com Hatch + plugin `hatch-vcs` (bem usado), fica assim:

```toml
[build-system]
requires = ["hatchling", "hatch-vcs"]
build-backend = "hatchling.build"

[project]
name = "meu_pacote"
dynamic = ["version"]

[tool.hatch.version]
source = "vcs"

[tool.hatch.version.raw-options]
local_scheme = "no-local-version"
```

- `dynamic = ["version"]` é o gatilho no `[project]`
- `tool.hatch.version` define de onde vem a versão (aqui: VCS)

---

## 4) Centralização de configs (a “killer feature”)

A regra: ferramenta *X* lê `pyproject.toml` em `[tool.X]` (ou variação). Isso elimina `.flake8`, `setup.cfg`, `pytest.ini`, `mypy.ini` espalhados.

### Ruff (linter + formatter)

Ruff suporta config via `pyproject.toml`.

```toml
[tool.ruff]
target-version = "py311"
line-length = 100
src = ["src"]
extend-exclude = ["build", "dist"]

[tool.ruff.lint]
select = ["E", "F", "I", "UP", "B", "SIM", "RUF"]
ignore = ["E501"] # se você usar o formatter do Ruff, geralmente ignora line-too-long

[tool.ruff.format]
quote-style = "double"
indent-style = "space"
```

### Pytest

Pytest suporta `pyproject.toml`; desde pytest 9 dá pra usar `[tool.pytest]` com tipos TOML nativos.

```toml
[tool.pytest]
minversion = "9.0"
addopts = ["-ra", "--strict-config", "--strict-markers"]
testpaths = ["tests"]
xfail_strict = true
filterwarnings = [
  "error",
]
```

### Mypy

Mypy suporta `pyproject.toml` em `[tool.mypy]` e overrides via `[[tool.mypy.overrides]]`.

```toml
[tool.mypy]
python_version = "3.11"
strict = true
warn_unused_configs = true
files = ["src", "tests"]
exclude = ["^build/", "^dist/"]

[[tool.mypy.overrides]]
module = ["tests.*"]
disallow_untyped_defs = false
```

---

## 5) Dependency Groups (PEP 735): dev/test/docs do jeito padrão

**PEP 735** padroniza `[dependency-groups]` para deps que **não** entram nos metadados do pacote (diferente de extras) e servem para **dev/test/docs/typing** etc.

```toml
[dependency-groups]
dev  = ["ruff>=0.8", "mypy>=1.10", "pytest>=9.0"]
docs = ["mkdocs>=1.6", "mkdocs-material>=9.5"]

test = [
  { include-group = "dev" },
  "pytest-cov>=5.0",
]
```

- `include-group` é parte da spec (reuso/composição).
- `uv` suporta isso bem e o grupo `dev` é “special-cased” (sincroniza por padrão).

Exemplos de uso com `uv` (prático, sem mágica):

- Instalar grupos no modo “pip interface”: `uv pip install --group docs`
- Sync do projeto e deps: `uv sync` (comportamento de `dev` e flags como `--no-dev`).

---

## 6) Exemplo “full-stack”: um `pyproject.toml` que eu usaria em projeto grande

```toml
# -------------------------
# Build backend (Hatchling)
# -------------------------
[build-system]
requires = ["hatchling>=1.26", "hatch-vcs>=0.4"]
build-backend = "hatchling.build"

# -------------------------
# Project metadata (PEP 621)
# -------------------------
[project]
name = "acme-analytics"
dynamic = ["version"]
description = "Analytics toolkit for Acme's data platform"
readme = "README.md"
requires-python = ">=3.11"
license = { text = "MIT" }
authors = [
  { name = "Acme Data Team", email = "data@acme.com" }
]
keywords = ["analytics", "etl", "observability"]
classifiers = [
  "Development Status :: 4 - Beta",
  "Programming Language :: Python :: 3",
  "Programming Language :: Python :: 3 :: Only",
  "Programming Language :: Python :: 3.11",
  "Typing :: Typed",
]
dependencies = [
  "pydantic>=2.7",
  "httpx>=0.27",
  "rich>=13.8",
]

[project.optional-dependencies]
postgres = ["asyncpg>=0.29"]
redis    = ["redis>=5"]
cli      = ["typer>=0.12"]

[project.urls]
Repository = "https://github.com/acme/acme-analytics"
Documentation = "https://acme.github.io/acme-analytics"
Changelog = "https://github.com/acme/acme-analytics/releases"

[project.scripts]
acme = "acme_analytics.cli:main"

# -------------------------
# Versioning (VCS tags)
# -------------------------
[tool.hatch.version]
source = "vcs"

[tool.hatch.version.raw-options]
local_scheme = "no-local-version"

# -------------------------
# Build configuration
# (assumindo src-layout)
# -------------------------
[tool.hatch.build.targets.wheel]
packages = ["src/acme_analytics"]

[tool.hatch.build.targets.sdist]
exclude = [
  "/.github",
  "/.venv",
  "/.ruff_cache",
  "/.mypy_cache",
  "/dist",
  "/build",
]

# -------------------------
# Dependency Groups (PEP 735)
# -------------------------
[dependency-groups]
dev = [
  "ruff>=0.8",
  "mypy>=1.10",
  "pytest>=9.0",
  "pytest-cov>=5.0",
  "pre-commit>=3.7",
  "types-requests",
]

docs = [
  "mkdocs>=1.6",
  "mkdocs-material>=9.5",
  "mkdocstrings[python]>=0.25",
]

test = [
  { include-group = "dev" },
  "pytest-xdist>=3.6",
]

# -------------------------
# Ruff
# -------------------------
[tool.ruff]
target-version = "py311"
line-length = 100
src = ["src"]
extend-exclude = ["build", "dist"]

[tool.ruff.lint]
select = ["E", "F", "I", "UP", "B", "SIM", "RUF"]
ignore = ["E501"]

[tool.ruff.format]
quote-style = "double"
indent-style = "space"

# -------------------------
# Pytest (native TOML types)
# -------------------------
[tool.pytest]
minversion = "9.0"
addopts = ["-ra", "--strict-config", "--strict-markers"]
testpaths = ["tests"]
xfail_strict = true
filterwarnings = ["error"]

# -------------------------
# Mypy
# -------------------------
[tool.mypy]
python_version = "3.11"
strict = true
warn_unused_configs = true
files = ["src", "tests"]
exclude = ["^build/", "^dist/"]

[[tool.mypy.overrides]]
module = ["tests.*"]
disallow_untyped_defs = false
```

---

## Bônus rápido: se você preferir o padrão do `uv` (uv_build)

Troque o `[build-system]` para:

```toml
[build-system]
requires = ["uv_build>=0.9.26,<0.10.0"]
build-backend = "uv_build"
```

Isso é exatamente o que o `uv init` mostra e o `uv` recomenda upper-bound para manter builds estáveis.
