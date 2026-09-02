---
tags: [tipo/atomica, camada/padrao, design, dev/frontend]
criado: 2026-09-02
---

# A lista do select nativo não aceita estilo, então ele não serve de dropdown padrão

> Fechado, o `<select>` veste a casca do sistema e engana. Aberto, quem desenha
> a lista é o navegador — outra fonte, outro realce, outro canto — ao lado de
> controles que são seus.

## O problema

A regra que parece sensata, e que eu tinha escrito no catálogo: lista curta e
fechada usa a **seleção nativa**, porque ela ganha em teclado, leitor de tela e
celular; o combo custom entra só quando a lista é longa o bastante para precisar
de busca.

O argumento é bom e a conclusão é errada, porque ele mede só o controle fechado.
A parte que não dá para vestir é justamente a que aparece no momento da decisão:
a lista. Num formulário de cadastro isso passa; numa **barra de ferramentas**,
onde o select está encostado num segmentado e num botão do sistema, ele denuncia
que veio de outro lugar.

## A solução

Inverter o padrão: **o combo do sistema é o dropdown padrão, inclusive para três
opções**. A seleção nativa fica para formulário de cadastro, onde o seletor do
sistema operacional é vantagem de verdade — no celular ele vira a roleta nativa,
e ninguém quer reimplementar isso.

A escada fica:

- duas a quatro opções que cabem visíveis → **segmentado**
- escolha única → **combo**
- escolha acumulável → **combo múltiplo**
- formulário de cadastro, mobile-first → **seleção nativa**

## O que mais vale lembrar

- **Trocar o select por combo cobra o alinhamento junto.** O combo veste a casca
  de campo, que reserva a linha do rótulo; ao lado de um botão que não tem
  rótulo, ou os dois ganham a linha ou nenhum ganha. E a fila de ações precisa
  alinhar pelo topo: `flex` sem `items-*` alinha por `stretch`, e o controle com
  rótulo estica.
- **Um combo custom só é aceitável se resolver o que o nativo dava de graça**:
  foco, teclado, Escape, clique fora e — se ele vive dentro de painel com vidro
  — [[Vidro cria contexto de empilhamento, e nenhum z-index atravessa isso]].

## Conexões
- Princípio: [[A variante de um controle muda a intenção, não o tamanho]]
- Irmã: [[Primitiva de botão fecha o tamanho e abre só a variante]] ·
  [[Vidro cria contexto de empilhamento, e nenhum z-index atravessa isso]]
- Visto em: [[Navetech Hub]]
- Mapa: [[Design]]
