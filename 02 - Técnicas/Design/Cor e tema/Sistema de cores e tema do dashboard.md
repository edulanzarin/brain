---
tags: [tipo/atomica, camada/padrao, design, dev/frontend]
criado: 2026-07-18
---

# Sistema de cores e tema do dashboard

> Toda cor é um **token semântico** em CSS var, definido duas vezes (claro e escuro).
> Componente nunca usa hex — usa `var(--token)`. Trocar de tema é trocar `data-theme`.

## Os tokens

Superfícies e texto: `--page`, `--surface`, `--surface-2`, `--ink`, `--ink-2`, `--muted`,
`--grid` (linhas de gráfico), `--baseline` (eixos), `--hairline` (bordas, rgba com alpha).

Semânticos de dado:
- `--ent` (entradas, azul) e `--sai` (saídas, verde) — os dois lados de tudo.
- `--esp-1..5` + `--esp-outras` — paleta **categórica** (espécie, séries, fatias). Ordem fixa
  e validada (ver [[Validar paleta de gráficos antes de escolher cores]]).
- Estado: `--good`, `--critical`, `--warning`.

Valores reais (claro / escuro): ent `#2a78d6` / `#3987e5`, sai `#008300` / `#00a300`.
No escuro as cores são levemente mais claras/saturadas pra manter contraste no fundo escuro.

## Claro + escuro

- `:root, :root[data-theme="light"]` define o tema claro; `:root[data-theme="dark"]` o escuro.
  Cada bloco redefine os MESMOS tokens. `color-scheme` acompanha.
- No Tailwind v4, exponho os tokens com `@theme inline { --color-surface: var(--surface); … }`
  pra virarem classes utilitárias (`bg-surface`, `text-muted`, `border-hairline`).

## Sem flash no carregamento

Script inline no `<head>` (antes do render) lê o tema do `localStorage` (ou `prefers-color-scheme`)
e seta `document.documentElement.dataset.theme`. Assim a página já nasce no tema certo — nada de
piscar branco. Ver o padrão em [[Padrões de componentes de dashboard]].

## Conexões
- Princípios: [[Token semântico em vez de valor literal]] · [[Hierarquia por superfície, não por borda]]
- Ver também: [[Padrões de componentes de dashboard]] · [[Validar paleta de gráficos antes de escolher cores]]
- Mapa: [[Design]]
