---
titulo: "Preparar a submissão: citar os artefatos e auditar a reprodução"
tags: [zenodo, doi, reprodutibilidade, latex, pesquisa]
nivel: avancado
atualizado: 2026-08-21
---

# Preparar a submissão: citar os artefatos e auditar a reprodução

Depositar os dados é metade do trabalho. A outra metade é fazer com que o artigo, o
repositório e os depósitos digam exatamente a mesma coisa — e provar, antes de submeter, que
os números impressos podem ser regenerados a partir do que você publicou.

> **Pré-requisito:** dataset e código já depositados. Se ainda não é o caso, comece por
> [Publicar dataset e código no Zenodo](publicar-dataset-e-codigo.md).
>
> **Tempo:** 1–2 h para a auditoria da seção 3, uma vez por submissão.

**Por que este tutorial existe.** O parecerista moderno — humano ou assistido por
ferramentas — de fato abre os DOIs, resolve os metadados no DataCite, confere se os
criadores do depósito são os autores do artigo, clona o commit citado e tenta regenerar os
números das tabelas. As orientações abaixo vêm de um ciclo real de submissão com cinco
rodadas de parecer em que os artefatos foram verificados a cada rodada, e cada uma dessas
checagens falhou pelo menos uma vez antes de ficar verde.

---

## 1. A declaração de disponibilidade de dados

Modelo adaptável. Repare que ele cita **versão, licença, release, commit e snapshot** — não
só um link:

```text
The dataset supporting this study is openly available in Zenodo at
https://doi.org/10.5281/zenodo.XXXXXXX (version 1.0.3, CC BY 4.0),
including the per-seed partition manifests and the per-run results
with per-instance predictions. The full pipeline and evaluation code
are available at https://github.com/USER/REPO (release v1.0.3, commit
abc1234); the release snapshot is archived at
https://doi.org/10.5281/zenodo.YYYYYYY, which preserves the exact
tree independently of the live repository.
```

Quatro detalhes fazem diferença no parecer:

1. **Versão** explícita — sem ela, não há reprodução exata.
2. **Licença** declarada — responde à pergunta antes que ela seja feita.
3. **DOI em vez de URL de navegador.**
4. **Commit curto + snapshot arquivado.** Esse é o que menos aparece em tutoriais e o que
   mais protege: ferramentas de avaliação chegam a enxergar *cache velho* do GitHub e
   apontar como irregular um repositório que está íntegro. O snapshot com DOI é a prova
   independente do estado exato do código naquele momento.

### 1.1. Referência no estilo IEEE

Dataset e software entram na lista de referências como itens próprios. Copie a citação já
formatada da própria página do registro no Zenodo — ela oferece o estilo IEEE pronto.

### 1.2. Um detalhe de LaTeX que vira apontamento

Carregue o `hyperref`:

```latex
\usepackage[hidelinks]{hyperref}
```

Sem ele, o PDF não tem links reais. Uma URL quebrada em duas linhas faz o visualizador
auto-detectar apenas o fragmento inicial (`https://github.com/`), e o clique cai em 404 —
justamente no link que deveria comprovar a disponibilidade dos dados.

Teste clicando em cada link no PDF final, antes de submeter.

### 1.3. Onde mais mencionar

- Na **metodologia**, com o DOI.
- No **abstract**, se a liberação dos dados for uma contribuição declarada — e, nesse caso,
  liste-a também entre as contribuições.
- No **Lattes** e no **ORCID**, como produção técnica.

---

## 2. O script de reprodução

O repositório deve conter um script — por exemplo `scripts/reproduce.py` — que, apontado
para o pacote de resultados **baixado do Zenodo**, regenera os números centrais do artigo:
tabelas principais, decomposições, testes estatísticos. Use apenas bibliotecas básicas, para
que ele sobreviva a mudanças de ambiente.

A declaração de disponibilidade deve dizer o que esse script regenera. É o que transforma
*"os dados estão disponíveis"* em *"as análises são recomputáveis"* — e o segundo é o
critério que os pareceristas efetivamente aplicam.

## 3. A auditoria *clean-room*

Em uma máquina ou pasta limpa, **sem login em nada** — é essa a condição do parecerista:

1. Baixe o código **pelo DOI do snapshot**, não pelo seu clone local, e os pacotes de
   resultados **pelo DOI do dataset**.
2. Confira os SHA-256 contra o `CHECKSUMS` que você publicou no depósito.
3. Confirme que o snapshot corresponde ao commit citado no artigo.
4. Restaure o ambiente pinado e rode o script de reprodução:

   ```bash
   uv sync
   uv run python scripts/reproduce.py --resultados ./resultados_por_execucao
   ```

5. Compare a saída, item a item, com uma lista dos números impressos no artigo. Automatize o
   `OK` / `FALHOU` — conferência manual de dezenas de números não é confiável.
6. Guarde o log e o resumo como registro da auditoria.

> **Por que vale a pena.** Numa auditoria real, o *clean-room* encontrou uma divergência que
> cinco rodadas de parecer não pegaram: uma soma de quadrados impressa no artigo tinha sido
> calculada sobre uma base de precisão diferente da depositada — predições por instância
> versus métricas por execução, que diferem na última casa decimal. O ambiente limpo pegou em
> minutos, e o artigo foi corrigido antes da submissão.
>
> **Regra do cânone:** quando duas bases legítimas divergem no arredondamento, o número
> impresso deve ser o que o auditor **consegue regenerar dos artefatos depositados**.

### 3.1. Armadilhas de execução no Windows

- **Rode fora do OneDrive.** Locks durante o sync quebram o `uv sync`. Veja
  [Insync: configuração e conflitos](../insync/configuracao-e-conflitos.md) se você usa
  sincronização no Linux.
- **PowerShell 5.1:** ferramentas que escrevem avisos no *stderr* — o `uv` faz isso —
  derrubam scripts que usam `$ErrorActionPreference = "Stop"`, mesmo quando o comando teve
  sucesso. Contorne envolvendo as chamadas nativas:

  ```powershell
  cmd /c "uv sync 2>&1"
  ```

  Ou, melhor, use o PowerShell 7, que trata isso de forma mais previsível — veja
  [Instalação do PowerShell 7](../windows/powershell-7.md).

---

## 4. Erros comuns na submissão

| Erro | Por que é um problema | O que fazer |
| --- | --- | --- |
| README ou `CITATION.cff` citando versão antiga | Vira "documentação pública dessincronizada" em parecer | Ciclo de sincronização a cada release citada no artigo |
| PDF sem `hyperref` | Link quebrado em duas linhas → 404 no clique | `hidelinks` e teste de clique no PDF final |
| Número impresso calculado em base diferente da depositada | O auditor não consegue regenerar → "divergência" | Auditoria da seção 3; cânone é a base depositada |
| Citar só o Concept DOI no artigo | Não fixa a versão que gerou os números | Version DOI, com o número da versão explícito |
| Creators divergentes entre artigo e depósitos | Consulta editorial | Conferir via `api.datacite.org/dois/<DOI>` |

---

## 5. Checklist antes de submeter

**Depósito:**

- [ ] Dataset organizado, README completo, checksums gerados **e publicados no depósito**
- [ ] Verificação de dados pessoais, EXIF e autorizações concluída
- [ ] Registro do dataset publicado, aberto e com licença; manifests de partição e
      resultados por execução incluídos
- [ ] `.zenodo.json` e `CITATION.cff` (os **dois** blocos de autores) no repositório
      **antes** da release
- [ ] Registro do código publicado via release; o snapshot corresponde ao commit citado
- [ ] Creators = autores do artigo, com ordem, afiliação e ORCID, **nos dois registros** —
      conferidos via DataCite
- [ ] Vínculos `Is supplement to` / `Is supplemented by` nos dois sentidos

**Artigo:**

- [ ] Artigo, README e `CITATION.cff` citam a **mesma** versão, commit e DOIs
- [ ] Declaração de disponibilidade com DOI, versão, licença, release, commit e snapshot
- [ ] Dataset e código na lista de referências, no formato IEEE copiado do registro
- [ ] Badge de DOI no README; `hyperref` no LaTeX e links testados com clique no PDF

**Verificação final:**

- [ ] Auditoria *clean-room* da seção 3 executada, 100% verde, log arquivado
- [ ] Abrir os DOIs em janela anônima e baixar sem login — é exatamente o que o parecerista
      fará

---

## 6. Sugestão de política para o grupo de pesquisa

Se você coordena ou participa de um grupo, vale transformar isso em prática combinada em vez
de esforço individual:

- Todo aluno cria ORCID no primeiro semestre.
- O grupo mantém uma *community* no Zenodo, e todos os depósitos são submetidos a ela.
- Depósito de dados e código como **condição** para submissão de artigo do grupo.
- README do dataset revisado por um colega antes da publicação.
- Auditoria de reprodução em ambiente limpo como etapa obrigatória pré-submissão, com o log
  arquivado junto ao material do artigo.
- A cada release citada em artigo, **um único responsável** executa o ciclo de sincronização
  — artigo, README, `CITATION.cff` e vínculos — no mesmo dia.
- Datasets bem construídos tratados como produto do grupo, com autoria explícita e menção
  nos relatórios.

## Referências

- [Publicar dataset e código no Zenodo](publicar-dataset-e-codigo.md) — o passo anterior
- [help.zenodo.org](https://help.zenodo.org) — documentação oficial
- `https://api.datacite.org/dois/<DOI>` — ver o registro como o parecerista vê
