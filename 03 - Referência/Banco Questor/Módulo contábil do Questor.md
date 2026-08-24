---
tags: [tipo/atomica, camada/referencia, dev/backend, banco/questor, sql]
criado: 2026-07-18
---

# Módulo contábil do Questor

> A contabilidade é partida dobrada em `lctoctb` (débito/crédito → contas do `planoespec`), com saldos pré-agregados em `saldoctb` e o de-para para a RFB em `planoreferencial`. Todos os outros módulos (fiscal, folha, financeiro, patrimônio) desembocam aqui via `codigooriglctoctb`.

## Lançamentos — `lctoctb` (~32,5M linhas)

Partida dobrada, uma linha por lançamento. PK `(codigoempresa, chavelctoctb)`.

| Coluna | O que é |
|---|---|
| `datalctoctb` (date) | data do lançamento (índice de período) |
| `contactbdeb` / `contactbcred` (bigint) | conta debitada / creditada → `planoespec.contactb` |
| `valorlctoctb` (numeric) | valor do lançamento |
| `codigohistctb` (smallint) | histórico padrão → `historicoctb`; `complhist` = complemento texto livre |
| `codigolotectb` | lote → `lotectb` |
| `codigooriglctoctb` (varchar 2) | **origem/módulo que gerou** → `origemlctoctb` (mapa de integração, abaixo) |
| `tipolancamento` (char 2) | `LN` normal, `LF` fiscal, `LS` societário (dupla escrituração) |
| `codigoestab`, `codigousuario`, `datahoralctoctb`, `origemdado` | auditoria padrão |

- `lctoctbexcluido` (~10,5M) guarda os lançamentos **excluídos** — rastro de auditoria.
- Filtrar **sempre** por `codigoempresa` + `datalctoctb`; é tabela enorme → [[Agregar antes de juntar em tabelas gigantes no Postgres]].

### Duas datas, dois relatórios (verificado ago/2026)

`datalctoctb` é a data do FATO (competência) e `datahoralctoctb` é o carimbo de quando a linha foi gravada. As duas divergem muito: no escritório, lançamentos de maio seguiam sendo gravados em agosto. Relatório de movimento usa a primeira; produtividade de equipe usa a segunda — ver [[Produtividade se mede pela hora do registro, não pela data do fato]].

Há índice dedicado no carimbo (`ixlctoctbdatahora`), então filtrar por período de trabalho não custa mais caro que por competência. Custo medido de uma varredura agregada do escritório inteiro, sem filtro de empresa: mês ~1,5s (≈2M lançamentos), trimestre ~10s, ano ~30s.

- `codigolotectb` **vem sempre zerado** nesta base — não serve como agrupador de "rodada de trabalho". Para reconstruir a rodada, use a combinação `empresa × dia × origem` por usuário.
- `tipolancamento` é `LN` em praticamente tudo; `LF`/`LS` aparecem em punhados de linhas.
- `origemdado` (1 manual, 2 importado, 3 integração) **não** é confiável para separar trabalho a dedo de rotina: a integração fiscal grava boa parte das linhas como `1`. Quem separa é `codigooriglctoctb`.

## Plano de contas — `planoespec` (por empresa)

Plano de contas **específico de cada empresa**. PK `(codigoempresa, contactb)`.

- `contactb` (bigint) = conta "reduzida" (a que aparece em `lctoctb`).
- `classifconta` = classificação hierárquica (ex.: `1.1.01.0001`) — usar para montar a árvore/balanço.
- `descrconta`, `apelidoconta`, `tipoconta` (sintética/analítica), `natursaldo` (devedora/credora).
- `codigodre`/`contadre` (e dra/dva/dfc/dlpa…) → `dre`: mapeiam a conta nos demonstrativos.

## Plano referencial (RFB / ECD) — `planoreferencial`

De-para da conta da empresa para o **plano de contas referencial** da Receita (SPED Contábil/ECD).

- `planoreferencial` (codigoplanorefer, descrplanorefer) + `planorefercontas` (as contas referenciais).
- `planoespecplanorefer` (~8,1M) = o mapeamento `conta da empresa ↔ conta referencial`, por empresa. Grande porque cruza cada conta de cada empresa.

## Saldos — `saldoctb` / `saldoctbmensal`

Saldos **já acumulados** por conta (evita somar 32M de lançamentos).

- `saldoctb` (~10,5M) — PK `(codigoempresa, codigoestab, contactb, datasaldo, tipolancamento)`; `valordeb`, `valorcred`. `datasaldo` marca o fechamento do período.
- `saldoctbmensal` (~5,4M) — visão mensal; `saldoctbmensalhist` = histórico.
- Separados por `tipolancamento` (LN/LF/LS) → dá para ler saldo societário e fiscal.
- Para balancete/balanço, preferir `saldoctb*` a varrer `lctoctb`.

## Implantação de saldos — `implsaldoctb` (função nativa)

O Questor tem uma **função própria de Implantação de Saldos**, separada dos lançamentos. Em vez de partida dobrada em `lctoctb`, ela grava o saldo de abertura **direto por conta** em `implsaldoctb` (~12k linhas, em uso):

| Coluna | O que é |
|---|---|
| `contactb` | conta → `planoespec.contactb` |
| `datasaldo` (date) | data do saldo de abertura |
| `naturezasaldo` (smallint) | **`1` = devedor, `-1` = credor** |
| `valorsaldo` (numeric) | o saldo (positivo; a natureza dá o lado) |
| `tipolancamento` (char) | `LN` | `codigomoeda` `0` | `origemdado` `2` |
| `codigoempresa`, `codigoestab`, `codigousuario`, `datahoraimplsaldoctb` | chave + auditoria |

- **Não tem contrapartida, histórico nem complemento** — uma linha = uma conta com saldo e natureza. É o oposto de importar lançamentos contra uma conta transitória "Saldos a Implantar".
- Origens dedicadas em `origemlctoctb`: `IS` "Alteração Saldo Inicial Societário", `IF` "Alteração Saldo Inicial Fiscal" (fora `IP` Importação genérica).
- Auxiliares: `implsaldoger` (por centro de custo, vazia aqui) e `implsaldoctbrecsal` (recálculo).
- A tela que popula `implsaldoctb` é **"Cadastro: Saldos Contábeis"** (Contabilidade Questor): campos Filial, Moeda (Real=0), Tipo Lançamento (Normal=LN), Conta Contábil, Data Saldo, Natureza do Saldo (Débito/Crédito), Valor do Saldo — 1:1 com a tabela.
- **Mas essa tela NÃO tem importador** (verificado jul/2026): é digitação manual, uma conta por vez. Inviável para um balancete inteiro.
- **Consequência prática:** para implantar saldo em massa, o único caminho automatizável é importar **lançamentos** (a tela de Lançamentos tem layout de importação), gerando partida dobrada em `lctoctb` contra uma conta transitória "Saldos a Implantar" — que exige histórico e complemento. O caminho "saldo direto" existe no modelo de dados mas não é importável. Visto em: [[Navetech Hub]] (seção Contábil, implantação de saldos).

## Lotes e histórico

- `lotectb` — agrupador de lançamentos: PK `(codigoempresa, codigolotectb)`; `descrlotectb`, `datainicial/finallotectb`, `situacaolotectb`.
- `historicoctb` — textos de histórico padrão (`codigohistctb` → `descrhistctb`).

## Integração: de onde vêm os lançamentos (`origemlctoctb`)

`codigooriglctoctb` diz qual módulo gerou o lançamento — é o **mapa de como o ERP alimenta a contabilidade**:

`FP` Folha de Pagamento · `FI` Fiscal · `CP` Contas a Pagar · `CR` Contas a Receber · `FN` Financeiro · `IM` Controle Patrimonial · `IP` Importação · `CB` Contabilidade (manual) · `CC` Conciliação de Cartão · `CE` Empréstimos · `LA` Lalur · `AA` Ajustes Anteriores · `XX` Extemporâneo · `ZZ` Zeramento (e outros). Assim dá para separar o que é lançamento manual do que veio automático de cada módulo.

**O que a Navecon realmente usa** (mai–ago/2026, escritório inteiro): só seis origens aparecem — `FI` Fiscal e `IP` Importação respondem por ~99% do volume; `CB` (digitado na contabilidade), `FP` (folha), `ZZ` (zeramento) e `IM` (patrimônio) somam o resto. O digitado a dedo é pequeno em quantidade (~1%) e grande em valor — é onde mora o ajuste.

Para leitura de gestão, o código vira **natureza** do lançamento: digitado (`CB`), importado (`IP`, `CC`), integrado de outro módulo (`FI`, `FP`, `CP`, `CR`, `FN`, `IM`, `CT`, `CE`, `AD`) e apuração/ajuste (`ZZ`, `XX`, `AA`, `LA`, `EF`, `AI`, `IS`, `IF`, `TF`, `TR`, `TS`). Sem esse eixo, um mês de integração fiscal esconde as poucas milhares de linhas feitas à mão.

## Por que importa

Base de qualquer relatório contábil (balancete, DRE, razão, livro diário) e da futura seção Contábil do [[Navetech Hub]]. O `codigooriglctoctb` conecta este módulo a folha, fiscal, financeiro e patrimônio.

## Conexões
- Índice do banco: [[Banco Questor]] · Convenções: [[Panorama e convenções do banco Questor]]
- Plano de contas e balancete: [[Plano de contas padrão do Questor e leitura do balancete]]
- Alimentado por: [[Modelo de dados fiscais do Questor]] · [[Módulo de folha e eSocial do Questor]] · [[Módulo financeiro do Questor]] · [[Módulo patrimonial do Questor]]
- Contas por empresa: [[Cadastros centrais do Questor - empresa, estab, pessoa]]
- Mapa: [[Banco Questor]]
