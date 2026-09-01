---
tags: [tipo/atomica, camada/principio, dev]
criado: 2026-08-30
---

# A regra mora fora da porta que a chama

> Toda regra de negócio chega ao mundo por uma porta: um formulário, uma rota
> HTTP, um comando, um webhook. Se a regra for escrita DENTRO da porta, ela
> passa a só existir quando a porta existe — e a única forma de conferir se ela
> está certa é atravessar a porta, com tudo que isso arrasta junto.

## A regra

Duas camadas, sempre:

- **A porta** faz o que só existe naquela chegada: ler o formulário, conferir a
  sessão, invalidar cache, redirecionar.
- **A regra** recebe por parâmetro o que a porta lhe deu e devolve um resultado
  descrito por tipo. Não sabe se veio de um clique, de um cron ou de um script.

O corte não é estético. É o que decide se a regra pode ser exercitada.

## Como a porta se anuncia

O sinal é sempre o mesmo, e ele aparece na primeira tentativa de testar: alguma
função só funciona dentro do contexto do framework e explode fora dele.

| O que explode | O que ele é |
|---|---|
| `cookies()`, `headers()`, `auth()` | quem chegou |
| `revalidatePath()`, `redirect()` | o que responder |
| `req`, `res`, `FormData` | como chegou |

Quando o teste morre numa dessas, a resposta certa **não é simular o
framework**. É mover a regra para fora dele. Simular a porta testa a porta.

## Por que isso vira princípio e não preferência

Porque o custo é assimétrico. Regra dentro da porta funciona perfeitamente até
o dia em que se precisa dela em dois lugares — e aí o segundo lugar é um
webhook, um comando de linha, uma migração, e a saída passa a ser duplicar a
regra ou invocar a porta por dentro. Regra fora da porta nunca cobrou nada por
estar fora.

## Vale lembrar

O resultado sai por **união marcada** (`{ tipo: "recusada", motivo }`), não por
booleano: é o que deixa a porta decidir entre mostrar erro e redirecionar.
Devolver `true`/`false` empurra a decisão de volta para dentro dela, e com ela a
regra.

## Conexões
- Aplica: [[A regra que a server action executa mora fora dela]] — a forma
  concreta em Next, e os dois casos que promoveram isto a princípio.
- Irmã: [[Um invariante se garante na estrutura, não no processo]]
- Irmã: [[Semear teste cria linha nova, não muta linha real]]
- Mapa: [[Base]]
