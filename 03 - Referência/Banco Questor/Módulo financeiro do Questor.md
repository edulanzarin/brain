---
tags: [tipo/atomica, camada/referencia, dev/backend, banco/questor, sql]
criado: 2026-07-18
---

# Módulo financeiro do Questor

> O financeiro que tem dados aqui são as **duplicatas** (contas a pagar/receber) que nascem das notas fiscais, mais a **conciliação bancária**. O módulo Questor Financeiro "completo" (cadastro próprio de cliente/fornecedor em `pessoafinanceiro`) está quase vazio neste escritório — e a view `cliente` é uma armadilha.

## Duplicatas — geradas das notas fiscais

As parcelas a pagar/receber saem direto das notas (o financeiro é derivado do fiscal, via `chavelctofis*`):

- **`duplicataent`** (~55k) — contas a **pagar** (duplicatas de notas de entrada): PK `(codigoempresa, codigopessoa, chaveduplicataent)`; `chavelctofisent` → [[Modelo de dados fiscais do Questor|nota de entrada]], `codigopessoa` → fornecedor, `numeroduplicataent`, `indtitulo`.
  - `duplicataentparcela` (~64k) — as **parcelas**: `numeroparcela`, `datavencimento`, `valorparcela`, `situacao`.
- **`duplicatasai`** (~114k) — contas a **receber** (duplicatas de notas de saída): PK `(codigoempresa, codigoestab, chaveduplicatasai)`; `chavelctofissai`, `codigopessoa` → cliente, `nfsenumero`.
  - `duplicatasaiparcela` (~129k) — parcelas.

Para fluxo de caixa/aging: agrupar `*parcela` por `datavencimento` e `situacao`, ligando ao cabeçalho para empresa/contraparte.

## Conciliação bancária — `concbanclcto` (~1,2M)

Extrato bancário importado e sua conciliação. PK `(codigoempresa, codigoestab, contactb, datalctobanc, seqlctobanc)`.

- `valorlcto`, `naturlcto` (crédito/débito), `descrbanco`, `numerodcto`, `nome`.
- `concillctoctb` (S/N) + `chavelctoctb` → liga ao lançamento contábil conciliado ([[Módulo contábil do Questor]]).
- `contactb` = a conta bancária no `planoespec`.

## Questor Financeiro "completo" (pouco usado) e a armadilha `cliente`

Existe um módulo financeiro com **cadastro próprio**, separado do fiscal:

- `pessoafinanceiro` (~227) + `pessoafincliente` (~216) — clientes/pagadores do módulo financeiro, com **endereço normalizado** (`pessoaendereco` → `endereco` → `logradoro`/`bairro`/`tipologradoro`/`municipio`) e contatos (`pessoacontato`).
- A view **`cliente`** monta o cliente a partir desse mundo (`pessoafinanceiro`, não `pessoa`).

**Armadilha**: `cliente`/`pessoafinanceiro` **não** são as contrapartes das notas fiscais. Aqui elas estão quase vazias (o escritório não usa o financeiro completo). Para fornecedor/cliente das notas, use **`pessoa`** ([[Cadastros centrais do Questor - empresa, estab, pessoa]]). Confirma a regra: nome de tabela/view no Questor não é contrato — validar com os dados.

## Por que importa

Contas a pagar/receber e conciliação são a matéria-prima de relatórios de fluxo de caixa e inadimplência. E o aviso sobre `cliente` evita construir em cima do cadastro errado.

## Conexões
- Índice do banco: [[Banco Questor]] · Convenções: [[Panorama e convenções do banco Questor]]
- Deriva do fiscal: [[Modelo de dados fiscais do Questor]] · Concilia no contábil: [[Módulo contábil do Questor]]
- Cadastro certo de contraparte: [[Cadastros centrais do Questor - empresa, estab, pessoa]]
- Mapa: [[Banco Questor]]
