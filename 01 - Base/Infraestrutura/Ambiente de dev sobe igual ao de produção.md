---
tags: [tipo/atomica, camada/principio, infra, docker]
criado: 2026-07-20
---

# Ambiente de dev sobe igual ao de produção

> A mesma receita sobe nos dois lugares. O que muda entre eles é só variável de
> ambiente, nunca a estrutura.

## A regra

Um `docker-compose.yml` com a mesma topologia em dev e em produção: app, banco e
migrations como serviços, mesma rede, mesmos nomes derivados do slug
([[O nome do projeto governa o nome dos recursos]]). O que difere vai pro `.env` e,
quando precisa, num `docker-compose.override.yml` — nunca num arquivo paralelo escrito
à mão que ninguém lembra de atualizar.

Em particular: **banco em container também em dev**. Postgres instalado na máquina é a
fonte clássica de "na minha máquina funciona" — versão diferente, extensão diferente,
collation diferente.

## Por que

Diferença entre ambientes não aparece quando é criada; aparece no deploy, no pior
horário, com o menor contexto. Toda vez que dev e produção divergem em estrutura, o
primeiro deploy vira depuração de infra em vez de verificação da feature.

O corolário vale pro build também: verificar em `next dev` não prova nada sobre
produção — ver [[Verificar no build de produção, não só em dev]].

## O que aceita divergir

Volume montado pra hot reload, porta publicada, nível de log e dado semente. Tudo isso
é configuração. Se a divergência precisar de um serviço a mais ou a menos, ela deixou
de ser configuração e virou risco.

## Conexões
- Depende de: [[Configuração vem do ambiente, não do código]]
- Padrões que aplicam: [[Migrations em container próprio no Docker Compose]] · [[Next.js standalone no Docker e o outputFileTracingRoot]]
- Irmã: [[O nome do projeto governa o nome dos recursos]] · [[Verificar no build de produção, não só em dev]]
- Mapa: [[Base]] · [[Infra]]
