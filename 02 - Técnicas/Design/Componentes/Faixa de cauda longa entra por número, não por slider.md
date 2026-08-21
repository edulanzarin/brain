---
tags: [tipo/atomica, camada/padrao, design, armadilha]
criado: 2026-08-21
---

# Faixa de cauda longa entra por número, não por slider

> O slider de dois polegares é o controle bonito de faixa e o reflexo automático pra
> qualquer filtro `min..max`. Ele só funciona quando o dado é razoavelmente uniforme:
> numa grandeza de cauda longa, metade do catálogo se espreme nos primeiros pixels do
> trilho e o polegar não consegue separar 30 de 900.

## O problema

O trilho é linear em PIXEL. A distribuição do dado quase nunca é.

Medido num catálogo de 428 itens: preço de venda vai de 1 a 1.000.000, mediana 40, p75
180, p95 5.000. Ou seja, **75% dos itens vivem nos primeiros 0,018% da barra**. Arrastar
o polegar um pixel pula centenas de itens na região onde estão quase todos, e atravessa
o vazio na região onde não há quase nenhum. O controle existe, responde, anima — e não
consegue expressar a pergunta que o usuário tem ("itens de até 200").

É o mesmo erro de régua que [[A régua de um medidor é percentil, não máximo]] descreve na
saída, agora na entrada: escolher a escala pelos extremos em vez de pela distribuição.

## A solução

O controle segue a forma do dado, não o tipo do campo:

- **Escala curta e uniforme** (0–100%, nível 1–550, contagem 0–346): slider, que é onde
  ele brilha — explorar arrastando, sem saber o número exato.
- **Cauda longa** (preço, receita, tamanho de arquivo, tempo): **par de números** ("de"
  / "até"). Quem filtra por preço já tem um número na cabeça; digitar 200 é mais rápido
  que caçar 200 num trilho de um milhão.
- **Os dois juntos** quando a faixa é média: slider pra explorar, números pra precisar —
  são dois gestos da mesma pergunta.
- Diga o extremo por escrito perto do campo ("vai de 1 a 1M"), senão o usuário não tem
  como calibrar o que digitar.

Escala logarítmica no trilho resolve no papel e falha na mão: o usuário não sabe que a
escala é log, e a posição do polegar deixa de significar o que ele espera.

## O que mais vale lembrar

O teste é de um minuto: **ordene os valores e veja onde cai o p75**. Se ele está no
primeiro décimo do trilho, o slider está errado, por mais que a tela pareça pronta.

Isso não é sobre sliders — é sobre a régua de qualquer controle contínuo. Vale pra
zoom de timeline, seletor de intervalo de datas e barra de progresso não-linear.

## Visto em

No piwdex2, a Pokédex filtra "valor de venda" com slider e o extremo é 6,5 bilhões:
o controle nasceu inútil e ninguém percebeu, porque o `NumberRange` ao lado resolvia na
prática. Na página de Itens a mesma decisão foi tomada de olhos abertos — preço entrou
só por número, enquanto chance (0–100%), nível e contagem de fontes ficaram com slider.

## Conexões
- Princípio: [[A régua sai da distribuição, não dos extremos]]
- Irmã: [[A régua de um medidor é percentil, não máximo]] · [[Controles de filtro do dashboard]]
- Visto em: [[piwdex2]]
- Mapa: [[Design]]
