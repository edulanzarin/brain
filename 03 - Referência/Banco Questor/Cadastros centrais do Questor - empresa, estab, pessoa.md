---
tags: [tipo/atomica, camada/referencia, dev/backend, banco/questor]
criado: 2026-07-18
---

# Cadastros centrais do Questor - empresa, estab, pessoa

> Contraintuitivo: `empresa` quase não tem dados — a identidade fiscal (CNPJ, endereço, natureza jurídica) mora em `estab` (o estabelecimento). E "pessoa" existe em dois mundos: `pessoa` (fiscal, 7,2M) e `pessoafinanceiro` (financeiro, quase vazio). Errar isso quebra qualquer join.

## `empresa` é só o guarda-chuva (4 colunas!)

`empresa` tem **apenas** `codigoempresa` (smallint, PK), `nomeempresa` e dois campos de logo. **Não tem CNPJ, endereço nem regime.** Serve só para identificar/nomear a empresa e agrupar os estabelecimentos. ~1.465 empresas (empresa `1` = o próprio escritório NAVECON; as demais são os clientes).

## A identidade fiscal está em `estab` (70 colunas)

`estab` é o **estabelecimento** (matriz e filiais). PK `(codigoempresa, codigoestab)` — em geral `codigoestab = 1` é a matriz. Aqui mora o que se espera da "empresa":

- `inscrfederal` — **CNPJ** (formatado, ex.: `16.749.937/0001-02`); `tipoinscr` (CNPJ/CPF); `cpfrespcno`.
- `nomeestab` / `nomeestabcompleto` (razão social) / `nomefantasia` / `apelidoestab`.
- Endereço: `enderecoestab`, `numenderestab`, `bairroenderestab`, `cependerestab`, `siglaestado` + `codigomunic` → `municipio`.
- `codigonaturjurid` → `naturjuridica`; `inscrestad`, `inscrmunic`; `codigoativfederal` (CNAE); `porteempresa`; `capitalsocial`.
- `datainicioativ` / `dataencerativ` — abertura/encerramento; `certificado` (certificado digital).

> Regra prática: para "dados cadastrais da empresa X", junte `empresa` (nome) **com** `estab` (CNPJ, endereço, regime). CNPJ = `estab.inscrfederal`.

## `pessoa` — a contraparte fiscal (7,2M)

Fornecedores/clientes das notas fiscais. PK `codigopessoa`. `nomepessoa`, `inscrfederal` (CNPJ/CPF), `siglaestado`, `codigomunic`. Detalhada em [[Modelo de dados fiscais do Questor]]. **É o cadastro que importa** para análise fiscal.

## Dois mundos de pessoa (armadilha)

| Mundo | Tabela | Uso | Volume aqui |
|---|---|---|---|
| **Fiscal** | `pessoa` | contraparte das notas (lctofis*) | 7,2M — cheio |
| **Financeiro** | `pessoafinanceiro` / view `cliente` | cliente/fornecedor do módulo Questor Financeiro, com endereço normalizado | ~227 — quase vazio |

Não são a mesma coisa nem se ligam por chave óbvia. Para contraparte de nota use `pessoa`; a view `cliente` é do financeiro (ver [[Módulo financeiro do Questor]]).

## `municipio` — três sistemas de código

Uma tabela, chaves diferentes conforme o mundo:

- `(siglaestado, codigomunic)` — chave **fiscal** (usada por `pessoa`, `estab`). `codigomunic` é interno do Questor.
- `codigomunicipio` (integer) — chave do mundo **financeiro/cadastro** (usada pela view `cliente`).
- `codigofederal` — o **código IBGE** do município (é este que se usa para cruzar com bases externas), `codigorais` para RAIS. Nome em `nomemunic`.
- `estado` — PK `siglaestado`; `nomeestado`, alíquotas de ICMS.

## Outros cadastros centrais

- `socio` (~2,7k) — quadro societário das empresas.
- `produto` — por empresa, PK `(codigoempresa, codigoproduto)` (ver [[Modelo de dados fiscais do Questor]]).
- `cargo`, `funcao`, `cbo`, `banco`, `agencia` — cadastros globais usados pela folha.

## Por que importa

Todo join começa por empresa/estab/pessoa. Saber que a identidade fiscal está em `estab` (não em `empresa`) e que há dois cadastros de pessoa evita relatórios com CNPJ vazio ou contrapartes erradas.

## Conexões
- Índice do banco: [[Banco Questor]] · Convenções: [[Panorama e convenções do banco Questor]]
- Contraparte fiscal detalhada: [[Modelo de dados fiscais do Questor]]
- Cadastro financeiro: [[Módulo financeiro do Questor]]
- Mapa: [[Banco Questor]]
