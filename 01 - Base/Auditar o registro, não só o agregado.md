---
tags: [tipo/atomica, camada/base, dados]
criado: 2026-07-31
---

# Auditar o registro, não só o agregado

> Toda agregação — soma, saldo, rollup, balde — **descarta informação de propósito**, e parte do que ela descarta é justamente a anomalia. Um erro que se **compensa** some no total (o item no lugar errado deixa uma conta a mais e outra a menos, a soma fecha); um registro **mal-endereçado** some no rollup (lançamento numa conta agrupadora não desce pra folha nenhuma). Quem só olha o número agregado vê "tudo certo". Para achar esse tipo de defeito, a lente tem que descer ao **registro individual** — varrer a linha, não conferir o total.

## Por que importa

"O total bate?" e "cada registro está no lugar certo?" **parecem a mesma pergunta e não são**. A primeira é barata e tranquilizadora; a segunda é a que pega fraude, erro de digitação e integração quebrada. Um sistema de conferência precisa das duas lentes, e a agregada nunca substitui a fina — ela ativamente esconde o que a fina existe para achar.

Não é só contabilidade: média que engole o outlier, dashboard que soma o que devia listar, teste que confere a contagem de linhas mas não o conteúdo. Onde há agregação, pergunte o que ela apagou — e ofereça a lente que desce ao grão.

## Conexões
- Irmã: [[Espelhar por balde esconde item no lugar errado]] (o mesmo cegamento no caso da reconciliação por espelho) · [[Balancete é movimento do período, saldo é consequência]] (o saldo esconde o fluxo do mês)
- Visto em: [[Navetech Hub]] — Auditoria de Lançamentos (a varredura do `lctoctb` pega o lançamento em conta sintética que some do rollup do balancete); Balancete Fiscal (conta errada que se anula no total agregado)
- Mapa: [[Base]] · [[Dados]]
