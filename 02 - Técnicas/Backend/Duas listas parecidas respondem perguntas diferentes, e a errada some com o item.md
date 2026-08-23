---
tags: [tipo/atomica, camada/padrao, dev/backend, armadilha]
criado: 2026-08-24
---

# Duas listas parecidas respondem perguntas diferentes, e a errada some com o item

> Quando um sistema externo expõe o mesmo tipo de coisa por duas rotas —
> `/shop` e `/balls`, catálogo e inventário, plano e assinatura —, é tentador
> escolher uma e usá-la em todo lugar. As duas trazem id, nome e ícone; a
> diferença aparece só na ausência. E ausência não dá erro: o item simplesmente
> não está na lista, e ninguém vê o que não foi renderizado.

## O caso

No [[piwdex2]] o painel do robô tinha dois seletores de pokébola: qual usar na
captura automática, e qual repor quando acabar. Os dois liam `/api/game/shop`.

A Idle Ball nunca aparecia. Ela existe na conta, é a bola infinita, e é
justamente a que se quer deixar ligada na captura — só que **não está à venda**,
então não está no catálogo da loja. O bug não tinha exceção, log nem tela de
erro: era um `<option>` a menos.

As duas rotas respondem perguntas diferentes, e cada seletor faz uma delas:

| Pergunta | Fonte | O que ela tem a mais |
|---|---|---|
| "com o que capturar?" | `/api/game/balls` | bola infinita, bola de evento, bola fora de venda |
| "o que posso comprar?" | `/api/game/shop` | preço |

Uma lista tem preço e não tem estoque; a outra tem estoque e não tem preço. Nem
uma é superconjunto da outra, e é por isso que "escolher a mais completa" não era
uma saída.

## Como se pega antes

- **Nomeie a lista pela PERGUNTA, não pela rota.** `bolasDaConta` e `bolasDaLoja`
  se recusam a ser trocadas uma pela outra; `bolas` aceita qualquer coisa.
- **Procure o item que só existe em uma.** A pergunta de teste é "o que esta
  fonte NÃO traz?", e ela costuma ter resposta rápida: o gratuito, o infinito, o
  descontinuado, o de evento.
- **Desconfie de acerto por interseção.** Se as duas listas coincidem em 90% dos
  itens, o bug fica escondido durante todo o desenvolvimento e aparece para o
  usuário que usa o item do 10%.
- Vale para qualquer par catálogo/posse: itens à venda x itens na mochila, planos
  ofertados x plano assinado, features do produto x features da conta.

## Conexões
- Princípio: [[Estado mutável se lê da fonte no uso, não de cópia guardada]]
- Irmã: [[Campo que a normalização não copia vira número errado, não erro]] (o
  mesmo silêncio, um nível abaixo: lá o campo some, aqui a linha inteira)
- Visto em: [[piwdex2]]
- Mapa: [[Backend]]
