---
tags: [tipo/atomica, camada/padrao, dev/backend, sql]
criado: 2026-08-12
---

# O que o Questor não dá mora no app-db chaveado pela identidade dele

> O app lê o Questor (produção, só-leitura) e grava tudo o que o Questor não tem no seu próprio Postgres, referenciando a identidade do Questor por valor.

## O problema

O Questor é a base do sistema, mas é produção e a conexão é só-leitura — nenhuma query
do app escreve lá. E ele não tem tudo: falta o que é do app (gestor por setor, e-mail,
acompanhamento de experiência) e o cadastro dele vem sujo (setor/cargo errados), além de
gente que nem existe lá (PJ). Precisa gravar isso sem tocar no Questor.

## A solução

Dois pools separados: um só-leitura pro Questor, um gravável pro banco do app (`app-db`).
Cada tabela do app que se refere a um registro do Questor guarda a **identidade dele por
valor** — `(codigoempresa, codigofunccontr)` para contrato, `classiforgan` para setor —
**sem FK cruzando bancos** (não dá, são bancos diferentes). O merge é em TS na rota:

```
base   = query(Questor)              // read-only
overlay = appQuery(app-db)           // correções por (empresa, contrato)
efetivo = overlay ?? base            // coalesce campo a campo, no servidor
```

Três formas do overlay, todas no app-db:
- **correção**: `rh_funcionario_override(campos jsonb)` por `(empresa, contrato)` — só os campos mudados; jsonb pra evoluir sem migration.
- **entidade ausente**: `rh_pessoa_pj` com id próprio; quando precisa se passar por colaborador nos envios, ganha um "contrato" sintético `OFFSET + id` (offset alto isola do espaço real do Questor, sem colisão em coluna `int` sem FK).
- **renomear/criar setor**: `rh_setor(classiforgan pk, nome, origem)` — `classiforgan` real quando espelha o Questor, gerado (`APPnn`) quando é setor próprio; assim o resto que já chaveia por `classiforgan` (gestores, envios) não muda.

## O que mais vale lembrar

A ficha compartilhada com outro módulo continua crua; o overlay é uma **wrapper** do
módulo que edita (não muda a query base). E um lookup na fonte por chave sintética volta
vazio de propósito — quem consome tem que degradar (usar o nome guardado, resolver a
entidade local).

## Conexões
- Princípio: [[Sobre fonte read-only, o editável mora no seu banco chaveado pela identidade dela]]
- Irmã: [[Atributo efetivo é o do dono, ou o local quando não há dono]]
- Visto em: [[Navetech Hub]]
- Mapa: [[Backend]]
