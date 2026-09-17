---
tags: [tipo/atomica, camada/principio, armadilha]
criado: 2026-09-17
---

# Substituição global se decide membro a membro, não pelo nome da família

> Nome comum não é papel comum. Antes de trocar tudo que casa com um nome, liste
> os membros e classifique cada um: quem carrega **forma** se substitui, quem
> carrega **significado** fica de fora.

## A regra

Toda substituição ampla — de fonte, de token, de string, de variável, de rota —
começa escolhendo um nome e terminando em "tudo que se parece com isso vira
aquilo". O passo que quase sempre falta é o do meio: **enumerar o que o nome
pega** e perguntar, de cada item, se ele é mesmo a mesma coisa.

A resposta costuma ser não para uma minoria. E é justamente essa minoria que
quebra, porque ela não parece diferente — ela só *é*.

## Por que

Substituir supõe intercambialidade: se A vira B, é porque A e B fazem o mesmo
trabalho. Isso vale para o que é **forma**, e falha para o que é **significado**.
Duas letras de alfabetos diferentes desenham a mesma coisa; um ícone e uma letra,
não. O nome da família não sabe dessa diferença — ele agrupa por origem, por
fabricante, por prefixo, e origem comum não implica função comum.

Por isso o sintoma é sempre assimétrico e enganoso: 95% da troca funciona de
primeira, e o que sobra some por completo em vez de ficar feio. Coisa que some
não pede revisão — some.

## Os dois casos que ensinaram

**Um hex espalhado não diz sua intenção.** Com `#2a78d6` escrito em dois arquivos,
ninguém sabe se é a mesma decisão ou coincidência — e trocar os dois por ser o
mesmo valor é apostar que era a mesma. É o argumento de
[[Token semântico em vez de valor literal]]: o nome do papel existe pra tornar
essa pergunta respondível antes da troca, não depois.

**A família `Segoe UI` do Windows não é uma fonte.** São dezesseis entradas de
registro, e quatro delas não desenham texto: `Segoe UI Emoji`, `Segoe UI Symbol`,
`Segoe UI Historic` e, fora do prefixo, `Segoe Fluent Icons`. Redirecionar "tudo
que começa com Segoe" apaga seta, wi-fi, bateria e o botão de fechar da janela.
Uma quinta, `Segoe UI Variable`, carrega forma mas é usada por um subsistema
diferente — mesma família no nome, outro caminho de código. Ver
[[Trocar a fonte do Windows é redirecionar a família Segoe; as de ícone ficam de fora]].

## Na prática

- **Enumere antes de escrever a substituição.** `reg query`, `grep -l`, `git grep
  -c` — o que der a lista completa. A lista quase sempre é maior que a ideia que
  se tinha dela, e é ela, não o nome, que define o alcance.
- **Classifique por função, não por nome.** A pergunta de cada membro é "isso
  carrega forma ou significado?". Ícone, emoji, símbolo, código de status, chave
  de tradução e identificador carregam significado.
- **O membro exemplar não representa a família.** Testar a troca no caso central
  confirma os 95% que iam funcionar de qualquer jeito. O teste que vale é no
  membro esquisito.
- **Quem é usado por outro subsistema fica fora até segunda ordem.** Mesmo
  carregando forma, ele responde a outro dono, e o ganho de incluir raramente
  paga o risco.

## Conexões
- Irmã: [[Token semântico em vez de valor literal]]
- Padrão que aplica: [[Trocar a fonte do Windows é redirecionar a família Segoe; as de ícone ficam de fora]]
- Mapa: [[Base]]
