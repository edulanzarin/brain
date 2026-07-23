---
tags: [tipo/atomica, camada/referencia, dev/backend, banco/questor, sql]
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
- `funclocal` — lotação por vigência: PK `(codigoempresa, codigofunccontr, datatransf)`; aponta `codigoestab` + `classiforgan` (o **organograma/setor**) e dados CAGED/RAIS. A vigência aqui é `datatransf` (data da transferência), não `datainicial`; lotação atual = maior `datatransf`.
- `funcescala` (jornada), `funcctps`, `funclegais`, `funcsindicato`, `funcadicional` (insalubridade/periculosidade), etc.
- `cargo` (`descrcargo`, `cbo` → `cbo`) e `funcao` (`descrfuncao`) são cadastros globais.

## Rotatividade (turnover): admissão, desligamento e efetivo

Tudo sai de `funccontrato`, no nível de **contrato**:

- **Admissões** no período = contratos com `dataadm` dentro do intervalo.
- **Desligamentos** no período = contratos com `datadem` dentro do intervalo.
- **Efetivo numa data D** (headcount) = contratos admitidos até D e ainda não desligados: `dataadm <= D and (datadem is null or datadem >= D)`. É estoque, consequência do fluxo — mesma ideia de [[Balancete é movimento do período, saldo é consequência]].
- **Turnover** = ((admissões + desligamentos) / 2) ÷ colaboradores ativos × 100. Existem variantes do denominador — a fórmula "de livro" usa **efetivo médio** ((início+fim)/2), mas o relatório de RH que o escritório usa como referência usa **colaboradores ativos = efetivo no FIM** do intervalo (verificado batendo número a número: setor com 7 ativos, 1 adm e 1 dem → 14,29% = (1+1)/2/7). Alinhar ao denominador do DP, senão os números divergem do que ele confere.
- **Quebra por setor (organograma)**: atribui cada contrato à sua lotação vigente (`funclocal` → `classiforgan` + `codigoestab`) e junta em `organograma (codigoempresa, codigoestab, classiforgan)` pra pegar `descrorgan` (o nome do setor, ex.: TINTURARIA MALHA). Some por setor os mesmos ativos/adm/dem.

Conta-se por **contrato**, não por pessoa (uma pessoa com dois vínculos conta dois). Armadilha: **transferência** entre estabelecimentos da mesma empresa gera par desligamento+admissão que infla o índice. Recorte de vínculo: nesta base o `codigocateg` (smallint) vem **nulo** — quem tem valor é `funccontrato.categoria` (varchar, ex.: `'01'` empregado, `'07'`, `'11'`) e `tipovinculo` (varchar, ex.: `'10'` CLT, `'80'` diretor); use esses pra filtrar CLT. E `codigosit` é status cadastral (=1 pra todos), **não** serve de ativo/inativo — o vivo/desligado é `datadem`. A série mensal sai do padrão [[Estoque e fluxo numa série a partir de datas de início e fim]].

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
- Mapa: [[Banco Questor]]
