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

Sombra nas bordas que **some sozinha** quando o conteúdo está encostado na ponta.
Sem JavaScript, com quatro camadas de fundo e a mistura de
`background-attachment`:

```css
background-image:
  linear-gradient(to right, var(--surface), transparent),   /* cobre */
  linear-gradient(to left,  var(--surface), transparent),   /* cobre */
  linear-gradient(to right, rgb(0 0 0 / 35%), transparent), /* sombra */
  linear-gradient(to left,  rgb(0 0 0 / 35%), transparent); /* sombra */
background-position: left center, right center, left center, right center;
background-repeat: no-repeat;
background-size: 24px 100%, 24px 100%, 12px 100%, 12px 100%;
background-attachment: local, local, scroll, scroll;
```

O truque está na última linha. As camadas de cobertura são `local`, então rolam
**junto com o conteúdo** e saem de cena conforme ele se afasta da borda. As
camadas de sombra são `scroll`, então ficam **grudadas na moldura**. Encostado na
ponta, a cobertura fica exatamente em cima da sombra e a apaga; assim que o
conteúdo se afasta, a cobertura vai junto e a sombra aparece.

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
