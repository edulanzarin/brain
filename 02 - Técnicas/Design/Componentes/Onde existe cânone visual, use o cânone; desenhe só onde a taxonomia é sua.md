---
tags: [tipo/atomica, camada/padrao, design]
criado: 2026-08-24
---

# Onde existe cânone visual, use o cânone; desenhe só onde a taxonomia é sua

> Antes de desenhar um símbolo, pergunte se o público **já tem esse símbolo na
> cabeça**. Se tem, desenhar um próprio não é caprichar — é cobrar um pedágio: a
> pessoa precisa aprender o seu para ler o que já sabia ler.

## Como o critério se aplica

A pergunta não é "o meu ficou bom?". É **de quem é a taxonomia**:

| a taxonomia é… | o que fazer |
|---|---|
| do domínio externo, e o público a conhece (tipo de pokémon, naipe de carta, bandeira de país, ícone de rede social) | **use o cânone** |
| sua, inventada pelo produto ("ataque especial", "ouro por abate", categoria de item) | **desenhe** |

Foi exatamente a linha que separou dois lotes no mesmo site: os 18 tipos passaram
a ser os símbolos oficiais dos jogos, e os glifos de domínio (vida, ataque, ouro,
gema, TM, nível) continuaram desenhados à mão — porque "ataque especial" não tem
símbolo que ninguém decorou.

## O argumento que NÃO é o certo

"O pronto é mais bonito." Pode não ser — os desenhados à mão estavam razoáveis, e
em alguns casos liam melhor no miúdo. Trocar por estética seria opinião contra
opinião.

O argumento que decide é **reconhecimento**, e ele é assimétrico: o cânone já está
aprendido, o seu não. Isso não empata com traço melhor.

## O segundo ganho, que é o que se sente sem saber nomear

**Consistência óptica.** Dezoito peças com o mesmo peso visual é a parte mais
difícil de desenhar à mão e a que mais denuncia quando falha — o relato chega como
*"ficou meio torto"*, sem apontar qual. Um conjunto curado já resolveu isso, e
resolver sozinho custa passadas de calibragem que ninguém orça.

## Adaptar o pronto ao seu tema

Conjunto de terceiro costuma vir com decisões embutidas que brigam com o seu
sistema — no caso, cada ícone trazia um **círculo de fundo colorido** e a cor
cravada. Extraia só o glifo e deixe a cor vir de `currentColor`: assim ele entra
na sua escala de cor em vez de trazer outra.

E **mantenha o `viewBox` do original**. Reescalar à mão para "padronizar" só
introduz erro de arredondamento, multiplicado pelo número de peças.

## A obrigação que vem junto

Licença. MIT pede o aviso de copyright junto de "porções substanciais" — dezoito
ícones são substanciais. O arquivo de licença vai **na pasta do componente**, não
num README que ninguém abre ao mover o código.

## Conexões
- Princípio: [[Nota carrega só o que a pessoa não sabe]]
- Irmã: [[Mais resolução não compra qualidade em ícone; trocar de meio compra]] ·
  [[Trocar arte por ícone de linha exige recalibrar tamanho, não só trocar o componente]]
- Visto em: [[piwdex2]]
- Mapa: [[Design]]
