---
tags: [tipo/atomica, camada/principio, design, armadilha]
criado: 2026-09-02
---

# Propriedade escolhida pelo visual redefine a estrutura por baixo

> Um efeito pedido por causa da aparência quase sempre vem com uma mudança de
> regra estrutural embutida — e o defeito aparece longe, em outro componente,
> sem erro nenhum para procurar.

## A ideia

Existe uma família de propriedades que muda o VISUAL e, de quebra, redefine o
sistema de referência de tudo que está dentro dela: quem é o ancestral de
rolagem, quem é o bloco que contém o posicionado, quem é a raiz do
empilhamento. Elas são pedidas por um motivo estético — arredondar, desfocar,
animar, cortar — e cobram o preço num lugar que ninguém liga ao pedido.

O sintoma é sempre o mesmo, e é o que torna a armadilha cara: **não há erro**. O
elemento existe, tem tamanho, tem cor, e simplesmente aparece no lugar errado,
atrás de outra coisa, ou grudado onde não deveria. Quem procura, procura no
componente que quebrou — que está certo.

## Por que a solução óbvia não funciona

A reação natural é aumentar o número: um `z-index` maior, um `top` diferente.
Isso não funciona porque o número foi reinterpretado. Ele passou a valer DENTRO
de uma fronteira nova, e nenhum valor vence uma fronteira em que não se está.

A saída é sempre uma das duas: **tirar a peça de dentro da fronteira** (é o que
o portal faz) ou **assumir a fronteira e medir a partir dela**.

## O reflexo que evita

Ao adicionar uma propriedade pelo visual, perguntar o que ela redefine para
quem está dentro. E, do outro lado, quando algo flutuante aparece no lugar
errado, procurar o ancestral antes de procurar o próprio componente: a causa
quase nunca está em quem exibe o defeito.

## Conexões
- Depende de: [[Hierarquia por superfície, não por borda]]
- Exemplo: [[Vidro cria contexto de empilhamento, e nenhum z-index atravessa isso]] ·
  [[Sticky gruda no container que rola, não na janela]]
- Mapa: [[Design]] · [[Base]]
