---
tags: [tipo/atomica, camada/referencia, dev/backend, banco/questor, sql]
criado: 2026-07-19
---

# Vínculo nota fiscal e lançamento contábil no Questor

> A contabilização de uma nota fiscal são lançamentos em `lctoctb` com `codigooriglctoctb='FI'` e `chaveorigem = 'ME'/'MS' + a chave da nota` (com zero-fill). É o elo para conferir se a nota foi contabilizada (base da automação "Conferência Fiscal").

## O elo (verificado jun/2026)

- **Entradas**: `lctoctb.chaveorigem = 'ME' || lpad(chavelctofisent::text, 10, '0')`.
- **Saídas**: `lctoctb.chaveorigem = 'MS' || lpad(chavelctofissai::text, 10, '0')`.
- Só nos lançamentos com `codigooriglctoctb='FI'`. Uma nota gera **vários** lançamentos (partida dobrada: mercadoria + cada imposto) com o **mesmo** `chaveorigem`.
- Outros prefixos de `chaveorigem` em FI: **`IM`** = impostos apurados (ex.: `IMP01`, mensal), **`RE`** = retenções. Não são notas.
- **`datalctoctb` = `datalctofis`** (100% na amostra) → dá pra filtrar os dois pelo mesmo período.
- `numerodcto`/`transctb`/`codigolctoprog` **não** servem de elo (nulos ou irrelevantes). A nota (`lctofis*`) **não tem** coluna de vínculo com o contábil — o elo mora no `chaveorigem` do lançamento.

Conferência = LEFT JOIN da nota (por `chave`) contra o conjunto `substring(chaveorigem from 3)::bigint` dos FI `ME`/`MS` da empresa no período. Nota sem match = **não contabilizada**.

## Nem toda nota deve ser contabilizada (senão dá falso positivo)

- **NFSe, CTe, NFCOM, NF3E → 100% contabilizadas.**
- **NFe de mercadoria → parcial**: só as que afetam resultado/patrimônio. **Remessas, retornos, industrialização por encomenda, consignação, vasilhame, amostra grátis, bonificação, comodato NÃO geram lançamento** (correto, não é erro). Numa indústria isso é a maioria das NFe (ex.: empresa 1200 em jun: 951 de 1358 NFe-entrada eram remessa/retorno).
- **Canceladas** não devem ser contabilizadas.

## Como saber se um CFOP "deve contabilizar"

**Resolvido em jul/2026 — ver [[Plano de contabilização por CFOP no Questor]].** A configuração existe e é exata: as colunas `cfop.codigotabctbfis*` (na tabela `cfop`, que é **por empresa + estabelecimento**) apontam, por tributo, a tabela de contabilização. **CFOP sem nenhum slot preenchido = não contabiliza** — remessa, retorno, industrialização. Não precisa de heurística. Confirmado na 1200: o CFOP `1901002` ("Entrada para Industrialização por Encomenda") vem com zero componentes, ou seja, o próprio ERP diz que não gera lançamento.

O que eu tinha escrito antes, corrigido:

- `cfoptabctbfis` **não** é o caminho — 7 linhas no banco inteiro (não é "vazia só na 1200"). `cfop.finalidade` zerada segue valendo.
- A **regra empírica** ("CFOP é contabilizável se ≥1 nota dele foi contabilizada no período") funcionava como aproximação — 951 falsos → 37 gaps reais nas entradas da 1200 — mas foi **substituída** pela config real. Continua útil para outra coisa: calibrar *quais contas recebem lançamento nota a nota*, já que ICMS/IPI de saída são contabilizados na apuração mensal (prefixo `IM`), não na nota.

## Não confundir

- **`codigooriglctoctb='IP'`** (Importação) = movimento **bancário** (Pix/TED, extrato/conciliação), **não** nota fiscal. Em jun era a maior origem (87k). Ver [[Módulo contábil do Questor]] (mapa de origens).

## Conexões
- Índice: [[Banco Questor]] · Contábil: [[Módulo contábil do Questor]]
- Visto em: [[Questor Hub]] (abas Conferência Fiscal, Conferência de Contas e Configuração no módulo Contábil)
- Notas fiscais: [[Modelo de dados fiscais do Questor]] · Impostos apurados: [[Impostos no Questor - onde fica cada um]]
- Config das contas por CFOP: [[Plano de contabilização por CFOP no Questor]]
- Mapa: [[Banco Questor]]
