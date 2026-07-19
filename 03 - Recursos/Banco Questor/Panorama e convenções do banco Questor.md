---
tags: [tipo/atomica, dev/backend, banco/questor]
criado: 2026-07-18
---

# Panorama e convenções do banco Questor

> O Questor é um ERP contábil inteiro num só banco PostgreSQL: ~2.865 tabelas, 213 GB, multi-empresa. A maioria segue as mesmas convenções (chave por `codigoempresa`, colunas de auditoria, dupla escrituração), então aprender as convenções vale por centenas de tabelas.

## O banco

- **Engine** PostgreSQL 14.22 · **banco** `Navecon` · **~213 GB** · conexão sempre read-only ([[Questor - conexão read-only e regras]]).
- **~2.865 tabelas base** + 23 views no schema `public`. Só ~1.410 têm dados; ~1.455 estão vazias (features do ERP que este escritório não usa). Não se assuste com o número: o schema é o do produto Questor inteiro, não o que está em uso.
- É o banco de um **escritório de contabilidade** (NAVECON). `empresa` 1 = o próprio escritório; as demais (~1.465 empresas, ~1.794 estabelecimentos) são os **clientes**. Ver [[Cadastros centrais do Questor - empresa, estab, pessoa]].
- Há também um schema `fiscal_sql` (tabelas `users`, `execution_logs`) que **não é do Questor** — é de alguma app externa; ignorar ao mapear o ERP.

## Mapa dos módulos (por prefixo de tabela)

| Módulo | Prefixos principais | Nota |
|---|---|---|
| **Fiscal** (notas de entrada/saída, impostos) | `lctofisent*`, `lctofissai*`, `cfop`, `ncm`, `cst*` | [[Modelo de dados fiscais do Questor]] |
| **Contábil** (razão, saldos, plano de contas) | `lctoctb`, `saldoctb*`, `plano*`, `lotectb`, `dre` | [[Módulo contábil do Questor]] |
| **Folha / eSocial** (RH, cálculo, eventos) | `func*`, `calculo*`, `evento`, `periodocalculo`, `esocial*`, `xml*` | [[Módulo de folha e eSocial do Questor]] |
| **Financeiro** (duplicatas, conciliação) | `duplicata*`, `concbanc*`, `pessoafin*` | [[Módulo financeiro do Questor]] |
| **Patrimonial** (bens, depreciação, CIAP) | `pat*`, `lctobem*` | [[Módulo patrimonial do Questor]] |
| **Reforma tributária** (IBS/CBS) | `ncmrelacibscbs`, `cstibscbs`, `*produtoiva` | [[Reforma tributária IBS-CBS no Questor]] |
| **SPED / apurações** | `sped*`, `efd*`, `total*`, `defis*`, `dirf*` | (dentro das notas fiscal/contábil) |
| **Logs / auditoria** | `log*` | [[Logs e auditoria no Questor]] |
| **Config / domínio** | `config*`, `cfg*`, `tab*` + centenas de cadastros de código | (decodificam os `codigo*`) |

## Convenções que valem para o banco todo

- **Multi-empresa por `codigoempresa`** (`smallint`). Quase toda tabela transacional começa a PK por ele. **Sempre filtrar por empresa** — reduz drasticamente qualquer consulta.
- **PKs compostas**: cabeçalho + filhas repetem a chave do pai (ex.: `(codigoempresa, chavelctofissai)` some para todos os itens). As chaves de lançamento (`chave*`) são `bigint`.
- **Colunas de auditoria recorrentes** (aparecem em quase tudo):
  - `codigousuario` → `usuario` (quem lançou). `0` = **ADMINISTRADOR** (conta do sistema; importações automáticas caem nele).
  - `datahoralcto` / `datahoralctofis` / `datahoralctoctb` (`timestamp`) — carimbo completo; a coluna `data*` correspondente é só a data.
  - `origemdado` (`smallint`) — como o dado entrou: `3` integração/e-Doc (domina), `2` importado, `1` manual.
  - `idsyn` / `statussinczen` — sincronização com a nuvem (Questor Syn).
- **Dupla escrituração** (`tipolancamento`, em `lctoctb`/`saldoctb`): `LN` Normal, `LF` Fiscal, `LS` Societário. O ERP mantém visões fiscal e societária lado a lado (base de LALUR/ECF).
- **Histórico por vigência ("último registro por data")**: dados que mudam ao longo do tempo (salário, cargo, lotação, contrato) ficam em tabelas com `datainicial`; o **estado atual é o registro de maior data**. A view `funcionario` implementa exatamente esse padrão — ver [[Módulo de folha e eSocial do Questor]].
- **Dois mundos de cadastro de pessoa**: o fiscal usa `pessoa` (7,2M linhas); o financeiro usa `pessoafinanceiro` (~227, quase sem uso aqui). Não confundir — ver [[Cadastros centrais do Questor - empresa, estab, pessoa]].
- **Códigos decodificados por tabelas de domínio**: quase todo `codigoX`/`tipoX smallint` tem uma tabela (`tipocalculo`, `origemlctoctb`, `historicoctb`, `cfop`…) que traduz. Ao ver um código estranho, procure a tabela irmã.
- **Views de conveniência**: 23 views em `public` (`funcionario`, `cliente`, `agenciabanc`, `vwgrupoempresa`…) encapsulam os joins "oficiais" do Questor — boas para descobrir como as tabelas se ligam.

## Por que importa

É a base de qualquer sistema/automação sobre o Questor ([[Questor BI]] e futuros). Entender estas convenções evita redescobrir o schema tabela a tabela. Para tabelas gigantes, ver [[Agregar antes de juntar em tabelas gigantes no Postgres]].

## Conexões
- Índice do banco: [[Banco Questor]]
- Regras de acesso: [[Questor - conexão read-only e regras]]
- Módulos: [[Modelo de dados fiscais do Questor]] · [[Módulo contábil do Questor]] · [[Módulo de folha e eSocial do Questor]] · [[Módulo financeiro do Questor]] · [[Módulo patrimonial do Questor]] · [[Logs e auditoria no Questor]]
