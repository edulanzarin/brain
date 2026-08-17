---
tags: [tipo/atomica, camada/padrao, dev/backend, armadilha]
criado: 2026-08-17
---

# Avaliação por comparáveis segmenta a régua por faixa, não mediana única

> Estimar valor por "mediana de preço-por-unidade do mercado inteiro" funciona no
> meio da distribuição e ESMAGA a cauda: quando o preço cresce mais que linear com o
> atributo (raridade, qualidade), o item de elite precisa ser comparado com elite —
> a régua se segmenta por faixa, como avaliação de imóvel por comparáveis do bairro.

## O problema

O mercado é dominado pelo item comum e barato. Uma régua única (mediana global de
preço/unidade) aprende o preço do comum e projeta linearmente — e o item raro, cujo
preço real é desproporcional (só sai de mecânica cara, oferta mínima), sai avaliado
por uma fração do que vale. No piwdex: um Haunter Q2.105 (qualidade que só sai de
breeding) avaliado como se fosse a média do lixão.

## A solução

1. **Segmentar a régua por faixa do atributo que dobra o preço** (bandas de
   qualidade, com cortes nos limiares que o domínio já usa) e por dimensões que são
   OUTRO mercado (shiny vs não-shiny).
2. **Cadeia de fallback com amostra mínima**: espécie+faixa → faixa global → régua
   geral. Cada degrau só responde com N amostras; senão desce.
3. **Teto de sanidade pelos anúncios reais**: se existe oferta ativa igual-ou-melhor
   mais barata que a estimativa, a estimativa desce pra ela — ninguém paga mais do
   que o mercado cobra agora.
4. Expor a **proveniência** (qual degrau respondeu, com quantas amostras) — a UI
   pode sinalizar estimativa fraca.

## O que mais vale lembrar

- Mediana continua sendo o agregador certo dentro da faixa (resiste a preço troll).
- O sinal de que a régua está errada é reclamação assimétrica: o meio parece ok e
  os extremos saem absurdos.

## Conexões
- Princípio: (folha isolada — candidata se aparecer segundo caso)
- Visto em: [[piwdex]]
- Mapa: [[Backend]]
