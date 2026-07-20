---
tags: [tipo/atomica, design, dev/frontend, conceito, projeto/navedesk]
criado: 2026-07-19
---

# Paleta categórica com estética por restrição

> Maximizar ΔE puro produz paleta acessível e feia. O jeito certo é inverter os
> papéis: **estética vira restrição, acessibilidade vira objetivo**.

## O problema

[[Validar paleta de gráficos antes de escolher cores]] manda computar em vez de
escolher no olho. Mas se a busca for só "maximizar o menor ΔE2000 entre os pares",
o otimizador entrega exatamente isso — e nada mais. No [[NaveDesk]] a primeira
busca livre devolveu rosa neon `#fe4191`, oliva `#9f8f14` e lima `#bbe40d`:
ΔE mínimo de 19,3, impecável na métrica, impossível de usar num dashboard.

O motivo é que ΔE não sabe nada sobre harmonia. Cores muito separadas em
luminosidade e croma são, por construção, cores que brigam entre si.

## A inversão

Em vez de otimizar cor livre e torcer pelo resultado:

1. **Fixar as famílias de matiz** que se leem bem num dashboard (azul, teal,
   âmbar, laranja/vermelho, violeta, neutro), cada uma como uma faixa de matiz
   em CIELCh.
2. **Limitar o croma** por família — teto baixo mata mostarda e magenta neon
   antes de eles existirem.
3. **Buscar dentro dessas caixas** a combinação de luminosidade e croma que
   maximiza o menor ΔE2000 sob visão normal + protan + deutan + tritan.

## Saturar o objetivo

O detalhe que faz a diferença: **parar de perseguir ΔE quando ele já é
suficiente**. Acima de ~12 as cores são inconfundíveis em qualquer visão, e
ganho adicional só é pago em cor berrante. Então o score satura:

```
score = min(ΔE_pior_par, 12) − desvio_de_croma / 1000
```

Enquanto o conjunto não separa, ΔE manda. Depois do teto, quem decide é a
estética (croma perto de um alvo confortável). O peso pequeno no segundo termo
garante que beleza nunca compre separação abaixo do teto.

No NaveDesk isso levou de 19,3 (feio) para 12,0 no claro e 12,2 no escuro — ainda
50% acima do alvo de 8, e com paleta que dá pra olhar.

## Detalhes que custaram tempo

- **Luminosidade também separa.** Dicromata perde um eixo de matiz; se todas as
  cores tiverem o mesmo L*, elas colapsam. A busca precisa varrer L*, não só H.
- **Rejeitar fora do gamut, não fazer clamp.** Clampar RGB distorce a cor e
  invalida o ΔE que você acabou de calcular.
- **Validar com um script separado do que gerou.** O gerador e o validador
  compartilharem bug é fácil demais; rodar o validador independente no fim pegou
  a paleta final honestamente.

## Conexões
- Faz parte de: [[Design]]
- Origem: [[NaveDesk]]
- Pressuposto: [[Validar paleta de gráficos antes de escolher cores]]
- Ver também: [[Cor de gráfico e cor de texto pedem contrastes diferentes]],
  [[Sistema de cores e tema do dashboard]]
