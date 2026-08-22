---
tags: [tipo/atomica, camada/padrao, design, dev/frontend]
criado: 2026-08-22
---

# Área de toque cresce por pseudo-elemento, não pela caixa

> Link dentro de frase — rodapé, migalha, referência no meio de um parágrafo — tem a
> altura da linha: 12 a 22px. Está abaixo do piso de acessibilidade e não há como
> engordar a caixa: crescer o link **empurra a frase em volta**.

## A regra

Cresce a **área sensível**, não o elemento. Um pseudo-elemento fora do fluxo, só no
toque:

```css
@media (pointer: coarse) {
  .tap { position: relative; }
  .tap::after {
    content: "";
    position: absolute;
    left: 0; right: 0; top: 50%;
    min-height: 44px; height: 100%;
    transform: translateY(-50%);
  }
}
```

O link continua do tamanho da letra e o parágrafo não se mexe; o que muda é onde o toque
é aceito.

Três detalhes que custam retrabalho se passarem batido:

- **`position: relative` é obrigatório.** Sem ele o `absolute` escapa pro ancestral
  posicionado mais próximo e a área sensível aparece em outro lugar da tela.
- **Só no toque.** No mouse isso vira uma caixa invisível roubando `hover` dos vizinhos.
- **Só o eixo vertical.** `left/right: 0` mantém a largura do link; esticar na
  horizontal faz alvos vizinhos da mesma linha se sobreporem, e aí o toque acerta o
  errado — que é pior que o alvo pequeno.

## Onde se aplica

Onde o alvo é **texto corrido**: rodapé institucional, migalha de navegação, lista de
drops, "mostrar mais". Onde o alvo é **controle** (botão, campo, aba), o certo é o
degrau maior na primitiva — ver
[[Alvo de toque pergunta pelo apontador, não pela largura da janela]].

## Conexões
- Princípio: [[Todo estado da tela tem visual]]
- Irmã: [[Alvo de toque pergunta pelo apontador, não pela largura da janela]]
- Visto em: [[piwdex2]]
- Mapa: [[Design]]
