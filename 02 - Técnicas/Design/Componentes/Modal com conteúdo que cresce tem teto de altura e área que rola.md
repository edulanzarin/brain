---
tags: [tipo/atomica, camada/padrao, design, dev/frontend]
criado: 2026-08-06
---

# Modal com conteúdo que cresce tem teto de altura e área que rola

> Conteúdo ilimitado (timeline, lista, anotações) faz o modal crescer além da tela.
> O painel do modal ganha teto, e o que cresce rola numa área própria.

## O quê

Um modal centralizado com conteúdo que pode crescer (N anotações, N itens) estoura a
viewport e fica feio. Dois níveis de solução:

1. **Rede geral no componente Modal**: painel com `max-h` (ex.: 88dvh) e corpo
   `overflow-y-auto`. Nenhum modal ultrapassa a tela. O botão de fechar fica fixo no
   painel, fora da área que rola — senão some ao rolar.
2. **Área própria pro que cresce**: numa ficha larga, dividir em colunas (identidade |
   timeline) e dar à timeline `max-h` + `overflow-y-auto`. Ela rola sozinha e o resto
   fica parado. Mais largo cabe mais informação — bom quando o registro acumula histórico.

E na entrada do texto: **limite de caracteres** (hard stop no campo + revalidação no
servidor) pra não gravar uma bíblia; contador discreto, sem microcopy explicativo.

## Por que importa

O estado "muita coisa" também é um estado que precisa de desenho
([[Todo estado da tela tem visual]]), e cortá-lo com teto + scroll mantém a régua de
interface enxuta ([[Interface enxuta e compacta, sem desperdício de espaço]]). Vale pra
qualquer overlay: modal, drawer, popover.

## Conexões
- Princípio: [[Todo estado da tela tem visual]]
- Irmã: [[Blocos de dado - card, KPI e gráfico]]
- Visto em: [[Navehub]]
- Mapa: [[Design]]
