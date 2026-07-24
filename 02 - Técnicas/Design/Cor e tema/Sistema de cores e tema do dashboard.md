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

**Isso vale também pros NEUTROS, não só pras cores de dado.** Reusar o mesmo cinza de
texto secundário (`--muted`) e a mesma borda nos dois temas parece simétrico, mas no fundo
escuro o mesmo valor perde contraste perceptual — texto secundário e bordas somem. No
escuro os neutros sobem um degrau: muted mais claro (`#898781` → `#a3a19a`), hairline com
mais alpha (10% → 15%), e um passo a mais de separação em `--surface`/`--surface-2`.
A regra prática: **tema escuro não é o claro invertido — cada camada neutra precisa ser
recalibrada no fundo escuro** (visto no Navetech Hub, jul/2026: "às vezes fica difícil de ver").

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
