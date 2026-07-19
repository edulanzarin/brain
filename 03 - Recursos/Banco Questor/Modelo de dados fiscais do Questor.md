---
tags: [tipo/atomica, dev/backend, conceito, banco/questor]
criado: 2026-07-18
---

# Modelo de dados fiscais do Questor

> Cabeçalho de nota e itens ficam em tabelas separadas; a contraparte da nota é a `pessoa` ligada por `codigopessoa`. Entradas e saídas são espelhadas (`lctofisent*` vs `lctofissai*`).

## Chaves

- Cabeçalho entrada `lctofisent` PK `(codigoempresa, chavelctofisent)`; saída `lctofissai` PK `(codigoempresa, chavelctofissai)`.
- Toda tabela filha referencia `(codigoempresa, chavelctofis{ent,sai})`.
- `codigoempresa` é `smallint`; a chave da nota é `bigint`.

## Cabeçalho (`lctofisent` / `lctofissai`)

Uma linha por nota. Campos que mais importam:

- `datalctofis` (date) — data de lançamento, usada nos filtros/relatórios.
- `valorcontabil` — valor total da nota.
- `especienf` — NFE, CTE, NFSE, NFCE, NF… (usar `upper(btrim(...))`).
- `cdmodelo` — modelo fiscal (55 NFe, 57 CTe, 65 NFCe…).
- `codigoempresa`, `codigoestab`.
- `codigopessoa` → `pessoa` — a **contraparte** (fornecedor na entrada, cliente na saída).
- `cancelada` ('0'/'1'), `cdsituacao` — ver [[Canceladas e devoluções no Questor]].
- `numeronf`, `serienf`, `chavenfeent`/`chavenfesai` (chave de acesso 44 díg.).
- `siglaestadoorigem`/`destino` — **mal preenchidas** (saídas só ~20%); pra UF usar `pessoa.siglaestado`.
- `emitentenf` ('P' = a própria empresa emitiu; 'T' = terceiro).
- `datahoralctofis` (timestamp) — carimbo completo; `datalctofis` é só a data.
- Valores extras da nota: `vlrfrete`, `vlrseguro`, `vlrdesc` (desconto), `vlroutrdesp`, `vlrpedagio`, `modalidadefrete`, `indpagto`/`meiopagamento`, `finalidadeoperacao`.
- `lctofissai` tem **66 colunas** no total; as acima são as úteis pra BI/listagem. `cdsituacao` observado: 0 normal, 2 cancelada, 8 outra (denegada/inutilizada?).
- **Quem lançou**: `codigousuario` → tabela `usuario` (`nomeusuario`). `codigousuario = 0` = **ADMINISTRADOR** (conta do sistema; as importações automáticas caem nele). Demais códigos são pessoas.
- **Origem do dado**: `origemdado` (smallint). Observado só `3` (domina — integração/e-Doc automático) e `2` (pouco — importado); `1` seria manual (não visto). Significado dos códigos não é documentado no banco — inferido.

### Contraparte é `codigopessoa` (verificado)

Comparando o CNPJ embutido na chave da NFe (posições 7–20) com `pessoa.inscrfederal`:
em entradas com `emitentenf='T'`, 97% batem (é o fornecedor). Com `emitentenf='P'` a
entrada foi emitida pela própria empresa (devolução) e a pessoa é o cliente.

## Itens (`lctofisentproduto` / `lctofissaiproduto`)

Uma linha por produto (`seq`): `codigoproduto`, `codigocfop`, `unidademedida`,
`valorunitario`, `valortotal`, `quantidade` e os impostos de item — ver
[[Impostos no Questor - onde fica cada um]]. **Não têm `cancelada`** (juntar ao
cabeçalho pra excluir canceladas). Tabelas enormes (saídas ~47M).

**Auditoria é só no cabeçalho**: as tabelas de item têm apenas `datalctofis`, **não** têm `codigousuario` nem `datahoralctofis`. Quem lançou e quando (análise de produtividade por colaborador, ritmo horário) sai do cabeçalho: `codigousuario` → `usuario` e `datahoralctofis` (timestamp) dão hora/dia-da-semana. `codigousuario = 0` = ADMINISTRADOR/importação automática. Foi essa a base da seção Produtividade do [[Questor BI]].

## Tabelas de apoio

- `produto` — PK `(codigoempresa, codigoproduto)`; `descrproduto`, `codigoncm`, `unidademedida`. Por empresa.
- `cfop` — PK `(codigoempresa, codigoestab, codigocfop)`; `descrcfop`, `finalidade`, flags de apuração. `codigocfop` é o código interno do Questor (ex: 5102002), não o CFOP fiscal puro de 4 díg.
- `pessoa` — PK `codigopessoa`; `nomepessoa`, `inscrfederal` (CNPJ/CPF), `siglaestado`, `codigomunic`. É a contraparte **fiscal**; há outro cadastro de pessoa no financeiro — ver [[Cadastros centrais do Questor - empresa, estab, pessoa]].
- `municipio` — chave `(siglaestado, codigomunic)`; nome em `nomemunic`. Município da contraparte: `pessoa.codigomunic + pessoa.siglaestado → municipio`. O `codigomunic` é interno do Questor; o **código IBGE** está em `municipio.codigofederal`.
- `empresa` — PK `codigoempresa`; só `nomeempresa` (4 colunas). **CNPJ/endereço/regime não ficam aqui, e sim em `estab`** — ver [[Cadastros centrais do Questor - empresa, estab, pessoa]].
- `estado` — PK `siglaestado`; `nomeestado`, alíquotas de ICMS.

## Dimensões úteis do cabeçalho (verificadas)

- `cdmodelo`: **55** NFe, **65** NFCe, **57** CTe, **99** outros. (Igual a `especienf` na prática.)
- `modalidadefrete`: **0** por conta do emitente (CIF), **1** por conta do destinatário (FOB), **2** terceiros, **3** próprio remetente, **4** próprio destinatário, **9** sem frete.
- Valores de ajuste da nota: `vlrfrete`, `vlrdesc`, `vlrseguro`, `vlroutrdesp` (frete e desconto costumam vir preenchidos; seguro quase sempre 0).

## Grupos (armadilha)

`grupoprocessam`/`grupoempresa` **não** são grupos de empresas para análise — são de
processamento interno. Ver [[grupoprocessam do Questor não é grupo de empresas]].

## Por que importa

Base de qualquer relatório/BI sobre o Questor — usado no [[Questor BI]]. Consultar essas
tabelas gigantes exige [[Agregar antes de juntar em tabelas gigantes no Postgres]].

## Conexões
- Índice do banco: [[Banco Questor]] · Convenções gerais: [[Panorama e convenções do banco Questor]]
- Usado por: [[Questor BI]]
- Conexão: [[Questor - conexão read-only e regras]]
- Detalha impostos: [[Impostos no Questor - onde fica cada um]] · Reforma: [[Reforma tributária IBS-CBS no Questor]]
- Cadastros: [[Cadastros centrais do Questor - empresa, estab, pessoa]]
- Ver também: [[Canceladas e devoluções no Questor]] · [[Receitas SQL do Questor]] · [[grupoprocessam do Questor não é grupo de empresas]]
