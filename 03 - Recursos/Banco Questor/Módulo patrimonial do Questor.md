---
tags: [tipo/atomica, dev/backend, banco/questor, sql]
criado: 2026-07-18
---

# Módulo patrimonial do Questor

> `patbem` é o cadastro do bem (ativo imobilizado); `lctobem` é o lançamento fiscal do bem (crédito de ICMS do ativo — CIAP); e a depreciação/encargos alimentam a contabilidade (origem `IM`). Bens costumam nascer de uma nota de entrada (`chavelctofisent`).

## O bem — `patbem` (~36k)

Cadastro do ativo imobilizado. PK `codigobem` (bigint, global).

| Coluna | O que é |
|---|---|
| `codigoempresa` | empresa dona do bem |
| `numerobem`, `adicao` | numeração do bem |
| `dataaquisicao`, `dataincorporacao` | datas de aquisição/incorporação |
| `valor`, `quantidade` | valor e quantidade |
| `codigogrupo` → `patgrupo` | grupo/classe do bem (por empresa) |
| `chavelctofisent` | nota fiscal de aquisição → [[Modelo de dados fiscais do Questor|entrada]] |
| `tipopatrimonio`, `situacao`, `datasituacao` | tipo e situação (ativo/baixado) |
| `datafinaldeprfiscal` / `datafinaldeprsocietaria` | fim da depreciação (visão fiscal × societária) |

- `patgrupo` — grupos/classificação de bens (`codigogrupo`, `classificacao`, `descricao`), por empresa.
- `patbemcontacontabil` — contas contábeis vinculadas ao bem.

## Lançamento fiscal do bem / CIAP — `lctobem`

`lctobem` (~2k) registra o bem sob a ótica **fiscal** (crédito de ICMS do ativo imobilizado): PK `(codigoempresa, codigoestab, chavelctobem)`. Traz `descrbem`, `valorcontabil`, `valorcreditoicms`, `valoricmsproprio`/`valoricmsst`, `vidautil`, `chavelctofisent`, `contactb`, dados de venda do bem, `codigobem` → `patbem`.

- `lctobemcredito` (~26k) — as **parcelas do crédito** de ICMS (apropriação em 48 meses — CIAP).
- `lctobemdebito` — estornos/débitos.

## Depreciação e integração contábil

- `patenccontacontabil` (~1,9M) — **encargos** (depreciação) por bem/conta.
- `pattabelactblcto` / `pattabelactblctoctb` (~430k cada) — geração e vínculo dos lançamentos contábeis do patrimônio.
- `patcreditodacon` (~456k) — crédito de depreciação acelerada (DACON/incentivos).
- `patperiodocontabilizado` — controle de períodos já contabilizados.

Tudo isso desemboca em [[Módulo contábil do Questor]] com origem **`IM`** (Controle Patrimonial).

## Por que importa

Base para relatórios de imobilizado, depreciação e CIAP (crédito de ICMS do ativo). Liga fiscal (nota de aquisição) ↔ patrimônio ↔ contabilidade.

## Conexões
- Índice do banco: [[Banco Questor]] · Convenções: [[Panorama e convenções do banco Questor]]
- Nota de aquisição: [[Modelo de dados fiscais do Questor]] · Integra em: [[Módulo contábil do Questor]]
