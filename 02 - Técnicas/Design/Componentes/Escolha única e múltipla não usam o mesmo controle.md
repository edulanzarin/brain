---
tags: [tipo/atomica, camada/padrao, design]
criado: 2026-08-30
---

# Escolha única e múltipla não usam o mesmo controle

> Uma tela de quarenta caixas deixava marcar cabelo loiro E ruivo, olho preto E
> verde. Não faltava validação: faltava o controle dizer que ali só cabe um.

## O problema

Etiqueta, característica, filtro — tudo vira "uma lista de coisas para marcar",
e a caixa de seleção é o controle óbvio. Só que a lista mistura duas naturezas
diferentes:

- **Acumulável**: tatuagem, piercing, silicone. Marcar duas é verdade.
- **Eixo**: cabelo, olhos, biotipo. Marcar duas é contradição.

Com caixa nos dois casos, nada na tela impede a contradição, e ela só aparece no
perfil publicado. Aviso não resolve: quem estava marcando não estava lendo.

## A solução

**O controle carrega a regra.** Eixo vira fila de chips, que a pessoa já lê a
vida inteira como "escolha um"; acumulável continua caixa, que se lê como
"marque quantas quiser". Entrar num eixo tira quem estava lá, sem pedir para
desmarcar antes — o sistema sabe fazer isso, e no meio do pedido o campo fica
vazio. Chip marcado desmarca no segundo clique: eixo não é obrigatório, e sem
essa saída quem clicou por engano fica preso.

**A trava mora no dado, não numa lista de grupos no código.** Uma coluna
`escolhaUnica` na etiqueta, com o nome do eixo: quem divide o valor não coexiste.
Acrescentar "cabelo colorido" ou um eixo novo vira uma linha no cadastro, não um
deploy. A tela lê a coluna para decidir chip ou caixa, então o desenho acompanha
o dado sozinho.

**E o servidor confirma.** A tela impede pelo desenho, mas quem manda o
formulário na mão passa por cima dela: na gravação, o excedente do eixo cai fora
em vez de derrubar o envio inteiro.

## O sinal de que o eixo existe

Se dois itens da mesma lista não podem ser verdade juntos, ali não é lista, é
**pergunta com resposta única** — e o rótulo do grupo costuma denunciar: "Cabelo"
é eixo, "Aparência" é acúmulo. Grupo cujo nome é um substantivo do corpo ou uma
categoria fechada quase sempre é eixo.

## Conexões
- Princípio: [[A definição em dado dirige o comportamento, não um caso no código]] —
  a exclusividade é dado do cadastro, e a tela e a gravação apenas obedecem.
- Irmã: [[Escala fechada em vez de valor solto]] ·
  [[Sinal booleano da fonte não ocupa o lugar de uma escala]] ·
  [[Palavra da interface é lida com o dicionário do usuário, não com o seu]]
- Visto em: [[Privello]]
- Mapa: [[Design]]
