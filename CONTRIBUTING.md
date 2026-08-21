# Como contribuir

Este repositório cresce por acúmulo: toda vez que uma configuração dá trabalho ou um erro
custa tempo, vira um arquivo aqui. Para que ele continue navegável depois de 50 arquivos,
o padrão abaixo precisa ser seguido.

Contribuições de fora são bem-vindas — inclusive, e principalmente, as pequenas.

## Antes de tudo: você não precisa escrever um tutorial

A contribuição mais valiosa costuma ser a mais simples. Se um comando não funcionou, um
passo mudou de lugar na interface, ou um trecho ficou confuso, **abra uma issue**. Quem
está aprendendo enxerga as lacunas que quem escreveu não consegue mais ver.

Ao abrir uma issue, ajuda muito incluir:

- Qual tutorial e qual seção.
- O que você esperava que acontecesse e o que aconteceu.
- A mensagem de erro literal, se houve uma.
- Seu sistema e as versões envolvidas (`wsl --version`, `python --version`, etc.).

## Enviando um pull request

1. Faça um *fork* do repositório.
2. Crie uma branch com nome descritivo: `git switch -c corrige-dns-wsl`.
3. Faça as alterações seguindo o padrão das seções abaixo.
4. Abra o PR explicando **o que mudou e por quê**. Se for correção, diga como você
   confirmou que a versão nova funciona.

Não há CI neste repositório: a revisão é manual. Um PR pequeno e focado é revisado rápido;
um que mistura cinco assuntos, não.

---

## 1. Criar o arquivo

```bash
cp _template/tutorial.md <assunto>/<nome-do-tutorial>.md
```

Se o assunto ainda não existe, crie a pasta — **uma pasta por assunto desde o primeiro
arquivo**. É o que mantém o caminho previsível (`zenodo/`, `doppler/`, `windows/`) e evita
que o arquivo fique escondido dentro de uma pasta que não tem a ver com ele.

## 2. Nomear

| Regra | ✅ | ❌ |
| --- | --- | --- |
| `kebab-case`, minúsculas, sem acentos | `clonar-distro.md` | `Clonando_Imagem.md` |
| Sem prefixo `tutorial_` — tudo aqui é tutorial | `powershell-7.md` | `tutorial_powershell_7.md` |
| Sem repetir o nome da pasta | `dbeaver/configuracao-inicial.md` | `dbeaver/dbeaver_configuracao.md` |
| Troubleshooting: prefixo `erro-` + **causa**, não sintoma | `erro-public-key-retrieval.md` | `erro-conexao.md` |
| Descreve o conteúdo, não a origem | `uv-workspaces-monorepo.md` | `introducao_uv.md` |

## 3. Front matter

Todo tutorial começa com este bloco:

```yaml
---
titulo: "Título legível do tutorial"
tags: [wsl, docker, gpu]
nivel: iniciante
atualizado: 2026-08-21
---
```

| Campo | Valores |
| --- | --- |
| `titulo` | A mesma frase do `#` no corpo do arquivo. |
| `tags` | Minúsculas, sem acento. Reaproveite tags existentes antes de inventar uma nova. |
| `nivel` | `iniciante`, `intermediario` ou `avancado`. |
| `atualizado` | `YYYY-MM-DD` da última revisão real do conteúdo. **Atualize ao editar.** |

> O GitHub renderiza esse bloco como uma tabelinha no topo da página — é intencional.

## 4. Escrever

O padrão de escrita que já funciona bem nos arquivos existentes:

- **Um `#` só por arquivo**, o título. Seções em `##`, subseções em `###`.
- **Contexto antes do comando.** Explique o *porquê* em uma ou duas frases antes de mandar
  o leitor rodar algo. É o que separa esta nota de um Stack Overflow qualquer.
- **Blocos de código com a linguagem declarada** — `bash`, `powershell`, `toml`, `ini`,
  `python`, `yaml`, `json`, `text`. Isso liga o *syntax highlighting*.
- **Placeholders sempre explícitos**: `seu_usuario`, `<nome-da-branch>`, `D:\Caminho`.
  Nunca deixe caminho ou usuário real da sua máquina no meio do comando.
- **Comandos destrutivos avisados.** `wsl --unregister`, `rm -rf`, `docker compose down -v`
  e afins levam um bloco de aviso antes:

  ```markdown
  > ⚠️ Este comando apaga a distro e tudo que estiver dentro dela. Exporte antes.
  ```

- **Sem segredos.** Nenhuma senha, token, chave ou string de conexão real. Se precisar
  ilustrar, use valor claramente falso (`segredo_super_seguro`).

### Tutoriais de troubleshooting

Esses seguem uma estrutura própria, porque são consultados sob pressão:

```markdown
# Erro: <mensagem exata do erro>

**Sintoma:** o que o usuário vê.
**Contexto:** onde acontece (SO, versão, ferramenta).

## Diagnóstico
Por que acontece.

## Solução
Os passos.
```

Colocar a **mensagem literal do erro** no arquivo é o que faz a busca do GitHub — e a do
Google — encontrar a anotação depois.

## 5. Imagens

- Ficam em `<assunto>/assets/`.
- Nome descritivo, `kebab-case`: `erro-dns.png`. Nunca `1770472110775.jpg`.
- Sempre com texto alternativo real, descrevendo o que a imagem mostra:

  ```markdown
  ![Terminal mostrando falha no ping para google.com mas sucesso para 8.8.8.8](assets/erro-dns.png)
  ```

- Prefira `.png` para captura de tela e recorte só a região relevante.
- Imagem é complemento, nunca substituta do texto: todo comando visível numa captura
  precisa também estar escrito em um bloco de código, para poder ser copiado.

## 6. Registrar no índice

**Este passo não é opcional.** Adicione a linha do tutorial na tabela da categoria certa em
[README.md](README.md) — e, se for troubleshooting, também na tabela
*"Resolvendo um erro?"*, com a mensagem de erro na primeira coluna.

Um tutorial que não está no índice é um tutorial que não será encontrado.

## 7. Commitar

Mensagens no imperativo, dizendo o que mudou:

```text
✅ Adiciona tutorial de configuração do Doppler no WSL
✅ Corrige comando de export no tutorial de clonagem de distro WSL
✅ Reestrutura pastas por assunto e cria índice no README

❌ Atualizações
```

---

## Checklist de revisão

Antes de fechar a edição:

- [ ] Front matter preenchido, com `atualizado` na data de hoje.
- [ ] Nome do arquivo segue o padrão da seção 2.
- [ ] Todo bloco de código tem a linguagem declarada.
- [ ] Comandos destrutivos têm aviso.
- [ ] Nenhum segredo, token ou caminho pessoal real no texto.
- [ ] Imagens em `assets/`, com nome descritivo e `alt` real.
- [ ] Links relativos, todos testados.
- [ ] Linha adicionada no índice do README.
