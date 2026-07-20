---
tags: [tipo/atomica, camada/padrao, dev/frontend, armadilha]
criado: 2026-07-18
---

# router.replace do Next falha no build de produção

> Atualizar query params com `router.replace` do `next/navigation` funcionou em dev mas não mudava a URL no build de produção; trocar por `window.history.replaceState` resolveu.

## O quê

No [[Questor BI]] (Next.js 16, App Router), os filtros escreviam o estado na URL via `router.replace(\`?\${params}\`)`. Em `next dev` funcionava; em `next build && next start` o clique não alterava a URL nem refazia as queries — falha **silenciosa**, sem erro no console.

Causa provável: o `replace` tentando refazer o RSC/navegação e sendo engolido. A correção foi usar a API nativa do browser:

```ts
window.history.replaceState(null, "", `${pathname}?${params.toString()}`);
```

O Next 16 sincroniza `useSearchParams` com `history.replaceState`/`pushState` nativos, então os componentes reagem à mudança sem round-trip no servidor — e de quebra ficou mais rápido (filtro puramente client-side).

## Por que importa

Bug que só aparece em produção é traiçoeiro: passa em todo teste manual em dev. Reforça a regra de **sempre verificar no build de produção**, não só no dev server. Ver [[Verificar no build de produção, não só em dev]].

## Conexões
- Princípio: [[Verificar no build de produção, não só em dev]]
- Ver também: [[Verificar no build de produção, não só em dev]]
- Visto em: [[Questor BI]]
- Mapa: [[Frontend]]
