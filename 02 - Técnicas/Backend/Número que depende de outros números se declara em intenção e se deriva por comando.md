---
tags: [tipo/atomica, camada/padrao, dev/backend, armadilha]
criado: 2026-09-03
---

# Número que depende de outros números se declara em intenção e se deriva por comando

> Vida de inimigo depende do DPS do jogador, que depende da arma, do jutsu e de
> quatro constantes de fórmula. Escrito à mão, ele envelhece calado na primeira
> mexida em qualquer uma dessas coisas.

## A regra

Quando um número é **consequência** de outros, não se escreve o número: escreve-se a
intenção que ele realiza, e um comando resolve o valor.

```
intencao.ts        editado à mão   "o inimigo do meio do rank dura 4 s"
                                   "atravessar o rank custa 9 h"
      |  npm run calibrar
      v
calibragem.json    GERADO          vidaBaseInimigo: 202, xpPorVida: 0,034
      |  npm run balanco
      v
relatório          confere se a intenção se cumpriu
```

O que muda: mexer em arma, jutsu ou constante deixa de exigir reajuste manual de 24
inimigos e 6 bosses. Roda o calibrador e a curva reencaixa.

## O arquivo derivado é commitado, e é isso que o torna revisável

`calibragem.json` entra no git. Não é cache: é o **valor acordado**, e o diff mostra
o efeito de cada mexida na intenção — "troquei a espada e a vida do inimigo de Kage
subiu 3x" aparece na revisão, não na sessão de jogo. Gerado e ignorado seria
irreprodutível; gerado e commitado é auditável.

Por isso o arquivo carrega, no próprio conteúdo, a frase que diz para não editá-lo.

## A intenção precisa de botão legível, não de número cru

Primeira tentativa: o boss declarava `defesa: 62000`. A curva de redução é
assintótica — `def / (def + 60 + 8·nível)` —, então 62000 virou **98,6% de corte** e
o boss ficou invencível sem ninguém perceber. O número era editável e era inútil.

Trocado por `reducaoAlvo: 0,68` ("ele corta 68%"), com a defesa derivada pelo inverso
da fórmula. O botão passou a dizer o que faz.

Vale para toda grandeza que passa por curva não linear: declare o **efeito**, derive
o parâmetro.

## O ponto fixo que não existe

A derivação só fecha em uma passada porque o valor derivado **não entra** no cálculo
que o produz: a vida do inimigo não afeta o DPS do jogador. Antes de escrever um
calibrador, confira isso — se entrar, é ponto fixo e precisa de iteração ou de
bisseção, e a conta de uma passada mente.

## Como se descobre que faltava isso

A primeira leitura do relatório reprovou 26 regras de uma vez, e uma delas era o
boss final pedindo **254 horas de golpe ininterrupto**. Números escritos à mão não
erram pouco: erram por ordem de grandeza, porque cada um foi escrito supondo os
outros.

O sinal de que um número devia ser derivado: para escolhê-lo, você precisou olhar
outros três.

## Conexões
- Princípio: [[Guarde a intenção e o processo se reconstrói dela]]
- Irmã: [[Configuração vem do ambiente, não do código]]
- Visto em: [[naruto-idle]]
- Mapa: [[Backend]] · [[Base]]
