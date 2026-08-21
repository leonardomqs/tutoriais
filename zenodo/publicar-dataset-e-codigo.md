---
titulo: "Publicar dataset e código no Zenodo com DOI"
tags: [zenodo, doi, dados-abertos, github, pesquisa, orcid]
nivel: intermediario
atualizado: 2026-08-21
---

# Publicar dataset e código no Zenodo com DOI

Ao final deste tutorial você terá dois registros públicos e citáveis — um para os dados,
outro para o código — cada um com seu DOI, vinculados entre si e prontos para entrar na
declaração de disponibilidade de dados do seu artigo.

> **Pré-requisitos:** uma conta ORCID (crie em [orcid.org](https://orcid.org) se ainda não
> tem) e, para arquivar o código, um repositório público no GitHub.
>
> **Tempo:** 40–60 min no primeiro depósito; cerca de 10 min nos seguintes.

Depois de depositar, o passo seguinte é [preparar a submissão do
artigo](preparar-submissao-do-artigo.md): declaração de disponibilidade, referências e a
auditoria de reprodução.

---

## 1. Por que isso deixou de ser opcional

Publicar os dados de uma pesquisa já foi gesto de boa vontade. Três forças mudaram isso:

- **Periódicos** exigem declaração de disponibilidade de dados na submissão. Quando não
  exigem, o parecerista pergunta — e a ausência vira objeção formal.
- **Agências de fomento** incorporaram ciência aberta aos editais, muitas vezes com
  exigência explícita de repositório com identificador persistente.
- **Reprodutibilidade:** em áreas com forte componente computacional, um pipeline que
  terceiros não conseguem reexecutar é tratado como não verificável, e isso pesa na
  decisão editorial.

Há também o argumento positivo, que costuma ser esquecido: um dataset bem documentado é um
**produto acadêmico independente**, com identificador próprio e citável por outros grupos.
Entra no currículo e no ORCID como item separado da dissertação.

O problema concreto que o Zenodo resolve: link de Google Drive ou de GitHub no artigo não
funciona como evidência, porque links quebram e não há garantia de qual versão o leitor vai
encontrar. A saída é depositar em um repositório que emita DOI e congele versões.

## 2. O que é o Zenodo

Repositório aberto de propósito geral criado no âmbito do OpenAIRE e mantido pelo CERN,
rodando sobre o InvenioRDM. Aceita datasets, código, figuras, apresentações, preprints e
relatórios. Os arquivos ficam no Data Centre do CERN, com compromisso de preservação de
longo prazo, e o serviço é aberto a pesquisadores de qualquer país.

**O que ele não é:**

- Não é periódico. Não há revisão por pares do que se deposita — a curadoria é inteiramente
  sua.
- Depositar não substitui o artigo nem conta como publicação.
- A qualidade do registro fica visível. Um dataset sem descrição, sem licença e com um
  arquivo chamado `dados_final_v3_ok.zip` comunica algo sobre o rigor do trabalho.

**Custo:** não há taxa. Depósito, DOI, armazenamento e preservação são gratuitos.

## 3. Comparação com as alternativas

| Opção | Custo | Acesso do leitor | Limite | Observação |
| --- | --- | --- | --- | --- |
| **Zenodo** | Gratuito | Aberto a todos | 50 GB por registro (mais sob pedido) | DOI, versionamento, integração com GitHub e ORCID |
| IEEE DataPort *Standard* | Gratuito | **Exige assinatura** | 2 TB | Parece aberto, mas está atrás de paywall |
| IEEE DataPort *Open Access* | Pago, por dataset | Aberto | 2 TB | Cobrança separada da APC do artigo; CC BY imposta |
| figshare | Gratuito (básico) | Aberto | 20 GB | Recomendado pelo próprio IEEE como alternativa |
| Dryad | Taxa por depósito | Aberto | Varia | Curadoria editorial; forte em ciências da vida |
| GitHub sozinho | Gratuito | Aberto | 100 MB por arquivo | Sem DOI, sem garantia de permanência |
| Drive / OneDrive | Gratuito | Depende do link | Variável | Não é repositório de pesquisa; o link quebra |

> ⚠️ **A confusão mais cara do IEEE DataPort.** O *Standard Dataset* é gratuito para
> depositar, mas exige assinatura para ser **acessado**. O *Open Access Dataset* é o que
> fica realmente aberto — e custa quase dois mil dólares por dataset, independente da APC
> do artigo. Existem artigos que afirmam dados "publicamente disponíveis" apontando para
> registros Standard: quando o parecerista esbarra no paywall, a alegação de
> reprodutibilidade cai no pior momento possível. O próprio IEEE lista figshare, Zenodo e
> Dryad como alternativas — usar o Zenodo não é desvio da norma.

> Valores e limites conferidos em agosto de 2026. Reconfirme nas páginas oficiais antes de
> qualquer decisão orçamentária.

## 4. Cinco conceitos antes de clicar em qualquer coisa

### 4.1. DOI

Identificador permanente que aponta para um **objeto**, não para um lugar — a
infraestrutura redireciona para onde o objeto estiver. Formato:
`10.5281/zenodo.1234567`. É esse número que entra na referência, nunca a URL que aparece na
barra do navegador.

### 4.2. Concept DOI e Version DOI

Ao publicar pela primeira vez, o Zenodo registra **dois** DOIs:

| DOI | Aponta para | Use quando |
| --- | --- | --- |
| **Version DOI** | Uma versão específica, congelada | Citar em artigo que reporta números — é o que permite reprodução exata |
| **Concept DOI** | O conjunto de todas as versões; resolve sempre para a mais recente | Citar em lugares que não podem ser editados depois (ver seção 8) |

A cada nova versão nasce um Version DOI novo, e o Concept passa a apontar para ele. Você
corrige um erro dois anos depois sem quebrar nenhuma citação existente.

> **Distinção que economiza dor de cabeça:** conteúdo novo (arquivos) exige **New version**,
> e isso gera um DOI novo. Já os **metadados** — criadores, descrição, palavras-chave,
> vínculos — podem ser corrigidos em registro já publicado com **Edit** → **Publish**, sem
> mudar o DOI e sem criar versão.

### 4.3. Licença

Todo arquivo público precisa de licença declarada. Recomendação:

| Tipo de material | Licença |
| --- | --- |
| Dados | **CC BY 4.0** — reuso amplo com atribuição |
| Código | **MIT** ou Apache 2.0 (GPL se quiser garantir derivados abertos) |
| Quando o financiador exigir domínio público | CC0 |

### 4.4. Níveis de acesso

Aberto, embargo (abre em data futura), restrito (acesso sob aprovação) e fechado.

> ⚠️ **Não use embargo durante a avaliação.** Se o parecerista não consegue baixar, o
> argumento de reprodutibilidade é derrubado exatamente no momento em que está sendo
> julgado. A exceção razoável é dado sob acordo de confidencialidade com empresa: nesse
> caso use "restrito" e explique a condição tanto no registro quanto na declaração do
> artigo.

### 4.5. Communities

Coleções temáticas ou institucionais. Se você faz parte de um grupo de pesquisa, vale criar
a comunidade do grupo para reunir dados, código e materiais de todas as dissertações num
lugar só.

---

## 5. Preparando o pacote antes de subir

Esta etapa é o que separa um depósito útil de um depósito decorativo. Faça tudo aqui
**antes** de abrir o navegador.

### 5.1. Estrutura e artefatos

Organize uma pasta de *staging* com estrutura previsível.

> ⚠️ No Windows, mantenha essa pasta **fora de diretórios sincronizados**. O OneDrive trava
> arquivos durante o sync e quebra tanto a compactação quanto a restauração de ambiente. Se
> você usa sincronização no Linux, veja [Insync: configuração e
> conflitos](../insync/configuracao-e-conflitos.md) para pausar o sync antes de operações
> grandes.

**Separe dataset e código em dois registros.** Eles têm ciclos de vida diferentes,
versionamento independente, e assim você tem dois DOIs citáveis.

Além dos dados brutos, empacote como artefatos próprios — um zip por artefato lógico, nunca
um monólito:

- **Manifests de partição por semente/repetição** — quem entra em treino, validação e teste
  em cada execução. Costuma ser o artefato mais valorizado em parecer: permite que
  trabalhos futuros comparem *nas mesmas partições* e que qualquer pessoa confira que não há
  vazamento entre conjuntos.
- **Resultados por execução** — métricas por run, configurações e, principalmente,
  **predições por instância**. São elas que tornam as análises do artigo recomputáveis por
  terceiros.
- **CHECKSUMS** — ver 5.3.

### 5.2. O README é obrigatório

Um dataset sem README é irreprodutível por mais organizado que esteja. Ele precisa
responder:

- O que é o conjunto e para que serve.
- Como os dados foram adquiridos: equipamento, condições, protocolo, período.
- O que significa cada coluna, rótulo e pasta.
- Como se dividem treino, validação e teste, e se a divisão é fixa ou reproduzível por
  script.
- **Limitações e vieses conhecidos** — declare antes que o parecerista descubra sozinho.
- Como citar, com o DOI já formatado.

### 5.3. Compactação e checksums

O limite é de 50 GB por registro, com cotas maiores sob pedido. Para muitos arquivos
pequenos, use um ZIP por split ou por classe.

Gere os checksums **antes** de subir e **publique o arquivo de checksums dentro do próprio
depósito**, com uma linha de descrição por pacote:

```text
<sha256>  dataset_variante_A.zip          (48.973 imagens, 8 classes, PNG)
<sha256>  manifests_10sementes.zip        (particoes por semente, validadas)
<sha256>  resultados_por_execucao.zip     (metricas + predicoes por instancia)
```

No Linux:

```bash
find . -type f -exec sha256sum {} + > CHECKSUMS.sha256
```

No Windows, pelo PowerShell:

```powershell
Get-FileHash -Algorithm SHA256 *.zip | Format-List
```

> O `sha256sum -r` que circula em tutoriais antigos **não existe** no coreutils do Linux.

### 5.4. Verificação ética e legal

Antes de tornar público, confirme:

- [ ] Sem dados pessoais identificáveis — rostos, placas, nomes, documentos.
- [ ] Sem EXIF com GPS não intencional nas imagens.
- [ ] Autorização por escrito para dados de terceiros ou de empresa.
- [ ] Se houve comitê de ética, os termos aprovados permitem o compartilhamento nesta forma.

> ⚠️ Publicação é irreversível na prática. Mesmo que você remova o arquivo depois, ele pode
> já ter sido baixado e indexado. Auditar antes é a única proteção real.

---

## 6. Criando a conta

1. Em [zenodo.org](https://zenodo.org), clique em **Sign up** e **entre com o ORCID**. Isso
   associa seu identificador desde o início, e o ORCID passa a aparecer automaticamente nos
   registros.
2. No seu perfil, abra **Linked accounts** e conecte também o **GitHub** — é o que habilita
   o arquivamento automático da seção 8.

## 7. Publicando o dataset

1. **New upload.** Arraste os arquivos e aguarde o upload concluir — a página precisa ficar
   aberta durante o envio.
2. **DOI:** se você precisa do número antes de publicar, para já citá-lo no manuscrito, use
   **Reserve DOI**.
3. **Resource type:** `Dataset`. (Use `Software` no registro do código.)
4. **Título** descritivo e autossuficiente: domínio, tipo de dado, escala. Nada de "Dados da
   dissertação".
5. **Creators:** veja o aviso abaixo — é o campo que mais gera apontamento de parecer.
6. **Description:** versão resumida do README, incluindo a **lista arquivo a arquivo do que
   há no depósito**, as limitações, o link do repositório de código e a frase
   `article DOI to be added upon publication`.
7. **License** (seção 4.3) e **Keywords** — use os termos que um pesquisador buscaria, não
   apenas os que já estão no título.
8. **Related works:**
   - `Is supplement to` → DOI do artigo, quando existir.
   - `Is supplemented by` → URL do GitHub e DOI do snapshot de código.

   Os vínculos precisam existir **nos dois registros, nos dois sentidos**. É isso que conecta
   os objetos nos índices.
9. **Communities:** a do seu grupo, se houver. Revise tudo e clique em **Publish**.

> ⚠️ **Creators = autores do artigo. Exatamente. Nos dois registros.**
>
> Mesmos nomes, **mesma ordem do artigo**, cada um com **afiliação e ORCID**.
>
> Este é um erro que acontece de verdade: um coautor entra no artigo mas não no depósito de
> código. O parecerista resolve os dois DOIs no DataCite, compara com a lista de autores do
> manuscrito e transforma a diferença em apontamento formal — divergência de autoria entre
> artigo e artefato costuma gerar consulta editorial.
>
> A correção é indolor (**Edit** no registro publicado, inserir o autor na posição correta,
> **Publish**, DOI inalterado), desde que você descubra antes da submissão. Confira os dois
> registros com os olhos do parecerista em `https://api.datacite.org/dois/<DOI>`, que mostra
> exatamente o que ele verá. O DataCite pode levar horas para refletir edições — a fonte
> imediata é a própria página do registro.

> ⚠️ **Antes de clicar em Publish:** arquivos de registro publicado **não podem ser
> alterados**. Correções de conteúdo exigem nova versão, com DOI novo. Metadados continuam
> editáveis sem mudar o DOI. Confira com calma — a estabilidade da citação é uma virtude que
> cobra atenção na largada.

---

## 8. Arquivando o código a partir do GitHub

Configurada uma vez, cada release do GitHub vira automaticamente um registro arquivado com
DOI próprio. Mas há um detalhe que costuma ser omitido e que estraga o primeiro registro:

> ⚠️ **Os metadados do registro automático vêm do repositório — prepare-os ANTES da
> release.** O webhook do Zenodo monta o registro do código a partir do `.zenodo.json`
> (preferido) ou do `CITATION.cff` presentes no repositório **no momento da release**. Sem
> esses arquivos, os creators saem errados ou incompletos. É assim que um registro de código
> nasce com 4 dos 5 autores. Dá para corrigir depois com **Edit**, mas o certo é nascer
> certo.

### 8.1. Preparar o repositório

O repositório precisa ser **público** e conter:

- O código.
- O ambiente pinado — por exemplo `pyproject.toml` mais o lockfile do `uv`. Veja
  [`pyproject.toml`: fundamentos](../python/pyproject-toml-fundamentos.md) e
  [CI e Docker com `uv`](../python/ci-e-docker-com-uv.md).
- `README.md` e `LICENSE`.
- `CITATION.cff` e `.zenodo.json`.
- O script de reprodução (descrito em [preparar a submissão do
  artigo](preparar-submissao-do-artigo.md)).

### 8.2. O `.zenodo.json`

```json
{
  "title": "nome-do-repo: descricao curta",
  "license": "MIT",
  "upload_type": "software",
  "creators": [
    {
      "name": "Sobrenome, Nome",
      "affiliation": "Instituicao",
      "orcid": "0000-0000-0000-0000"
    }
  ],
  "related_identifiers": [
    {
      "identifier": "10.5281/zenodo.XXXXXXX",
      "relation": "isSupplementTo",
      "resource_type": "dataset"
    }
  ]
}
```

### 8.3. O `CITATION.cff`

Atenção a uma armadilha: os autores aparecem em **dois blocos** — em `authors` e, dentro de
`preferred-citation`, em outro `authors`. **Atualize os dois.** É desse arquivo que o GitHub
tira o botão "Cite this repository".

### 8.4. Ativar a integração e publicar

1. No Zenodo, vá em **Linked accounts** → **GitHub** e ative o repositório. Se ele não
   aparecer na lista, use **Sync now**.
2. No GitHub, crie a release com versão semântica (`v1.0.0`).
3. O Zenodo captura o snapshot e emite o DOI sozinho — nada manual. Alguns minutos depois,
   confirme o DOI novo na página do registro.
4. Complete no registro o que faltar (os vínculos da seção 7) e cole o **badge de DOI** no
   topo do `README.md`.

> ⚠️ **Duas lições de versionamento que custam caro:**
>
> **(a) As notes de release são congeladas.** Não cite nelas o DOI de uma *versão específica*
> do dataset: quando o dataset ganhar versão nova, a nota antiga vira ponteiro errado que
> você não consegue editar de forma rastreável. Nas notes, use o **Concept DOI** ou não cite
> DOI nenhum.
>
> **(b) Cada release citada no artigo dispara um ciclo de sincronização.** Artigo, `README.md`
> e `CITATION.cff` devem passar a citar a **mesma** versão, commit e DOI, no mesmo dia. É um
> apontamento comum de parecer encontrar o README dizendo `v1.0.1` enquanto o artigo diz
> `v1.0.3`.

---

## 9. Erros comuns no depósito

| Erro | Por que é um problema | O que fazer |
| --- | --- | --- |
| Depositar como *Standard* no DataPort e dizer que os dados são públicos | O parecerista esbarra no paywall e a alegação cai | Use o Zenodo, ou pague o Open Access do DataPort |
| Citar URL do GitHub em vez de DOI | O link quebra e não há garantia de versão | Arquive a release no Zenodo e cite o DOI |
| Publicar sem README | Acessível, porém inutilizável | README pronto antes de subir (5.2) |
| Embargo durante a revisão | O parecerista não verifica nada | Publique aberto já na submissão |
| Esquecer os vínculos entre registros | Objetos órfãos nos índices | *Related works* nos dois sentidos |
| ZIP monolítico de dezenas de GB | Download inviável | Dividir por split ou por classe |
| Dados pessoais ou EXIF não auditados | Problema ético-jurídico irreversível | Auditar antes (5.4) |
| Creators diferentes dos autores do artigo | Gera consulta editorial | Espelhar a lista nos **dois** registros e conferir pela API do DataCite |
| Release sem `.zenodo.json` / `CITATION.cff` | O registro de código nasce com creators errados | Preparar os dois arquivos **antes** da primeira release |
| DOI de versão nas notes da release | As notes congelam e viram ponteiro errado | Concept DOI, ou nada |

---

## Próximo passo

Com os dois registros publicados e vinculados, siga para [preparar a submissão do
artigo](preparar-submissao-do-artigo.md), que cobre a declaração de disponibilidade de
dados, as referências no formato IEEE e a auditoria de reprodução em ambiente limpo.

## Referências

- [zenodo.org](https://zenodo.org) — a plataforma
- [help.zenodo.org](https://help.zenodo.org) — documentação oficial
- [orcid.org](https://orcid.org) — criar seu ORCID
- `https://api.datacite.org/dois/<DOI>` — ver o registro como o parecerista vê
