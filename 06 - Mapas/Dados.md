---
tags: [tipo/moc]
criado: 2026-07-20
---

# Dados

Modelagem, query e performance em banco. Aqui ficam as técnicas **gerais**; o schema
de sistema externo é referência e tem mapa próprio.

## Postgres

`02 - Técnicas/Banco de dados`

- [[Agregar antes de juntar em tabelas gigantes no Postgres]] — reduzir antes de
  juntar; o padrão que salvou consulta em tabela de 47M linhas.
- [[Estoque e fluxo numa série a partir de datas de início e fim]] — de datas de
  início/fim saem fluxo (entrou/saiu) e estoque (ativos numa data); série densa
  com `generate_series` + `count filter`.

## Referência de schema externo

- [[Banco Questor]] — o banco do ERP contábil, com todos os módulos. É **referência**,
  não técnica: descreve um sistema de terceiro que eu não controlo.

## Migrations

Migration é infra, não banco: [[Migrations em container próprio no Docker Compose]],
mapa [[Infra]].

## Princípios que mandam aqui

- [[Plataforma de IA hospedada prende o app pelo banco]] — o banco é o que realmente
  prende um app a um fornecedor.

---

Voltar para [[Início]]
