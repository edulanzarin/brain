---
tags: [tipo/atomica, camada/principio, seguranca]
criado: 2026-08-24
---

# Anonimato se perde na saída, não só na entrada

> Não guardar identidade é a metade fácil da promessa. A outra metade é o que o sistema MOSTRA: agregado com recorte fino reconstrói a pessoa a partir de dado que, linha a linha, não tem nome nenhum.

## A regra

Quando um canal promete anonimato, a promessa tem dois lados:

- **Entrada** — nenhuma coluna de identidade, nenhum vínculo com pessoa, nenhum IP "só para auditoria".
- **Saída** — nenhum caminho de leitura que estreite o conjunto até uma pessoa. Filtro combinado é esse caminho: "Contábil" cruzado com "menos de 6 meses de casa" costuma dar uma pessoa só, e o comentário dela aparece na tela assinado por eliminação.

A saída se protege com um piso: recorte com menos de N respostas não é exibido — e o mesmo piso vale para exportação, senão o botão de baixar é a porta dos fundos.

## Por que

O custo não é técnico, é a confiança: quem respondeu acreditando no anonimato não distingue "o banco não guarda quem" de "o painel não chega em quem". Basta a primeira pessoa ser identificada uma vez para a próxima rodada vir com respostas mornas — e pesquisa com resposta morna é pior que pesquisa nenhuma, porque parece dado.

Apareceu duas vezes, no mesmo sistema e por caminhos diferentes: no canal de denúncia (eNPS por setor num time de dois) e na pesquisa de clima (recorte por setor e tempo de casa). Em nenhuma das duas o vazamento estava no armazenamento — estava no painel que a gestão pediu, e com razão.

## Na prática

- Piso só com recorte ativo: o total geral não identifica ninguém, mesmo com poucas respostas.
- Trava, não recomendação: "não filtre demais" deixa o invariante na mão de quem está com pressa ([[Um invariante se garante na estrutura, não no processo]]).
- Vale para qualquer agregado sobre gente: ranking de equipe pequena, média salarial por cargo com um ocupante, comentário aberto num recorte estreito.
- Erro de leitura também vaza: mensagens diferentes para "recorte vazio" e "recorte suprimido" revelam se existe alguém ali.

## Contar sem identificar

Um canal aberto de propósito — denúncia sem login, formulário público, voto — não
pode exigir cadastro sem deixar de ser o que é. Mas sem nenhuma chave, nada
distingue vinte avisos de vinte pessoas de vinte avisos da mesma, e a fila de
decisão passa a ordenar por quem insistiu mais.

A saída é uma chave que serve para **contar** e não para **descobrir**: o resumo
do IP com o segredo da casa. Ela responde "já vi esta origem" e não responde
"quem é". Guardar o endereço cru seria coletar dado pessoal para resolver um
problema de contagem — e o dado coletado é o que vaza, não o que se calculou.

Duas ressalvas que a implementação precisa carregar:

- **Sem chave, sem limite.** Quando o cabeçalho não vem, contar agruparia todo
  mundo junto e o teto passaria a recusar a denúncia de estranhos entre si.
- **Quem tem conta não passa pelo limite.** Ali existe identidade, e quem modera
  julga a pessoa em vez do volume.

E os limites medem coisas diferentes: um por ALVO (o segundo aviso não acrescenta
nada à mesma decisão) e um por JANELA (o que segura quem sai avisando sobre a
cidade inteira). Um só dos dois deixa passar metade do problema.

## Conexões
- Depende de: [[Um invariante se garante na estrutura, não no processo]]
- Técnica que aplica: [[Recorte pequeno em pesquisa anônima identifica, então o painel se recusa a mostrar]]
- Técnica que aplica: [[Canal anônimo não guarda quem, e o retorno é um segredo do denunciante]]
- Mapa: [[Base]]
