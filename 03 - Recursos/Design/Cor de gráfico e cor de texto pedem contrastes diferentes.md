---
tags: [tipo/atomica, design, dev/frontend, acessibilidade, conceito, projeto/navedesk]
criado: 2026-07-19
---

# Cor de gráfico e cor de texto pedem contrastes diferentes

> A mesma cor pode estar aprovada como fatia de donut e reprovada como texto de
> badge. São alvos WCAG diferentes — então são **tokens diferentes**.

## O erro

Parece elegante fazer o badge de status "andamento" apontar para `--esp-3`, a
âmbar da paleta de gráficos: uma cor só, um lugar só pra mudar. Foi o que fiz no
[[NaveDesk]] — e a auditoria de contraste reprovou.

`#c28831` sobre branco dá **3,06:1**. Como série de gráfico está correto (o alvo
de elemento não-textual é 3:1). Como **texto** de badge está errado: texto normal
precisa de **4,5:1**.

A confusão é natural porque o token parece "a cor do âmbar". Mas cor não tem um
requisito — o **papel** dela tem. Preencher área e escrever palavra são papéis
diferentes.

## A correção

Uma camada de matizes em "grau texto", separada das séries:

```css
/* grau texto — badge ESCREVE com a cor: 4.5:1 */
--amber-ink: #9b6d27;
--violet-ink: #955fb6;

/* série de gráfico — só preenche área: 3:1 */
--esp-3: #c28831;
--esp-5: #9a62bc;
```

E o domínio aponta para o grau texto:

```css
--status-andamento: var(--amber-ink);
--status-aguardando: var(--violet-ink);
```

Num tema escuro os dois costumam coincidir (as categóricas já são claras o
bastante para passar de 4,5:1 sobre a superfície). Mesmo assim vale manter os
dois tokens: o componente não deveria precisar saber que naquele tema eles são
iguais.

## Por que descobri isso

Só apareceu porque a auditoria checava **cada token contra a superfície onde ele
realmente aparece**, com o alvo do papel dele — não um contraste genérico. Ela
pegou 5 falhas, todas no tema claro (o escuro passou limpo), e esta era a única
que não era "escurece um pouco": era erro de modelagem.

Vale como padrão: auditoria de contraste em CI, não inspeção visual. Ver
[[Sistema de cores e tema do dashboard]].

## Conexões
- Faz parte de: [[Design]]
- Origem: [[NaveDesk]]
- Ver também: [[Paleta categórica com estética por restrição]],
  [[Validar paleta de gráficos antes de escolher cores]]
