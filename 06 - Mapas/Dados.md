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
- [[Formulário montado pelo usuário — a definição no banco dirige renderer e validação]]
  — campos tipados + `config` jsonb por tipo; uma peça só renderiza, pré-visualiza e
  valida. Princípio: [[A definição em dado dirige o comportamento, não um caso no código]].
- [[Entidade núcleo cresce por tabela satélite, não por coluna]] — mantém a tabela central
  mínima; feature nova é tabela que referencia, não coluna adicionada. Escala por composição.
- [[Campo que vira indicador é coluna, o resto do documento é jsonb]] — num documento de
  forma fixa, o que se agrega/filtra vira coluna; narrativa e listas variáveis vão em jsonb.
- [[Numeric e bigint do Postgres chegam como string no driver pg]] — o `node-pg`
  entrega `numeric`/`bigint` como string; castar pra `float8` pra receber number.
- [[Consumir recurso de uso único é UPDATE condicional, não checar antes]] —
  cupom/vaga/estoque de um: o `WHERE estado_livre` no UPDATE decide a corrida pelo
  `rowCount`, sem lock. Princípio: [[Um invariante se garante na estrutura, não no processo]].

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
