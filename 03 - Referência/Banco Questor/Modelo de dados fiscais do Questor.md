---
tags: [tipo/atomica, camada/referencia, dev/backend, banco/questor]
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

- `datalctofis` (date) — a data do **DOCUMENTO** (a competência), apesar do nome. Não é quando alguém digitou — ver o aviso abaixo.
- `valorcontabil` — valor total da nota.
- `especienf` — NFE, CTE, NFSE, NFCE, NF… (usar `upper(btrim(...))`).
- `cdmodelo` — modelo fiscal (55 NFe, 57 CTe, 65 NFCe…).
- `codigoempresa`, `codigoestab`.
- `codigopessoa` → `pessoa` — a **contraparte** (fornecedor na entrada, cliente na saída).
- `cancelada` ('0'/'1'), `cdsituacao` — ver [[Canceladas e devoluções no Questor]].
- `numeronf`, `serienf`, `chavenfeent`/`chavenfesai` (chave de acesso 44 díg.).
- `siglaestadoorigem`/`destino` — **mal preenchidas** (saídas só ~20%); pra UF usar `pessoa.siglaestado`.
- `emitentenf` ('P' = a própria empresa emitiu; 'T' = terceiro).
- `datahoralctofis` (timestamp) — o carimbo de **quando a linha foi gravada**. NÃO é `datalctofis` com hora.
- Valores extras da nota: `vlrfrete`, `vlrseguro`, `vlrdesc` (desconto), `vlroutrdesp`, `vlrpedagio`, `modalidadefrete`, `indpagto`/`meiopagamento`, `finalidadeoperacao`.
- `lctofissai` tem **66 colunas** no total; as acima são as úteis pra BI/listagem. `cdsituacao` observado: 0 normal, 2 cancelada, 8 outra (denegada/inutilizada?).
- **Quem lançou**: `codigousuario` → tabela `usuario` (`nomeusuario`). `codigousuario = 0` = **ADMINISTRADOR** (conta do sistema; as importações automáticas caem nele). Demais códigos são pessoas.
- **Origem do dado**: `origemdado` (smallint). Observado só `3` (domina — integração/e-Doc automático) e `2` (pouco — importado); `1` seria manual (não visto). Significado dos códigos não é documentado no banco — inferido.

### `datalctofis` e `datahoralctofis` são datas DIFERENTES (verificado ago/2026)

O nome engana e esta nota já afirmou o contrário: `datahoralctofis` **não** é
`datalctofis` com hora. São o par fato × registro — a data do documento e o carimbo de
quando o fiscal escriturou.

Medido em `lctofissai`, documentos de jul/2026 (591.542 notas): **zero** tinham
`datahoralctofis::date = datalctofis`. Todas foram gravadas DEPOIS, com defasagem
mediana de 25 dias e massa entre 18 e 33 — o escritório fecha o mês anterior durante o
mês seguinte. O mesmo vale para `origemdado` 2 e 3, então não é artefato de importação.

Consequência prática, e é grande: **relatório de produtividade filtrado por
`datalctofis` responde outra pergunta**. "Quanto o time lançou em julho" filtrado assim
devolve as notas *de* julho, digitadas em agosto — ver [[Produtividade se mede pela hora
do registro, não pela data do fato]]. Para medir gente, o recorte é `datahoralctofis`;
para medir o negócio (faturamento, apuração), é `datalctofis`.

O preço: só `datalctofis` tem índice (`(codigoempresa, codigoestab, datalctofis)`).
Filtrar pelo carimbo é varredura sequencial — ~4 s para o escritório inteiro num mês nas
duas tabelas, medido. Aceitável atrás de um botão, caro num autoload.

A diferença entre as duas colunas também é, ela mesma, um indicador: é o **atraso de
escrituração**, e responde em que competência o time está trabalhando.

### Contraparte é `codigopessoa` (verificado)

Comparando o CNPJ embutido na chave da NFe (posições 7–20) com `pessoa.inscrfederal`:
em entradas com `emitentenf='T'`, 97% batem (é o fornecedor). Com `emitentenf='P'` a
entrada foi emitida pela própria empresa (devolução) e a pessoa é o cliente.

## Itens (`lctofisentproduto` / `lctofissaiproduto`)

Uma linha por produto (`seq`): `codigoproduto`, `codigocfop`, `unidademedida`,
`valorunitario`, `valortotal`, `quantidade` e os impostos de item — ver
[[Impostos no Questor - onde fica cada um]]. **Não têm `cancelada`** (juntar ao
cabeçalho pra excluir canceladas). Tabelas enormes (saídas ~47M).

**Auditoria é só no cabeçalho**: as tabelas de item têm apenas `datalctofis` (a data do documento), **não** têm `codigousuario` nem `datahoralctofis`. Por isso qualquer corte de imposto POR PESSOA tem de partir do cabeçalho e juntar a filha pela chave da nota — nunca filtrar a filha pela própria data, que é a do documento. Quem lançou e quando (análise de produtividade por colaborador, ritmo horário) sai do cabeçalho: `codigousuario` → `usuario` e `datahoralctofis` (timestamp) dão hora/dia-da-semana. `codigousuario = 0` = ADMINISTRADOR/importação automática. Foi essa a base da seção Produtividade do [[Navetech Hub]].

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

## `lctofis*` pode ter várias linhas por nota (armadilha de "duplicata")

Uma mesma nota (mesma `chavenfe*` de 44 díg.) pode virar **mais de uma linha** de
cabeçalho `lctofisent`/`lctofissai` — repartida por CFOP, cada linha com o seu
`valorcontabil` (a soma = valor da nota). Ex. verificado (PAPINHA/488, jun/2026):
NF-e 451949 = 2 linhas, CFOP 1902002 (R$ 12.781,62) + CFOP 1124002 (R$ 1.064,88),
`chavelctofissai` distintos. Consequências:

- **Detectar "nota duplicada no fiscal" por chave/documento é falso positivo em
  massa** — toda nota multi-CFOP parece "lançada 2×". Validado: por chave dava 1890
  grupos numa empresa; o critério rigoroso (linha idêntica = mesma `chavenfe` + CFOP +
  `valorcontabil` repetidos em 2+ `chavelctofis*` distintos) deu **0** em todas as
  empresas/mês testados. Duplicidade fiscal limpa é rara/inexistente nesses dados; a
  que importa é a **contábil** (re-rodar a contabilização), em [[Vínculo nota fiscal e lançamento contábil no Questor]].
- Ao somar por nota, agregue as linhas da `chavenfe*` (ou do documento), não trate
  cada linha como uma nota.

## Grupos (armadilha)

`grupoprocessam`/`grupoempresa` **não** são grupos de empresas para análise — são de
processamento interno. Ver [[grupoprocessam do Questor não é grupo de empresas]].

## Por que importa

Base de qualquer relatório/BI sobre o Questor — usado no [[Navetech Hub]]. Consultar essas
tabelas gigantes exige [[Agregar antes de juntar em tabelas gigantes no Postgres]].

## Conexões
- Ver também: [[Canceladas e devoluções no Questor]] · [[Receitas SQL do Questor]] · [[grupoprocessam do Questor não é grupo de empresas]]
- Visto em: [[Navetech Hub]]
- Índice do banco: [[Banco Questor]] · Convenções gerais: [[Panorama e convenções do banco Questor]]
- Conexão: [[Questor - conexão read-only e regras]]
- Detalha impostos: [[Impostos no Questor - onde fica cada um]] · Reforma: [[Reforma tributária IBS-CBS no Questor]]
- Cadastros: [[Cadastros centrais do Questor - empresa, estab, pessoa]]
- Mapa: [[Banco Questor]]
