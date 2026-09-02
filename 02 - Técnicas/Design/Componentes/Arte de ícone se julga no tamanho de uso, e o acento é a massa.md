---
tags: [tipo/atomica, camada/padrao, design]
criado: 2026-08-22
---

# Arte de ícone se julga no tamanho de uso, e o acento é a massa

> Ícone desenhado grande e aprovado grande falha calado quando entra na tela. O que
> sobrevive à redução não é o detalhe nem a metáfora: é a **silhueta** e a **área de
> cor**.

## A regra

**Julgue em folha de contato, no fundo real, ao lado dos que já existem.** Ícone isolado
e ampliado sempre parece bom. Lado a lado é que aparece o que destoa — e o que destoa
quase sempre é *massa*: arte que não preenche o quadro sai magra ao lado de arte que
preenche, mesmo com o mesmo grid.

**O acento tem que ser a área dominante, não um detalhe.** Chassi escuro com um filete
colorido lê bem a 140px e vira mancha preta a 44. Nas artes que sobrevivem, a cor é o
corpo: a gema é toda azul, a poção é meio frasco de vermelho, a bandeira é rosa inteira.
Nas que falham, a cor é uma tira.

**E a massa clara tem que cair no ASSUNTO, não na moldura.** Ter área de cor suficiente
não basta: importa em que parte da figura ela está. O ícone do Stadium do [[piwdex2]]
nasceu com a arquibancada clara e o campo escuro — área de cor de sobra, e no tamanho de
uso a peça era uma rosca com dois palitos atrás, que é a figura mais próxima que o olho
tinha pra fechar. Invertido (gramado aceso, arquibancada escura, que é como estádio
aparece em toda foto noturna), passou a ler de primeira. Nenhum detalhe a mais tinha
resolvido isso nas passadas anteriores. Pergunte qual é o assunto e entregue a maior
mancha clara pra ele; o resto do quadro é cerco.

**Separe por silhueta antes de separar por cor.** Num conjunto que aparece junto —
categorias, estados —, a forma é o que distingue pra quem não diferencia matiz, e é o
que sobra quando o ícone encolhe. Bolsa, gema, frasco, ampola, bandeira, disco, carta,
engradado: oito contornos que não viram a mesma mancha. Cor entra depois, como reforço.

## Escala é decisão, não preferência

Arte cheia (pixel, glifo sólido) e ícone de traço fino **não vivem na mesma escala**. O
glifo cheio preenche a caixa; o traço desenha só a borda e some no mesmo tamanho. Um
sistema saudável usa os dois e diz onde cada um vale:

- **figura**, de 24px pra cima — cabeçalho, estado vazio, card, tela de erro: arte;
- **chrome miúdo**, 14 a 16px — seta, fechar, filtro: ícone de traço.

Trocar um pelo outro sem recalibrar o tamanho é o erro que faz alguém concluir que
"pixel art é feia", quando o problema era a escala. Vale registrar essa fronteira no
código, junto do gerador — senão a próxima leva repete o caminho.

## Na prática

Se a arte é gerada por script, o **acabamento** (contorno, halo) mora no motor, não no
módulo de desenho: é o que faz uma peça nova pertencer ao conjunto. Quem desenha escolhe
a forma; o acabamento vem de graça e igual pra todos.

E toda arte tem reserva: peça que não carregou não pode virar caixa vazia — ainda mais
numa tela de erro, onde seria a segunda falha em cima da primeira.


## Uma identidade, uma marca — e o detalhe que se esconde sozinho

O [[Navetech Hub]] chegou a ter DUAS marcas para o mesmo módulo: um cubo
isométrico no launcher e uma figura plana (recibo, livro, crachá) na barra
lateral e nos chips. Parecia divisão de trabalho — "a peça grande e a pequena" —
e era problema: quem lê precisa aprender as duas, e a que aparece menos envelhece
sem ninguém notar.

A folha de contato desfez o falso dilema. Renderizado de 16 a 64px, o cubo lia
limpo em TODOS os tamanhos. Quem não sobrevivia era a **sigla** na face dele, que
abaixo de ~32px virava uma mancha suja — e mancha suja não é "detalhe pequeno",
é o que faz a peça inteira parecer defeituosa.

A saída não é ter duas artes: é a arte **esconder o próprio detalhe** quando ele
não cabe.

```tsx
const mostrarSigla = tamanho >= 32;   // dentro do desenho, não em quem chama
```

**O limiar mora no DESENHO.** Exposto como prop, a primeira tela nova esqueceria
dele — e ninguém repara numa sigla borrada num canto. Dentro da peça, ela é
correta por construção em todo lugar onde for usada.

O teste que revela isso é barato e é sempre o mesmo: rendere a peça na escada
inteira de tamanhos de uso, lado a lado. Não dá para deduzir em qual tamanho um
detalhe morre.

## Conexões
- Princípio: [[Estética é por projeto, princípio de design é que se reusa]]
- Irmã: [[Trocar arte por ícone de linha exige recalibrar tamanho, não só trocar o componente]] ·
  [[Glifo miúdo é lido como o símbolo mais próximo que a pessoa já conhece]]
- Visto em: [[piwdex2]] · [[Navetech Hub]]
- Mapa: [[Design]]
