---
tags: [tipo/atomica, camada/principio, design]
criado: 2026-09-01
---

# A casca se compartilha por público, não por marca

> Duas coisas da mesma empresa, no mesmo domínio, com o mesmo logo, tendem a
> nascer dentro da mesma casca — o mesmo topo, a mesma navegação, o mesmo
> rodapé. Parece economia e coerência. Mas a casca não serve à marca: serve a
> quem está usando, e o que decide se ela é uma só é se as duas coisas têm o
> mesmo público e o mesmo ritmo de volta.

## A regra

Antes de pendurar uma área nova na casca existente, duas perguntas:

- **É a mesma pessoa?** Não o mesmo login — a mesma pessoa na mesma intenção.
- **Ela volta com a mesma frequência?** Uma tela que se lê uma vez para decidir
  e uma que se reabre toda semana são dois ritmos, e a navegação de uma atrapalha
  a outra.

Duas respostas "sim" é uma casca. Um "não" já é casca própria, mesmo que o
código, o tema e o domínio continuem os mesmos.

## Por que

Porque a casca não é decoração: ela é o que a pessoa atravessa em TODA visita.
No [[Privello]] o classificado e o feed por assinatura moravam na mesma página.
A barra da vitrine oferece cidade, filtro e "anunciar" — o vocabulário de quem
está procurando uma acompanhante. Quem assina o feed de alguém não vem fazer
isso: vem ver o que ela postou hoje, e chegava passando por uma tela de busca
que não veio usar.

E o efeito no conteúdo é pior que o na navegação. Empilhados na mesma página, o
feed vira rodapé do anúncio — e quem assina passaria a entrar pelo classificado,
que é justamente a parte que ela já leu e não precisa reler. A ordem da página
ensina qual dos dois é o principal, e ela ensinou errado.

O mesmo produto já tinha o caso fácil resolvido sem ninguém discutir: a moderação
tem casca própria porque "admin é obviamente outra coisa". O que não é óbvio é
que dois produtos voltados ao PÚBLICO podem estar tão distantes quanto um deles
está do admin.

## O contrário também é regra

Quando o público e o ritmo são os mesmos, casca separada é o defeito: obriga a
pessoa a reaprender onde ficam as coisas a cada troca. É o caso da suíte de
módulos operada pela mesma pessoa o dia inteiro, que pede uma casca com dois
níveis dentro dela —
[[Navegação de dois níveis - trilho de produto e sidebar de contexto]].

## Na prática

- **A ponte entre as duas é um convite, não um bloco.** No lugar do conteúdo
  embutido, um cartão que diz o que existe do outro lado e leva. Ele cabe na
  página do outro produto sem competir com ela.
- **O caminho de volta é explícito nos dois sentidos**, senão a pessoa que
  atravessou fica presa.
- **Casca própria não é tema próprio.** Os tokens, os componentes e a tipografia
  continuam os mesmos: o que muda é a navegação e o que ela oferece.

## O preço de sair: a volta deixa de ser de graça

Casca compartilhada dá a volta sem ninguém pensar nela — a navegação de sempre
está lá, e quem entrou numa seção sai por onde entrou. Casca própria tira isso, e
o buraco não aparece para quem construiu: quem constrói chega pela URL e sai
fechando a aba.

No [[Privello]] a saída do feed existia, e mesmo assim não existia. Era uma lupa
rotulada "Procurar", e no celular sobrava só a lupa. Ninguém lê aquilo como
saída: lê como busca. A pessoa concluía, com razão, que tinha ficado presa.

Três coisas que a casca própria passa a dever:

- **A saída é nomeada pelo DESTINO**, não pelo gesto. "Voltar" sozinho é o botão
  do navegador; quem procura a saída quer saber para onde ela vai. Se o espaço
  aperta no celular, encurte o verbo e mantenha o destino.
- **Ela não some em nenhuma largura.** Rótulo escondido atrás de `sm:` é a saída
  desaparecendo exatamente na tela onde a barra do navegador também some.
- **Guia nova ajuda e não resolve.** Abrir a travessia numa aba nova deixa o
  produto anterior de pé e transforma "voltar" em trocar de aba — mas só para
  quem veio de lá. Quem chega por um link mandado em conversa não tem aba
  anterior nenhuma, e é justamente essa pessoa que a casca precisa atender.

## Conexões
- Irmã: [[O que responde pergunta rara não ocupa a rolagem de todo mundo]] — lá
  o corte é dentro da página; aqui é entre produtos.
- Irmã: [[Estética é por projeto, princípio de design é que se reusa]]
- Técnica que aplica: [[Navegação de dois níveis - trilho de produto e sidebar de contexto]]
- Visto em: [[Privello]]
- Mapa: [[Base]]
