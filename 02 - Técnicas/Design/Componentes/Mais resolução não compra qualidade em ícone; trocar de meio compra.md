---
tags: [tipo/atomica, camada/padrao, design, armadilha]
criado: 2026-08-24
---

# Mais resolução não compra qualidade em ícone; trocar de meio compra

> Quando um ícone desenhado em código "não está bonito o bastante", o reflexo é
> aumentar a grade. Quase sempre é o lever errado: o que limita não é o número de
> pixels, é o **meio**. Pixel art tem valor pela restrição; pedir a ela curva
> macia, gradiente e brilho suave é brigar com aquilo que ela é.

## O que aconteceu

Referência na mesa: um ícone de ovo num ninho, com sombreamento e volume. Os
ícones existentes eram 32×32 desenhados em Python, e a hipótese óbvia foi "a grade
é pequena demais". Fui pra 48×48, com dois tons a mais na rampa e sombra de
contato.

Piorou. Três passadas depois, o de 32 ainda lia mais limpo: numa grade pequena
cada pixel é uma decisão, e ao dobrar a grade os mesmos elementos viraram
detalhes miúdos que se atropelam — luzinha de 3px vira mancha, cruzeta vira
borrão, silhueta perde a mordida. O ganho de área virou ruído.

O mesmo desenho refeito em **SVG** ficou melhor que os dois na primeira passada,
e ainda por cima é nítido em qualquer tamanho.

## A regra

Pergunte o que o desenho precisa, e escolha o meio por isso:

| O desenho precisa de… | Meio |
|---|---|
| curva macia, gradiente, brilho suave, escala livre | **vetor (SVG)** |
| aresta dura, paleta travada, estética retrô declarada | **pixel** |
| textura, pintura, foto | nenhum dos dois — precisa de modelo de imagem |

Pixel art com 4 tons por material e curva antisserrilhada não é pixel art melhor:
é vetor mal feito, pago em esforço manual. E vetor imitando pixel é pior ainda.

## O que mais vale lembrar

**A silhueta é o teste, e ele é barato.** Pinte tudo de preto e olhe: se o assunto
não se reconhece como mancha, nenhum sombreado salva. Vale nos dois meios, e
reprova antes de você gastar a tarde no acabamento.

**Julgue no tamanho de uso.** Um ícone de slot de 24px se julga a 24px, e não a
512. Folha de contato nos tamanhos reais é o que corta detalhe que vira papa.

**Não entregue arte que você não viu renderizada.** Foi o erro de fundo das três
passadas perdidas: cada ajuste era escrito no escuro e conferido depois. Arte
escrita em código falha de jeitos que o código não mostra — ordem de forma errada,
brilho que lê como furo, contorno comendo o miolo. Se o render discorda da
intenção duas vezes seguidas, o problema é o desenho, não a coordenada.

## Conexões
- Princípio: [[A variante de um controle muda a intenção, não o tamanho]]
- Irmã: [[Trocar arte por ícone de linha exige recalibrar tamanho, não só trocar o componente]] ·
  [[Arte de ícone se julga no tamanho de uso, e o acento é a massa]] ·
  [[Peça desenhada fora do DOM é uma segunda implementação do tema, e ela envelhece calada]]
- Visto em: [[piwdex2]]
- Mapa: [[Design]]
