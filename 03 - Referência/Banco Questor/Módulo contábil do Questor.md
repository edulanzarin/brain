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

## Lotes e histórico

- `lotectb` — agrupador de lançamentos: PK `(codigoempresa, codigolotectb)`; `descrlotectb`, `datainicial/finallotectb`, `situacaolotectb`.
- `historicoctb` — textos de histórico padrão (`codigohistctb` → `descrhistctb`).

## Integração: de onde vêm os lançamentos (`origemlctoctb`)

`codigooriglctoctb` diz qual módulo gerou o lançamento — é o **mapa de como o ERP alimenta a contabilidade**:

`FP` Folha de Pagamento · `FI` Fiscal · `CP` Contas a Pagar · `CR` Contas a Receber · `FN` Financeiro · `IM` Controle Patrimonial · `IP` Importação · `CB` Contabilidade (manual) · `CC` Conciliação de Cartão · `CE` Empréstimos · `LA` Lalur · `AA` Ajustes Anteriores · `XX` Extemporâneo · `ZZ` Zeramento (e outros). Assim dá para separar o que é lançamento manual do que veio automático de cada módulo.

## Por que importa

Base de qualquer relatório contábil (balancete, DRE, razão, livro diário) e da futura seção Contábil do [[Navetech Hub]]. O `codigooriglctoctb` conecta este módulo a folha, fiscal, financeiro e patrimônio.

## Conexões
- Índice do banco: [[Banco Questor]] · Convenções: [[Panorama e convenções do banco Questor]]
- Plano de contas e balancete: [[Plano de contas padrão do Questor e leitura do balancete]]
- Alimentado por: [[Modelo de dados fiscais do Questor]] · [[Módulo de folha e eSocial do Questor]] · [[Módulo financeiro do Questor]] · [[Módulo patrimonial do Questor]]
- Contas por empresa: [[Cadastros centrais do Questor - empresa, estab, pessoa]]
- Mapa: [[Banco Questor]]
