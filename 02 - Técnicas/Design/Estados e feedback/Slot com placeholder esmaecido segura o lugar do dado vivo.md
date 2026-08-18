---
tags: [tipo/atomica, camada/padrao, design, dev/frontend]
criado: 2026-08-18
---

# Slot com placeholder esmaecido segura o lugar do dado vivo

> O contêiner do valor renderiza **sempre**; sem dado, mostra "—" esmaecido no
> mesmo lugar. A chegada do valor troca o conteúdo do slot, nunca o layout.

## O quê

O dialeto concreto (Tailwind/CSS) para telas alimentadas por stream, onde valores
aparecem e somem o tempo todo:

- **Classe `slot-empty`** (`color: var(--text-dim); opacity: 0.45`): o mesmo span
  alterna entre valor colorido e "—" esmaecido —
  `<span className={x ? "text-green" : "slot-empty"}>{x ?? "—"}</span>`.
- **Linha de altura fixa** (`h-11 items-center`) com slots, no lugar de
  `flex-wrap` de blocos condicionais. Se a linha aperta no mobile, ela rola na
  horizontal (`overflow-x-auto` + `shrink-0`), nunca quebra pra segunda linha.
- **`invisible` em vez de condicional** para ícone/aviso intermitente: o elemento
  fica no fluxo ocupando o espaço, só invisível. Como o texto real é renderizado
  (apenas oculto), a dimensão dos dois estados é idêntica por construção.
- **Badge de contagem é `absolute`** no canto do item — aparecer não empurra o
  vizinho.
- **Número que cresce muito** (dinheiro, XP/h) usa notação compacta
  (`Intl.NumberFormat("pt-BR", { notation: "compact" })` → "1,2 mi") com o valor
  cheio no `title`, mais `min-w` e `tabular-nums` no slot — a largura fica
  limitada e previsível.
- Botão que "não se aplica agora" vira `disabled` no mesmo lugar, não some. Slot
  de ação com estados mutuamente exclusivos mostra **um por vez** no mesmo
  contêiner de altura fixa (prioridade explícita), nunca empilha banners.
- Cards da mesma grade recebem `min-h` e linhas com `truncate`: o card com mais
  dados não fica mais alto que o vizinho.

## Por que importa

No [[piwdex]], a área VIP inteira é alimentada por um stream SSE — sem isso, cada
evento do robô reorganizava a tela: o HUD sticky mudava de altura, colunas
trocavam de tamanho (`className={x ? "flex-1" : ""}`), banners de alerta
empilhavam empurrando o painel. Com slots, a tela é uma grade parada em que os
números "acendem".

O esqueleto de carregamento é o mesmo padrão para a primeira pintura; este cobre
o resto da vida da tela.

## Conexões
- Princípio: [[Dado que chega preenche espaço reservado, não empurra a tela]]
- Irmã: [[Esqueleto de carregamento imita a forma do conteúdo]]
- Visto em: [[piwdex]]
- Mapa: [[Design]]
