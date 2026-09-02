---
tags: [tipo/atomica, camada/padrao, design]
criado: 2026-09-02
---

# Peça que só existe aberta se mostra pelo painel extraído, não por réplica

> O catálogo precisa mostrar o modal ABERTO — quem vê só um botão "Abrir modal"
> conclui, com razão, que o modal não existe. Mas desenhar uma versão estática à
> mão cria uma segunda implementação da peça, e é a segunda que envelhece calada.

## O problema

Modal, gaveta, menu e dica só existem enquanto estão abertos, e o modal ainda
por cima monta num portal com posição fixa — não dá para encaixá-lo numa página
de catálogo. A saída óbvia é remontar a moldura com as mesmas classes ali no
catálogo. Funciona no dia em que se escreve, e a partir do dia seguinte o
catálogo publica o visual de uma virada atrás: mexer no modal real não quebra a
réplica, então nada denuncia.

## A solução

Extrair o **painel** — o que a peça é visualmente — e montá-lo de dois jeitos:

```tsx
export function PainelModal({ titulo, rodape, children, ... }) { /* a moldura */ }

export function Modal({ aberto, ... }) {           // o uso real
  return createPortal(<div className="fixed inset-0 …">
      <div onClick={aoFechar} className="fixed inset-0 bg-…" />
      <PainelModal …>{children}</PainelModal>
    </div>, document.body);
}
```

O catálogo monta `PainelModal` direto na página. É **uma implementação, dois
lugares de montagem**: qualquer mudança na peça aparece no catálogo na mesma
hora, porque é a peça.

O que fica de fora do painel é exatamente o comportamento que não faz sentido
numa página: o portal, o fundo escurecido, a trava de rolagem, o Escape e o
retorno de foco.

## O que mais vale lembrar

- Vale para menu (`PainelMenu`) e dica (`BalaoDica`) pelo mesmo motivo.
- O botão que abre a peça de verdade **continua no catálogo, junto** — o painel
  estático mostra a forma, e o gatilho prova que o comportamento existe.
- A tentação de replicar aparece sempre que a peça depende de estado ou de
  posicionamento global. A pergunta que resolve: *o que aqui é aparência e o que
  é comportamento?* A aparência sempre pode ser extraída.

## Conexões
- Princípio: [[Catálogo de componentes é contrato vivo, não documentação]]
- Irmã: [[Peça desenhada fora do DOM é uma segunda implementação do tema, e ela envelhece calada]] · [[Modal com conteúdo que cresce tem teto de altura e área que rola]] · [[Peça de grade mostrada sozinha vai centrada num palco]]
- Visto em: [[Navetech Hub]]
- Mapa: [[Design]]
