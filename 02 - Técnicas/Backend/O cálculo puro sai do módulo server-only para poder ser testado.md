---
tags: [tipo/atomica, camada/padrao, dev/backend, dev/frontend]
criado: 2026-08-14
---

# O cálculo puro sai do módulo server-only para poder ser testado

> A regra de negócio (classificar, derivar prazo, escolher slot) vive num módulo
> PURO — sem DB, sem `import "server-only"`. O módulo server-only só orquestra a
> IO em volta dela. Assim a lógica fica testável num runner de unidade e reusável
> em mais de um caminho.

## O problema

Um módulo que começa com `import "server-only"` **não importa num test runner**:
o `server-only` existe para explodir quando alguém o puxa fora do bundle de
servidor, e o Vitest/Jest roda em Node comum. O mesmo vale para qualquer módulo
que importe o cliente de banco (`pg`) ou faça fetch no topo. Resultado: a parte
mais importante de conferir — a que decide *vencida / no prazo / a vencer*, o
prazo, o slot de aviso — fica trancada num arquivo que o teste não alcança.

## A solução

Extrair só o **cálculo** para um módulo neutro (sem IO, sem `server-only`), que o
módulo de servidor passa a importar:

```
rescisoes.ts            (server-only: escopo de sessão, queries, e-mail)
  └── importa
rescisoes-calculo.ts    (puro: addDias, montarItem, ordenarItens, slotDeAviso)
```

O servidor vira uma casca fina que busca os dados e chama as funções puras; o
teste importa `rescisoes-calculo` direto e cobre cada fronteira (dia do prazo,
antecedência+1, resolvida sem contagem, slot da vencida por dia). Comportamento
idêntico — é refatoração, não mudança.

## O que mais vale lembrar

- **Testabilidade é um efeito, não o motivo.** O motivo é a mesma régua de sempre:
  puxar a regra pro nível mais primitivo em que ela é verdade
  ([[Primitivos, reaproveitamento e modularidade vêm antes da escala]]). A regra
  de situação de rescisão não depende do Questor nem do app-db — então não mora
  no módulo que fala com eles.
- **Reuso cai de brinde.** Uma vez pura, a mesma função serve a fila da tela, ao
  digest do cron e ao card do painel, sem duplicar. Foi o que já se fez antes com
  `periodosEmAberto` do controle de férias: extraída e exportada para o Painel do
  DP reusar a regra CLT num agregado do escritório.
- **O tipo neutro puxa junto.** Os tipos compartilhados (contrato de item/config)
  ficam num arquivo sem `server-only` — o mesmo motivo por que o cliente já os
  importa sem arrastar código de servidor pro bundle.
- Marcar os testes fora do build de produção: `**/*.test.ts` no `exclude` do
  tsconfig, para o `next build` não os empacotar; o runner tem config própria.

## Conexões
- Princípio: [[Um invariante se garante na estrutura, não no processo]]
- Depende de: [[O que o Questor não dá mora no app-db chaveado pela identidade dele]]
- Visto em: [[Navetech Hub]]
- Mapa: [[Backend]]
