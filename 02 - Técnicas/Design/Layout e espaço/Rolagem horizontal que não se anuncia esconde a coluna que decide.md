---
tags: [tipo/atomica, camada/padrao, design, dev/frontend, armadilha]
criado: 2026-09-17
---

# Rolagem horizontal que não se anuncia esconde a coluna que decide

> Tabela larga em tela estreita rola em vez de espremer coluna — essa parte está
> certa. O defeito é a rolagem não se anunciar: no celular, o que fica fora da
> tela são as ÚLTIMAS colunas, e num extrato as últimas colunas são o valor.

## O problema

`overflow-x: auto` com `min-width` na tabela é a receita conhecida, e ela evita o
mal maior (coluna espremida até o texto virar uma letra por linha). Só que o
recorte é mudo: no extrato do [[telebot]] em 390px de largura apareciam "quando"
e "o quê", e "valor" e "saldo depois" ficavam além da borda sem nenhuma pista.

Quem abre a tela não conclui "deve rolar para o lado". Conclui que o extrato não
mostra valores. E a conclusão é razoável — nada na tela contradiz.

O print de celular é o que denuncia; lendo o JSX, a tabela parece completa.

## A solução

Uma pista nas bordas que **some sozinha** quando o conteúdo está encostado na
ponta. Sem JavaScript, com quatro camadas de fundo e a mistura de
`background-attachment`:

```css
background-color: var(--surface);   /* obrigatório: ver abaixo */
background-image:
  linear-gradient(to right, var(--surface), rgb(0 0 0 / 0%)),        /* cobre */
  linear-gradient(to left,  var(--surface), rgb(0 0 0 / 0%)),        /* cobre */
  linear-gradient(to right, rgb(255 255 255 / 16%), rgb(255 255 255 / 0%)), /* pista */
  linear-gradient(to left,  rgb(255 255 255 / 16%), rgb(255 255 255 / 0%)); /* pista */
background-position: left center, right center, left center, right center;
background-repeat: no-repeat;
background-size: 28px 100%, 28px 100%, 14px 100%, 14px 100%;
background-attachment: local, local, scroll, scroll;
```

Dois detalhes que a receita corrente omite e que custaram uma rodada cada:

- **Num tema escuro a pista é CLARA, não uma sombra preta.** Toda receita na
  internet usa `rgba(0,0,0,.35)`, porque foi escrita para fundo branco. Sobre
  `#101017`, preto a 35% é literalmente invisível: o primeiro desenho saiu com
  sombra, e o print de celular mostrou o corte seco, sem aviso nenhum. O teste
  só serve olhando o print — lendo o CSS, as duas versões parecem iguais.
- **O container precisa de `background-color` opaco.** As camadas de cobertura
  pintam a cor da superfície para apagar a pista na ponta; se o container for
  transparente, essa cobertura vira uma faixa mais clara que o fundo real,
  permanentemente visível nas duas bordas.

O truque está na última linha. As camadas de cobertura são `local`, então rolam
**junto com o conteúdo** e saem de cena conforme ele se afasta da borda. As
camadas de pista são `scroll`, então ficam **grudadas na moldura**. Encostado na
ponta, a cobertura fica exatamente em cima da pista e a apaga; assim que o
conteúdo se afasta, a cobertura vai junto e a pista aparece.

A cobertura precisa ser da cor da superfície de baixo. Se a tabela mudar de
fundo, esse valor muda com ela — por isso é token, não hex.

## O que mais vale lembrar

- A pista resolve a descoberta, não a ergonomia. Quando a coluna escondida é a
  razão de a tela existir, vale perguntar se ali a tabela é a peça certa: uma
  lista de cartões empilhados responde melhor no celular, ao custo de uma segunda
  implementação para manter.
- **Célula numérica não quebra linha** (`whitespace-nowrap`). "R$ 1.297," numa
  linha e "86" na outra deixa de ser um valor e vira dois pedaços de texto. Vale
  para data também.
- A pista mora no PRIMITIVO da tabela, não na tela. Escrita na tela, ela nasce
  numa e falta nas outras seis.

## Conexões
- Princípio: [[Todo estado da tela tem visual]] · [[Catálogo de componentes é contrato vivo, não documentação]]
- Irmã: [[Sticky gruda no container que rola, não na janela]] · [[Faixa que sangra estoura pela barra de rolagem, e o corte é na raiz]] · [[Blocos de dado - card, KPI e gráfico]]
- Visto em: [[telebot]]
- Mapa: [[Design]]
