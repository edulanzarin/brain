---
tags: [tipo/atomica, dev/backend, banco/questor, sql]
criado: 2026-07-18
---

# Módulo de folha e eSocial do Questor

> A folha separa **pessoa** (`funcpessoa`) de **vínculo** (`funccontrato`, por empresa); os dados que mudam (salário, cargo, lotação) ficam em tabelas de vigência por `datainicial`; e o resultado calculado da folha é `calculoevento` (rubrica × funcionário × período). A view `funcionario` já junta tudo.

## Núcleo: pessoa × vínculo

- **`funcpessoa`** — a **pessoa física** (96 colunas): `codigofuncpessoa` PK; `nomefunc`, `cpffunc`, `pisfunc`, endereço, nascimento, documentos. **Global**, sem `codigoempresa` — a mesma pessoa pode trabalhar em várias empresas.
- **`funccontrato`** — o **vínculo empregatício** por empresa: PK `(codigoempresa, codigofunccontr)`; `codigofuncpessoa` (FK → pessoa), `dataadm`, `datadem`, `categoria`/`codigocateg`, `tipovinculo`, `codigosit` (situação). Uma pessoa = vários contratos.
- ~19 mil pessoas / ~21 mil contratos na base.

## Vigências ("último registro por data")

Dados que mudam no tempo ficam em tabelas com `datainicial` (PK inclui a data); o **estado atual é o de maior `datainicial`**:

- `funcsalario` — salário (`valorsal`, `tiposalario`).
- `funccargo` — cargo/função (`codigocargo` → `cargo`; `codigofuncao` → `funcao`).
- `funclocal` — lotação (aponta o `codigoestab` onde trabalha) e dados CAGED/RAIS.
- `funcescala` (jornada), `funcctps`, `funclegais`, `funcsindicato`, `funcadicional` (insalubridade/periculosidade), etc.
- `cargo` (`descrcargo`, `cbo` → `cbo`) e `funcao` (`descrfuncao`) são cadastros globais.

## Cálculo da folha

- **`periodocalculo`** — o período/competência: PK `(codigoempresa, codigopercalculo)`; `compet` (competência), `datainicialfolha`/`datafinalfolha`, `datapgto`, `codigotipocalc` → `tipocalculo`, `fechado`.
- **`tipocalculo`** (tipo de folha): `1` Mensal, `8` Adiantamento salarial, `20-23` 13º salário, `40` Intermitente, `50/52` Férias (normais/coletivas), `60` Rescisão, `70/71` Provisão (13º/férias), `10/12` Distribuição/PLR, `6` Dissídio, `80` Transferência.
- **`funcpercalculo`** (~273k) — liga um contrato a um período calculado.
- **`calculoevento`** (~5,1M) — **o resultado da folha**: PK `(codigoempresa, codigopercalculo, codigofunccontr, codigoevento)`; `referevento` (referência: horas/dias/%), `valorevento` (R$), `baseevento`. É "quanto cada rubrica rendeu para cada funcionário em cada período".

## Rubricas — `evento`

Cadastro das rubricas/verbas (`codigoevento` → `descrevento`), com dezenas de flags de incidência (`inssmensal`, `fgtsmensal`, `irrfmensal`…). `tipoevento` classifica (observado): `1` provento/vencimento, `2` reembolso/salário-família, `3` desconto, `4` base de cálculo/informativo, `5` afastamento, `6` outros (banco de horas, abono). Confirmar por amostragem ao usar — os limites entre 3/4/6 são fluidos.

## View `funcionario` (atalho oficial)

A view `funcionario` junta `funccontrato` + `funcpessoa` + **o registro mais recente de cada tabela de vigência** (salário, cargo, local → `estab`, escala…) usando `max(datainicial)`. É a forma pronta de obter a "ficha atual" de cada funcionário sem reescrever essa lógica de vigência. Ótima referência de como as `func*` se ligam.

## eSocial (staging técnico — não é a folha)

`esocial*` (~429 tabelas) + `xml*` (~228) guardam **eventos, lotes e XML do eSocial** e seus retornos (`esocialret*`, `esocialdadoss*`, `esocialxml` ~10 GB de XML, `esocialeventoretornorubricas`…). É infraestrutura de transmissão ao governo. Para a folha "de negócio" (o que cada um ganhou), use `calculoevento` + `func*`, **não** as `esocial*`.

## Por que importa

Base para relatórios de folha/RH (custo de pessoal, headcount, admissões/demissões, provisões) e integração com a contabilidade (origem `FP` em [[Módulo contábil do Questor]]). Modelo pessoa-vínculo-vigência é a chave para não errar nas consultas.

## Conexões
- Índice do banco: [[Banco Questor]] · Convenções: [[Panorama e convenções do banco Questor]]
- Integra em: [[Módulo contábil do Questor]] (origem FP)
- Padrão de vigência também citado em: [[Cadastros centrais do Questor - empresa, estab, pessoa]]
