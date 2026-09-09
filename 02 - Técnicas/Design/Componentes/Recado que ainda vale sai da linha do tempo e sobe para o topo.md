---
tags: [tipo/atomica, camada/padrao, design]
criado: 2026-09-09
---

# Recado que ainda vale sai da linha do tempo e sobe para o topo

> A linha do tempo ordena por quando aconteceu. O recado que a equipe precisa
> ver ordena por "ainda vale" — e essas duas ordens não são a mesma.

## O problema

Um pedido que chega sempre igual: "queria anotar uma coisa e **fixar** em algum
lugar". A tentação é responder só com o campo de anotação, porque o histórico
do item já existe e aceita texto. Só que a linha do tempo é cronológica e
completa por natureza: dois cadastros e três edições depois, o "alvará
solicitado dia 3 para o cliente" — que continua valendo — está abaixo da dobra,
entre eventos automáticos que ninguém foi ler.

Guardar o recado numa **aba própria** também não resolve. Quem abriu a ficha
para conferir um certificado não clica na aba de anotações; ele passa direto
por cima do combinado e liga para o cliente perguntando o que já foi feito.

## A solução

Um campo (`pinned`) no registro e **dois lugares para o mesmo dado**:

- a linha do tempo continua cronológica e completa, e ali o item fixado só ganha
  uma marca discreta;
- o fixado é promovido para **fora da navegação** — acima das abas, colado no
  cabeçalho, visível em qualquer aba que a pessoa escolha.

Sem nada fixado, o bloco não existe. Moldura vazia com "nenhum recado fixado"
ocupa a mesma altura e não informa nada.

## O que mais vale lembrar

- **Fixar nasce junto com o texto.** Um botão "fixar" que só aparece depois de
  salvar transforma um ato em dois, e o segundo é o que se esquece. O campo de
  escrita já traz o alfinete.
- **Soltar mora onde o fixado aparece.** Se para desafixar é preciso voltar à
  aba de origem, o topo enche e nunca esvazia.
- **Só recado se fixa.** Cadastro e alteração são registro automático — não têm
  a quem se dirigir, e deixar fixá-los devolve o problema pro topo da tela.
- **A cor do fixado não pode ser a cor de estado da tela.** Âmbar de post-it é o
  reflexo, mas numa tela onde âmbar já significa "vence logo", o recado passa a
  ser lido como alerta. Usar o mesmo acento das anotações mantém o alarme livre.
- **Fixar é permissão de quem acompanha, não de quem edita o cadastro.** Amarrar
  o alfinete ao nível de edição entrega o quadro de avisos justamente a quem
  menos o usa.

## Conexões
- Princípio: [[Ordene pela grandeza que decide, não pela que impressiona]] — a
  linha do tempo ordena por recência, mas quem lê decide por "isso ainda vale".
- Irmã: [[O que responde pergunta rara não ocupa a rolagem de todo mundo]] (o
  caso inverso: o que quase ninguém pergunta é que sai do caminho) ·
  [[Cor de identidade não pode ocupar o lugar da cor de estado]]
- Visto em: [[Cofre Digital]]
- Mapa: [[Design]]
