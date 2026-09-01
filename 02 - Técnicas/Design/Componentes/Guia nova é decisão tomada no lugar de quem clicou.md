---
tags: [tipo/atomica, camada/padrao, design, armadilha]
criado: 2026-09-01
---

# Guia nova é decisão tomada no lugar de quem clicou

> Abrir link em outra guia parece gentileza — "não tiro você de onde estava" — e é uma escolha feita por cima da pessoa. Ela só se paga quando a volta **não recupera o contexto**. Se existe saída nomeada no destino, ou se o botão de voltar devolve a tela exata, a guia nova não resolve nada e cobra tudo.

## O teste

Antes de escrever `target="_blank"`, responda: **o que a pessoa perde se voltar?**

- Perde uma fila que levou tempo para montar, um formulário meio preenchido, uma
  posição de revisão? Guia nova, e avise antes do clique.
- Volta para a mesma tela que já estava? Mesma guia. O botão de voltar devolve o
  estado exato — mais do que a guia nova entrega, porque a guia nova só preserva
  a página, e voltar preserva onde ela estava.

O caso legítimo é a **bancada**: o moderador que abre um item para revisar sem
perder a fila. O caso ilegítimo é a **travessia de leitura**: quem está lendo
algo e resolve espiar outra coisa.

## Os custos que ninguém soma

- **No celular é pior.** Trocar de aba é um gesto escondido atrás de um menu;
  voltar é um botão dedicado que todo mundo já sabe usar.
- **A pessoa não pediu.** Quem quer guia nova tem clique do meio, toque longo e
  menu de contexto — todos disponíveis, todos por escolha dela.
- **Abas órfãs se acumulam.** Cada travessia deixa uma aba que ninguém fecha.

## O aviso é o cheiro do problema

Guia nova exige avisar leitor de tela antes do acionamento, senão a pessoa é
movida sem perceber. Quando o componente precisa de um `sr-only` "(abre em nova
guia)" e de um ícone de seta só para se explicar, vale reler a decisão: **um
comportamento que precisa se desculpar costuma estar resolvendo um problema que
outra peça já resolveu**.

## A armadilha de raciocínio que veio junto

Foi assim que a decisão errada passou: a guia nova e a saída nomeada nasceram no
mesmo commit, e o argumento que justificava uma foi usado para justificar as
duas — "quem chega por link mandado em conversa não tem aba anterior". Verdade,
e irrelevante para a guia nova: **essa pessoa não passa pelo link que abre a
guia nova.** Duas decisões no mesmo commit precisam de dois argumentos, e cada
um tem de falar do caminho que a sua decisão afeta.

## Conexões
- Princípio: [[A casca se compartilha por público, não por marca]] — casca própria é que cria a necessidade de dizer onde é a porta; a guia nova é a tentativa preguiçosa de não precisar dizer
- Irmã: [[Ver o plano e mandar executar são duas ações]] — o mesmo respeito por quem decide
- Visto em: [[Privello]] — travessia do anúncio para o feed
- Mapa: [[Design]]
