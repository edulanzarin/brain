---
tags: [tipo/atomica, camada/padrao, design, dev/frontend]
criado: 2026-08-22
---

# Alvo de toque pergunta pelo apontador, não pela largura da janela

> Interface densa nasce com medida de mouse: controles de 28 a 40px de altura, que o
> ponteiro acerta com folga. O dedo não. A correção reflexa é pendurar o tamanho maior
> num breakpoint de largura — e o breakpoint responde a pergunta errada.

## A regra

O que decide o tamanho do alvo é **`@media (pointer: coarse)`**, não `max-width`:

- desktop com a janela estreita continua no mouse e não precisa de 44px — engordar ali
  só estraga a densidade de quem está sentado;
- tablet grande com toque **precisa**, e nenhum breakpoint de largura pega esse caso.

No Tailwind v4 isso é a variante `pointer-coarse:`, aplicável direto na primitiva.

Dois pisos, e eles não são a mesma coisa:

- **24px** é o piso **duro** da WCAG 2.2 AA (critério 2.5.8). Não é conforto, é norma.
- **44px** é a recomendação de conforto da Apple. Ficar entre 24 e 44 é uma decisão de
  densidade legítima; ficar abaixo de 24 é defeito.

A regra mora na **primitiva**, num lugar só: o enum de tamanho continua fechado (quem
chama não escolhe altura), o que muda é o valor de cada degrau por apontador. Assim o
degrau denso do desktop fica intacto e o risco de regressão é zero.

## Dois detalhes que só aparecem medindo

**Campo de texto:** `font-size` abaixo de 16px faz o iOS dar zoom ao focar, e a página
inteira sai do lugar sozinha. O piso de 16px no toque não é estética.

**Input dentro de casca:** `align-items: center` centraliza sem esticar, então um
`<input>` dentro de um campo de 44px continua medindo 22. E tocar a folga da casca **não
foca um input** — só um `<label>` faria isso. Sem `self-stretch`, o alvo real é a altura
da letra.

## Como verificar

Medir a **caixa** do elemento não basta e engana nos dois sentidos: ela ignora área
sensível criada por pseudo-elemento e ignora oclusão. O que vale é perguntar ao
navegador quem responde num ponto: varra pontos ao redor do centro do alvo e cheque
`document.elementFromPoint`. Num site real isso levou "71 alvos abaixo de 24px" para
zero — sendo que boa parte dos 71 já estava resolvida e o medidor é que não enxergava.

Cuidado com dois falsos positivos: link de pular conteúdo (`sr-only`, 1x1 por desenho) e
`input` escondido atrás de `<label>` estilizado — nos dois o alvo é outro elemento.

## Conexões
- Princípio: [[A variante de um controle muda a intenção, não o tamanho]]
- Irmã: [[Área de toque cresce por pseudo-elemento, não pela caixa]] ·
  [[Fila de campos alinha por altura fixa de controle, não por items-end]]
- Visto em: [[piwdex2]]
- Mapa: [[Design]]
