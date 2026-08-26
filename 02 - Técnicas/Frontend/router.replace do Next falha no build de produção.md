---
tags: [tipo/atomica, camada/padrao, dev/frontend, armadilha]
criado: 2026-07-18
---

# router.replace do Next falha no build de produção

> Atualizar query params com `router.replace` do `next/navigation` funcionou em dev mas não mudava a URL no build de produção; trocar por `window.history.replaceState` resolveu.

## O quê

No [[Navetech Hub]] (Next.js 16, App Router), os filtros escreviam o estado na URL via `router.replace(\`?\${params}\`)`. Em `next dev` funcionava; em `next build && next start` o clique não alterava a URL nem refazia as queries — falha **silenciosa**, sem erro no console.

Causa provável: o `replace` tentando refazer o RSC/navegação e sendo engolido. A correção foi usar a API nativa do browser:

```ts
window.history.replaceState(null, "", `${pathname}?${params.toString()}`);
```

O Next 16 sincroniza `useSearchParams` com `history.replaceState`/`pushState` nativos, então os componentes reagem à mudança sem round-trip no servidor — e de quebra ficou mais rápido (filtro puramente client-side).

## Por que importa

Bug que só aparece em produção é traiçoeiro: passa em todo teste manual em dev. Reforça a regra de **sempre verificar no build de produção**, não só no dev server. Ver [[Verificar no build de produção, não só em dev]].

## Onde ele NÃO falha, e por que isso importa

Em 25/08/2026 o [[piwdex2]] fez o caminho inverso: as dez telas de ferramenta
escrevem o filtro na URL com `router.replace(\`${pathname}${busca}\`, { scroll:
false })`, e a nota daqui sugeria que todas estivessem quebradas em produção sem
ninguém saber.

Foram medidas. Build de produção (`next build && next start`), Chrome headless
dirigido pelo DevTools Protocol, valor injetado no input pelo setter nativo do
`HTMLInputElement` com evento borbulhando — que é o único jeito de mexer num
input controlado do React por fora. A URL acompanhou: `?q=ledian` apareceu, e a
tela filtrou junto.

Ou seja, **o defeito não é do `router.replace` em geral**. É de alguma condição
que o Navetech Hub tinha e o piwdex2 não, e enquanto ela não estiver isolada esta
nota descreve um SINTOMA, não uma regra. O que fica:

- Não saia trocando `router.replace` por `history.replaceState` preventivamente:
  metade das telas de um projeto pode estar certa.
- Quando desconfiar, MEÇA — e a medição é barata: são vinte linhas de script
  contra o DevTools Protocol, e ela responde em segundos o que print de tela
  nenhum responde (a falha é silenciosa: a tela filtra certo e só a URL fica pra
  trás).
- `history.replaceState` continua sendo a saída boa quando o sintoma aparecer,
  pelo motivo que já estava escrito: o Next 16 sincroniza `useSearchParams` com
  ele e o filtro fica client-side de verdade.

## Conexões
- Princípio: [[Verificar no build de produção, não só em dev]]
- Ver também: [[Verificar no build de produção, não só em dev]]
- Visto em: [[Navetech Hub]]
- Mapa: [[Frontend]]
