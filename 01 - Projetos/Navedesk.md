---
tags: [tipo/projeto, projeto/navedesk]
criado: 2026-07-19
status: ativo
codigo_em: ~/Dev/navedesk
---

# Navedesk

> Central interna de chamados (helpdesk) da Navecon. Solicitantes abrem tickets,
> técnicos atendem com SLA por prioridade, admins configuram usuários, políticas,
> categorias e tempos. Em produção, com dado real.

Código em: `~/Dev/navedesk`

## Estado atual

Sistema completo e rodando: RBAC (solicitante/técnico/admin), ciclo de vida do
ticket (aberto → andamento → aguardando → resolvido → fechado), mensagens
públicas e notas internas, anexos, eventos de auditoria, SLA com escalonamento,
busca full-text em Postgres, notificações com SSE, filtros salvos, templates de
ticket, menções, base de conhecimento, dashboard analítico e export CSV.

**Stack:** Next.js 15 · TypeScript · PostgreSQL 16 · Drizzle · Auth.js v5 ·
Tailwind v4. ~31k linhas, 178 arquivos TS. Testes em três níveis: unitário,
property-based (fast-check) e integração com testcontainers, mais e2e em
Playwright. 343 testes passando.

Em andamento na branch `rewrite/v2`: **reescrita da camada visual**.

## A decisão que define o projeto agora

O Eduardo pediu para "refazer o sistema inteiro, mais completo e robusto". Ao
abrir o código, o diagnóstico foi outro — e vale registrar porque é o tipo de
coisa que se esquece:

**O backend não era o problema.** Repositories, services, políticas isoladas,
engine de SLA, FTS, testes de verdade. Um rewrite chegaria em ~95% do mesmo
schema, pagando o risco de migrar histórico de tickets em produção por
essencialmente nenhum ganho de modelagem.

**O problema era visual.** O histórico de commits conta sozinho: `liquid glass
everywhere - vibrant wallpaper` seguido de dois `fix(theme)` tentando domar o
resultado. E o pedido veio junto com "usa o design do Questor BI, tá muito bom" —
que é a linguagem oposta do liquid glass.

Então o rewrite acontece **na UI**, e o schema evolui por migração aditiva (como
as 10 migrações manuais já existentes, todas idempotentes). Dado de produção
preservado por construção, sem script de cópia.

Único defeito real achado no schema: `app_settings.id` é `integer` no Drizzle e
`smallint` no banco — divergência entre o schema tipado e o real, a corrigir.

## Sistema de design (o que foi feito)

Substituição do tema escuro fixo "deep space" pela linguagem do [[Design]] do
[[Questor BI]]: superfície chapada, hairline discreta, número tabular, dois temas.

**Tokens em três camadas** — primitiva (única com hex), semântica (`--surface`,
`--ink`, `--hairline`) e de domínio (`--status-*`, `--prio-*`, `--sla-*`). O
componente consome a mais específica que couber, então mudar a cor de "andamento"
no sistema todo é editar uma linha.

A paleta categórica saiu de busca computada, não do olho — método em
[[Paleta categórica com estética por restrição]]. A auditoria de contraste
reprovou a primeira versão em 5 pontos e revelou um erro de modelagem que virou
[[Cor de gráfico e cor de texto pedem contrastes diferentes]].

`@theme inline` publica os tokens como utilitários Tailwind, então componente
escreve `bg-surface text-muted` em vez de `bg-[var(--bg-elev)]`. Colisão a evitar:
`@theme` já gera `border-hairline` como **cor**, então a espessura hairline
chama-se `border-thin`.

Validadores moram em `scripts/design/` — rodar a cada mexida na paleta.

## Aprendizados (viram notas atômicas)

- [[Paleta categórica com estética por restrição]]
- [[Cor de gráfico e cor de texto pedem contrastes diferentes]]
- [[Verificar no build de produção, não só em dev]] — os dois blocos de tema só
  foram dados como certos depois de conferir o CSS gerado pelo build.

## Próximos passos

- [ ] Reescrever os 57 arquivos restantes no vocabulário novo de tokens (13 ainda
      com `backdrop-blur` do liquid glass).
- [ ] Rebuild das telas nos padrões de [[Padrões de componentes de dashboard]]:
      sidebar em acordeão, KPI tiles, ChartCard, filtros na URL.
- [ ] Corrigir `integer` vs `smallint` em `app_settings.id`.
- [ ] Auditoria de contraste no CI, não só rodada à mão.
- [ ] Merge de `rewrite/v2` só depois de e2e passando.

## Links

- Mapa: [[Desenvolvimento]] · [[Design]]
- Design herdado de: [[Questor BI]]
