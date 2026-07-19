---
tags: [tipo/moc]
criado: 2026-07-18
---

# Banco Questor

Mapa do banco do sistema contábil **Questor** (banco PostgreSQL `Navecon` do escritório). É referência de schema para **qualquer** sistema/automação que leia o Questor — não é de um projeto só. Antes de construir algo que toque o banco, começar por aqui.

Notas em `03 - Recursos/Banco Questor`. Banco é **produção**: acesso somente leitura ([[Questor - conexão read-only e regras]]).

## Comece por aqui

- [[Panorama e convenções do banco Questor]] — escala (~2.865 tabelas, 213 GB), mapa dos módulos, convenções que valem para o banco todo.
- [[Questor - conexão read-only e regras]] — como conectar, read-only obrigatório, performance.

## Módulos

- **Fiscal** — [[Modelo de dados fiscais do Questor]] · [[Impostos no Questor - onde fica cada um]] · [[Canceladas e devoluções no Questor]] · [[Reforma tributária IBS-CBS no Questor]]
- **Contábil** — [[Módulo contábil do Questor]] · [[Vínculo nota fiscal e lançamento contábil no Questor]] · [[Plano de contabilização por CFOP no Questor]]
- **Folha / eSocial** — [[Módulo de folha e eSocial do Questor]]
- **Financeiro** — [[Módulo financeiro do Questor]]
- **Patrimonial** — [[Módulo patrimonial do Questor]]
- **Cadastros centrais** — [[Cadastros centrais do Questor - empresa, estab, pessoa]]
- **Logs / auditoria** — [[Logs e auditoria no Questor]]

## Consultas e armadilhas

- [[Receitas SQL do Questor]] — consultas prontas e testadas.
- [[Agregar antes de juntar em tabelas gigantes no Postgres]] — padrão obrigatório nas tabelas de milhões de linhas.
- [[grupoprocessam do Questor não é grupo de empresas]] — armadilha de nomenclatura.

## Quem usa

- [[Questor BI]] — primeiro sistema construído sobre este banco (módulo Fiscal).

---

Voltar para [[Início]] · Mapa técnico: [[Desenvolvimento]]
