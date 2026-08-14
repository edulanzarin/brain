---
tags: [tipo/atomica, camada/principio, dev/frontend]
criado: 2026-08-14
---

# Coerência em geração vem de âncora, não de liberdade

> Sempre que se **gera** conteúdo (procedural ou por IA), identidade e coerência
> vêm de **prender a geração a uma âncora forte** — peças curadas, uma referência,
> uma semente — não de gerar livre a cada vez e torcer pra bater.

## A ideia

Geração livre é sedutora: descreve e sai pronto. Mas cada chamada parte do zero,
então o resultado **deriva** — a segunda imagem não é o mesmo personagem da
primeira, o segundo frame não combina com o anterior. Quanto mais liberdade, mais
deriva. A coerência não nasce de um prompt melhor; nasce de **reduzir os graus de
liberdade**: dar ao gerador uma âncora que ele é obrigado a respeitar.

A âncora tem várias formas — todas o mesmo movimento:
- **compor peças curadas** em vez de gerar o todo (o resultado herda a coerência
  das peças, não a sorte do gerador);
- **referência + peso alto** ("siga esta imagem, e siga forte");
- **semente fixa** (mesma entrada → mesma saída).

O trade-off é real: mais âncora, menos variedade. O ponto de ajuste é o mínimo de
liberdade que ainda entrega variação útil sem perder a identidade — não o máximo de
liberdade que às vezes acerta.

## Onde já apareceu (dois casos, mesma lição)

- **Compositor de sprites do GDD**: em vez de gerar um spritesheet por espécie
  (inconsistente, caro, impossível de moderar), esqueletos de arquétipo desenhados à
  mão + biblioteca de peças que o DNA seleciona. Coerência **por construção**: a
  evolução herda o rig do pai e visivelmente descende dele.
- **Animar personagem no PixelLab**: `animate-with-text` com o `image_guidance_scale`
  default (1.4) larga a referência e "aluciná" um guerreiro diferente a cada
  direção; subindo pra ~4.0 ele gruda na referência e mantém a identidade entre
  frames e direções. Ver [[PixelLab só mantém o personagem ao animar com image guidance alto]].
- **Cena montada de fontes diferentes não casa**: personagem LPC + props gerados no
  PixelLab no mesmo lobby brigaram (escala de pixel, paleta, contorno, luz
  diferentes) — a cena parecia colada de dois jogos. A "âncora" aqui é a **família
  de assets única**: refazer chão e props no próprio LPC (mesma origem do
  personagem) uniu tudo de uma vez. Coerência vem de restringir a FONTE de
  variação, mesmo quando não se gera nada — só se monta.

## Conexões
- Irmã: [[A definição em dado dirige o comportamento, não um caso no código]]
- Visto em: [[Idle Game]]
- Mapa: [[Base]]
