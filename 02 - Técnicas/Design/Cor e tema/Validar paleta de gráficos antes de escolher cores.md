---
tags: [tipo/atomica, camada/padrao, dev/frontend, design]
criado: 2026-07-18
---

# Validar paleta de gráficos antes de escolher cores

> Cor de gráfico é computável: rode um validador de contraste/daltonismo em vez de escolher cores "no olho".

## O quê

Ao fazer dashboards ([[Navetech Hub]]), a ordem certa é: primeiro escolher a **forma** do gráfico pelo trabalho do dado (magnitude, identidade, comparação), e a **cor por último**. As cores categóricas devem ser atribuídas em ordem fixa (nunca cicladas) e a paleta passar por checagens objetivas:

- banda de luminosidade e piso de croma (não pode "virar cinza"),
- separação entre pares adjacentes sob daltonismo (protan/deutan/tritan) — alvo ΔE ≥ 8,
- contraste mínimo contra a superfície (3:1), senão exige rótulo/legenda visível.

Regras que carrego: **um eixo só** (nunca dois eixos Y), sequencial = um tom claro→escuro, divergente = dois tons + cinza no meio, legenda sempre presente com 2+ séries, e texto nunca usa a cor da série (usa tokens de tinta).

## Por que importa

Evita o erro clássico de paleta bonita mas ilegível pra ~8% dos homens (daltonismo) ou em impressão P&B. "Não avalie ΔE de cabeça — compute." Serve pra qualquer visualização, em qualquer lib (Recharts, matplotlib, d3…).

## Conexões
- Princípio: [[Token semântico em vez de valor literal]]
- Ver também: [[Projetos]]
- Visto em: [[Navetech Hub]]
- Mapa: [[Design]]
