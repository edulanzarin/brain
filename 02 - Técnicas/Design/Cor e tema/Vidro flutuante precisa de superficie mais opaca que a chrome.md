---
tags: [tipo/atomica, camada/padrao, design, dev/frontend]
criado: 2026-08-02
---

# Vidro flutuante precisa de superfície mais opaca que a chrome

> Numa UI de vidro, nem todo vidro é igual. A chrome estrutural pode ser arejada e
> transparente; um overlay que flutua **sobre o conteúdo** tem que ser opaco o
> bastante pra ler. São dois níveis de superfície, não um.

## O erro

Um token de vidro só (`--surface` translúcido, tipo 0.6–0.68 de alpha) usado tanto na
chrome quanto nos menus. Na chrome fica lindo — sidebar, topbar e painéis deixam o fundo
gradiente vazar de leve. Mas o **mesmo alpha num menu flutuante** (dropdown, "ver como",
modal, toast) deixa o conteúdo de trás atravessar o texto: o menu vira uma sopa ilegível,
ainda mais quando ele sobrepõe uma lista densa.

## A regra

Ter **duas camadas de superfície de vidro**:

- **Chrome / estrutura** (sidebar, cabeçalho, painéis fixos): alpha baixo, arejado. O que
  vaza atrás é o fundo da página, não conteúdo — pode ser transparente.
- **Overlay flutuante** (menu, dropdown, modal, toast, tooltip): alpha alto (~0.9), ainda
  com blur pra parecer fosco, mas opaco o suficiente pra o texto vencer o que estiver
  atrás. Um token só (`--surface-over`) e uma classe só (`.glass-over`) pra **todo** o
  que flutua — assim o vidro fica consistente em vez de cada popup inventar o seu.

O blur (`backdrop-filter`) ajuda na legibilidade, mas **não substitui o alpha**: blur
borra, não opacifica. Sobre conteúdo de alto contraste, só blur não salva.

## Cuidado com o override inline

Um `style={{ background: ... }}` no componente vence a classe de vidro e reintroduz o
alpha errado sem aparecer no CSS. Se um overlay continua transparente depois de trocar a
classe, procure o background inline. Visto em [[navetalks]] (ago/2026): o modal tinha
`.glass` trocado pra `.glass-over` mas seguia translúcido por causa de um
`style` inline antigo apontando pra `--surface`.

## Conexões
- Princípios: [[Hierarquia por superfície, não por borda]] · [[Token semântico em vez de valor literal]]
- Irmãs: [[Sistema de cores e tema do dashboard]] · [[Controles de filtro do dashboard]]
- Mapa: [[Design]]
