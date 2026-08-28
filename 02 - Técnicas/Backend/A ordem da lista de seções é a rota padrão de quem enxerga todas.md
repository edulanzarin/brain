---
tags: [tipo/atomica, camada/padrao, dev/backend, armadilha]
criado: 2026-08-28
---

# A ordem da lista de seções é a rota padrão de quem enxerga todas

> Quando a home de um módulo redireciona para a **primeira seção visível**, a
> ordem da lista deixa de ser estética e vira **regra de roteamento** — para quem
> acessa tudo. Num par por posse (colaborador / gestão), a de gestão vem primeiro,
> senão o admin cai na tela do colaborador.

## A armadilha

O par de [[Posse numa permissão binária é duas seções e recorte por linha]]
funciona porque cada cargo recebe UMA das duas seções: quem tem só a do dono cai
nela, quem tem só a de gestão cai na outra. A conta fecha para todo mundo — menos
para o **admin**, que não recebe seção nenhuma: ele passa no gate por ser admin e
portanto enxerga **as duas**. Aí quem decide onde ele aterrissa é o desempate, e o
desempate é a ordem da lista.

Com a seção do dono escrita primeiro, o admin abre o módulo e vê **os próprios
números** — que costumam ser zero, porque quem administra o sistema não é quem
opera nele. A leitura errada é imediata: "o módulo está vazio". O dado do time
está a um clique na sidebar, mas ninguém procura o que acha que não existe.

## A regra

Onde a seção é **home de módulo** e existe par por posse, a de gestão vem antes
na lista. Vale escrever isso no comentário ao lado da lista: a ordem parece
cosmética e alguém vai reordenar por gosto, restaurando o bug.

## Por que passa despercebido

O caso quebrado é o de quem tem **as duas** seções — e esse é sempre o admin, que
é justamente o perfil que ninguém testa, porque "admin vê tudo" soa como ausência
de regra. É o contrário: ver tudo é o único caso em que o desempate roda.

## Conexões
- Princípio: [[A home de um módulo é o resumo que carrega sozinho; automação não abre sozinha]]
- Irmã: [[Posse numa permissão binária é duas seções e recorte por linha]] · [[Permissão composta por papéis somados, não exceção por usuário]]
- Visto em: [[Navetech Hub]]
- Mapa: [[Backend]]
