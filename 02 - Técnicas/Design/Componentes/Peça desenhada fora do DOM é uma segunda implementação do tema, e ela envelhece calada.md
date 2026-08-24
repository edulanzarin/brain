---
tags: [tipo/atomica, camada/padrao, design, dev/frontend, armadilha]
criado: 2026-08-24
---

# Peça desenhada fora do DOM é uma segunda implementação do tema, e ela envelhece calada

> Cartão em canvas, imagem de OpenGraph, PDF, e-mail: tudo que é desenhado fora do
> DOM não lê token, não herda classe e não quebra quando o tema muda. **Fica
> errado em silêncio** — e como ninguém abre essas peças no dia a dia, elas
> continuam publicando o visual de duas viradas atrás.

## O sintoma

O site inteiro migrou de um dialeto pixel/neon de canto reto para superfície com
raio e elevação. Meses depois, o cartão que a ferramenta gera pra postar no grupo
ainda tinha grade de 20px no fundo, moldura dupla em fio neon e canto reto em
tudo. Ele não deu erro nem um dia: `#0b0d12` continua sendo uma cor válida.

Pior que o estilo: os **medidores em blocos**. A interface tinha trocado bloco por
linha contínua de propósito — bloco quantiza valor contínuo, e com 14 deles cada
um vale 7%, então 17,6 e 19,9 acendem o mesmo tanto e a barra afirma que são
iguais. A troca corrigiu a tela e deixou intacta a peça que de fato circula por
fora, que era onde a afirmação errada ia mais longe.

## A regra

**Toda peça fora do DOM entra na lista da virada de estilo, junto com os
componentes.** E o teste é grosseiro e funciona: gerar a peça e abrir ao lado da
tela. Se dá pra dizer qual das duas é mais velha, ela ficou pra trás.

A paleta vira uma tabela de hex declarada num lugar só do arquivo, com o token de
origem escrito ao lado de cada linha:

```ts
const COR = {
  fundo:  "#100e0c",   // --color-bg
  painel: "#1f1c18",   // --color-surface
  texto:  "#f0ecea",   // --color-text
}
```

Não elimina a cópia — canvas não lê variável CSS —, mas transforma "achar todo hex
espalhado no desenho" em "trocar seis linhas", e deixa o de-para auditável.

## O que mais vale lembrar (canvas)

**Gradiente radial com a mesma cor nas duas paradas é um disco**, não um brilho. O
halo tem de morrer em alfa zero pra ler como luz; com borda dura ele lê como
adesivo colado.

**A parada final é `rgba(mesma cor, 0)`, nunca a palavra `transparent`.** Parte
dos motores interpola `transparent` por preto transparente, e a borda do halo sai
suja.

**Posicione a partir da borda, não por deslocamento cravado.** Um `x = base +
medida(texto) + 200` só fica certo pra um comprimento de texto: "7%" e "100%"
jogam o vizinho pra lugares diferentes, e num deles ele encosta na margem. Meça e
ancore na direita da coluna.

**Cor em `fillStyle` depende do parser CSS do navegador.** Funções como
`color-mix()` podem simplesmente não pintar, sem exceção nenhuma. Para
translucidez, `globalAlpha` sempre funciona.

## Conexões
- Princípio: [[Token semântico em vez de valor literal]]
- Irmã: [[A mesma grandeza usa a mesma escada nas duas telas]] ·
  [[Blocos de dado - card, KPI e gráfico]] ·
  [[Sistema de cores e tema do dashboard]]
- Visto em: [[piwdex2]]
- Mapa: [[Design]]
