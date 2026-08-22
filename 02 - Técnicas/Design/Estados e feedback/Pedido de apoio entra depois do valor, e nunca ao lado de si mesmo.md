---
tags: [tipo/atomica, camada/padrao, design, dev/frontend]
criado: 2026-08-21
---

# Pedido de apoio entra depois do valor, e nunca ao lado de si mesmo

> Num projeto de graça, o pedido de apoio tem duas formas ao mesmo tempo — faixa
> no rodapé e balão flutuante — e três decisões que definem se ele é aceitável:
> **quando aparece** (depois de a pessoa usar), **quanto dura o "agora não"** (um
> prazo, não a eternidade nem a sessão) e **o que acontece quando os dois se
> encontram na mesma tela** (um some).

## O caminho de graça vem primeiro

Quem mantém ferramenta de fã costuma ter dois caminhos: dinheiro (link de
pagamento, valor escolhido por quem paga) e um caminho que **não custa nada a
quem apoia** — no [[piwdex2]], usar o código de indicação do dono ao comprar
diamante dentro do jogo, que não muda o preço da compra.

O de graça é o que a maioria consegue dar. Ele vem primeiro no texto e existe
sempre; o do dinheiro é o extra. Inverter a ordem gasta a única frase que a pessoa
vai ler pedindo o que quase ninguém vai dar.

**Corolário duro: link que ainda não existe não vira botão.** A URL de pagamento
nasce vazia numa constante, e vazia ela **não desenha botão nenhum** — nada de
"em breve", nada de `href="#"`. Botão que abre nada gasta exatamente a confiança
de que o pedido depende. Preencher a constante acende o botão no rodapé e no balão
ao mesmo tempo.

## O gatilho do balão é uso, não chegada

Aparecer no `onload` é pedir antes de entregar: a pessoa fecha sem ler, e o
pedido queimou. O gatilho que sobreviveu: **tempo na página E prova de uso** (30
segundos e ter rolado). Quem entra e sai em cinco segundos nunca vê — e é isso
mesmo.

Fechar guarda um **carimbo de tempo** e vale 30 dias. Fechar "de vez" perde o
pedido em quem volta todo dia; fechar "até recarregar" é assédio com quem volta na
sessão seguinte.

## Os dois nunca aparecem juntos

Faixa e balão pedem a mesma coisa. Com o rodapé em cena, o balão fica em cima do
próprio botão que ele quer que a pessoa clique — o site pedindo duas vezes na
mesma tela. Um `IntersectionObserver` no rodapé esconde o balão enquanto ele
estiver visível; **esconder não é dispensar** — rolou pra cima, ele volta.

`IntersectionObserver` e não comparação de `scrollY`: a altura do rodapé muda com
a largura da tela, e conta na mão erra justamente no celular, onde ele é mais alto.

## Três armadilhas que só aparecem testando

1. **O listener fecha sobre o primeiro render.** O balão sumia no X e **voltava no
   primeiro scroll seguinte**: o listener registrado uma vez continuava lendo o
   `dispensado` do render inicial. A decisão de "já dispensou" tem que morar num
   `ref`, que o listener lê sempre atualizado.
2. **O observer tem que nascer junto com o componente**, não depois de o balão
   aparecer — senão quem já está no fim da página vê o balão piscar antes de sumir.
3. **Superfície flutuante é a opaca.** Usar a superfície de conteúdo (vidro
   arejado) fez o sprite da página atravessar o texto do pedido. Ver
   [[Vidro flutuante precisa de superfície mais opaca que a chrome]] — e mesmo a
   opaca do sistema, em 94%, deixa passar número em cor viva: no flutuante, 100%.

## Conexões
- Princípio: [[Todo estado da tela tem visual]] · [[Nota carrega só o que a pessoa não sabe]]
- Depende de: [[Vidro flutuante precisa de superfície mais opaca que a chrome]]
- Irmã: [[Modal com conteúdo que cresce tem teto de altura e área que rola]]
- Visto em: [[piwdex2]]
- Mapa: [[Design]]
