# 📚 Tutoriais

Caderno de anotações técnicas de **Leonardo Garcia Marques**.

Cada arquivo aqui é um passo a passo que já foi executado de verdade — configuração de
ambiente, ferramenta nova, ou a solução de um erro que custou tempo. A ideia é simples:
**resolveu uma vez, documenta; da próxima, é só consultar.**

> Tudo em português, focado em **Windows 11 + WSL2 + Python + dados**.

---

## 🗂️ Índice

### 🖥️ Windows & Terminal

| Tutorial | Sobre |
| --- | --- |
| [Instalar o PowerShell 7](windows/powershell-7.md) | Por que trocar o PowerShell 5.1 pelo `pwsh`, instalação via `winget` e como deixá-lo padrão no Windows Terminal. |

### 🐧 WSL & Linux

| Tutorial | Sobre |
| --- | --- |
| [Estação de trabalho de IA](wsl/estacao-de-trabalho-ia.md) | Guia longo, do zero: Dev Drive (ReFS), WSL2 no disco rápido, Docker Desktop, Miniconda + PyTorch com GPU e VS Code remoto. |
| [Clonar uma distro WSL](wsl/clonar-distro.md) | `wsl --export` / `--import` para criar um ambiente de laboratório isolado, sem sujar o ambiente de trabalho. |

### 🐍 Python

| Tutorial | Sobre |
| --- | --- |
| [`pyproject.toml` — fundamentos](python/pyproject-toml-fundamentos.md) | As PEPs (517/518/621/735), escolha de backend de build, metadados, versionamento por tag Git e centralização de configs (Ruff, pytest, mypy). |
| [CI e Docker com `uv`](python/ci-e-docker-com-uv.md) | O que vem depois do TOML: `uv.lock`, GitHub Actions, `Dockerfile` multi-stage com cache mount e Docker Compose com healthcheck. |
| [Monorepo com `uv` e Ruff](python/uv-workspaces-monorepo.md) | Workspaces do `uv`: um `uv.lock` e um `.venv` para vários `apps/` e `libs/`, com formatação automática no VS Code. |

### 🔀 Git

| Tutorial | Sobre |
| --- | --- |
| [Comandos essenciais](git/comandos-essenciais.md) | Referência rápida: básicos, sincronização, branches, desfazer alterações e comandos avançados (`stash`, `rebase`, `cherry-pick`). |
| [Configurar Git no Linux com SSH](git/configuracao-linux-ssh.md) | Instalação, identidade dos commits, geração de chave `ed25519` e conexão com o GitHub via SSH. |

### 🗄️ Banco de dados

| Tutorial | Sobre |
| --- | --- |
| [DBeaver — configuração inicial](dbeaver/configuracao-inicial.md) | Excluir o DBeaver do Defender (performance) e desligar a telemetria. |

### 🧰 Ferramentas

| Tutorial | Sobre |
| --- | --- |
| [Doppler CLI no WSL](doppler/instalacao-cli-wsl.md) | Instalação, login, `doppler setup` e como rodar scripts com segredos injetados, sem `.env` local. |
| [Insync (OneDrive no Linux)](insync/configuracao-e-conflitos.md) | Licenciamento, estratégia de resolução de conflitos, reset completo e boas práticas para não perder arquivo. |

---

## 🔧 Resolvendo um erro?

Atalho para as anotações de *troubleshooting*, organizadas pela mensagem de erro:

| Sintoma / mensagem | Onde | Solução |
| --- | --- | --- |
| `Temporary failure in name resolution` — ping em IP funciona, em domínio não | WSL | [Erro de DNS no WSL](wsl/erro-dns.md) |
| `Error while loading conda entry point` (`No module named 'dotenv'` / `'tqdm'`) | WSL + Conda | [Entry points do Conda quebrados](wsl/erro-conda-entry-points.md) |
| `Public Key Retrieval is not allowed` ao conectar no MySQL 8 | DBeaver | [Erro de conexão MySQL 8](dbeaver/erro-public-key-retrieval.md) |

---

## 🧭 Trilhas sugeridas

Ordem recomendada quando o objetivo é maior que um tutorial só.

<details>
<summary><b>Montar a máquina de desenvolvimento do zero (Windows + IA)</b></summary>

1. [Instalar o PowerShell 7](windows/powershell-7.md) — terminal decente primeiro.
2. [Estação de trabalho de IA](wsl/estacao-de-trabalho-ia.md) — Dev Drive, WSL2, Docker e GPU.
3. [Configurar Git no Linux com SSH](git/configuracao-linux-ssh.md) — identidade e acesso ao GitHub.
4. [Clonar uma distro WSL](wsl/clonar-distro.md) — um ambiente de laboratório separado do de trabalho.

</details>

<details>
<summary><b>Estruturar um projeto Python moderno</b></summary>

1. [`pyproject.toml` — fundamentos](python/pyproject-toml-fundamentos.md) — entender o formato antes de copiar template.
2. [Monorepo com `uv` e Ruff](python/uv-workspaces-monorepo.md) — organizar `apps/` e `libs/`.
3. [CI e Docker com `uv`](python/ci-e-docker-com-uv.md) — automatizar a qualidade e empacotar para deploy.
4. [Doppler CLI no WSL](doppler/instalacao-cli-wsl.md) — tirar os segredos do código.

</details>

---

## 📐 Convenções

Regras que mantêm o repositório navegável conforme ele cresce:

| Item | Regra | Exemplo |
| --- | --- | --- |
| **Pastas** | Uma por assunto — nome da ferramenta ou do tema, em minúsculas | `wsl/`, `python/`, `dbeaver/` |
| **Arquivos** | `kebab-case.md`, sem acentos, sem prefixo `tutorial_` e sem repetir o nome da pasta | `clonar-distro.md`, não `tutorial_wsl_clonando.md` |
| **Troubleshooting** | Prefixo `erro-` + a causa, não o sintoma genérico | `erro-public-key-retrieval.md` |
| **Imagens** | Sempre em `assets/` dentro da pasta do assunto, com nome descritivo | `wsl/assets/erro-dns.png` |
| **Links** | Sempre relativos, para funcionarem no GitHub e no editor local | `[texto](../python/uv-workspaces-monorepo.md)` — nunca URL absoluta do GitHub |

Estrutura resultante:

```text
tutoriais/
├── README.md              # este índice — a porta de entrada
├── CONTRIBUTING.md        # como adicionar um tutorial novo
├── LICENSE                # CC BY 4.0
├── .editorconfig          # UTF-8, indentação e espaços em branco
├── .gitattributes         # finais de linha normalizados (Windows ↔ WSL)
├── .gitignore
├── _template/
│   └── tutorial.md        # ponto de partida para um arquivo novo
└── <assunto>/             # dbeaver, doppler, git, insync, python, windows, wsl
    ├── <tutorial>.md
    ├── erro-<causa>.md
    └── assets/            # imagens daquele assunto
```

---

## ➕ Adicionando um tutorial

```bash
cp _template/tutorial.md <assunto>/<nome-do-tutorial>.md
```

Preencha o *front matter*, escreva o conteúdo e **adicione a linha correspondente no índice
acima** — o README é o mapa; um tutorial fora dele é um tutorial perdido.

O passo a passo completo, com o padrão de escrita, está em [CONTRIBUTING.md](CONTRIBUTING.md).

---

## 📄 Licença

Conteúdo publicado sob [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/deed.pt-br) —
use, adapte e compartilhe, apenas mantendo o crédito.
