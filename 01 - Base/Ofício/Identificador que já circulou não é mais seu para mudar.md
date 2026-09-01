---
tags: [tipo/atomica, camada/principio, dev/backend]
criado: 2026-08-30
---

# Identificador que já circulou não é mais seu para mudar

> No instante em que um identificador sai do sistema — vira link numa conversa,
> número num e-mail, endereço colado numa bio — ele deixa de ser um detalhe de
> implementação e passa a ser promessa. Quem o guardou não fica sabendo quando
> você muda de ideia.

## A regra

Antes de criar um identificador que o mundo vai ver, responda duas perguntas:

1. **De onde ele nasce?** Se nasce de um campo que a pessoa edita — nome,
   título, categoria —, então cada edição publica um endereço novo e mata o
   anterior sem avisar ninguém. Identificador público nasce de escolha
   deliberada, ou de nada.
2. **Quem paga a troca?** Não é quem troca. É quem tinha o link antigo, e essa
   pessoa nem descobre por que parou de funcionar: ela vê "não encontrado" e
   conclui que o perfil, o pedido ou o formulário deixou de existir.

Daí saem as três consequências práticas:

- **Separe o que se lê do que se endereça.** O nome muda quando quiser; o
  endereço tem outro prazo. São duas coisas, e tratar as duas como uma é o que
  faz o endereço herdar a volatilidade do nome.
- **Se a troca precisa existir, ela tem custo e o custo aparece.** Trava de
  tempo, aviso do que vai quebrar, ou redirecionamento do antigo para o novo.
  Troca livre e silenciosa é a única forma que não se pode oferecer.
- **Mudança de casa não é mudança de identidade.** Reorganizar tabela é assunto
  interno; o identificador que o mundo segura atravessa a migration junto.

## Por que

O defeito não dá erro. O sistema fica coerente consigo mesmo — a linha existe, o
novo endereço funciona, todo teste passa. O que quebrou está fora, na mão de
alguém que você não pode consultar e cujo prejuízo você não vê. É a classe de
dano que nunca chega como bug report, só como movimento que sumiu.

## Dois casos

- **O @ do perfil no [[Privello]].** O endereço saía do nome ("helena, Porto
  Alegre" virava `helena-porto-alegre-7`), então trocar o nome derrubava todo
  link já mandado. Passou a existir um @ escolhido por ela, separado do nome: o
  nome muda quantas vezes quiser e o @ trava 30 dias entre trocas. O que a
  cidade no caminho e a caixa da letra causam já redireciona; o endereço do @
  anterior ainda morre na troca, e é dívida conhecida — a trava segura a
  frequência do estrago, não o estrago.
- **O token que já estava na caixa de entrada** —
  [[Registro que muda de casa leva junto o token já distribuído]]. A migration
  que promovia um modo a entidade própria ia gerar identificadores novos: no
  banco tudo batia, e o `/f/<token>` mandado por e-mail na semana anterior
  respondia "link inválido".

O primeiro é sobre o campo de onde a identidade nasce; o segundo, sobre ela
sobreviver a um refactor. A pergunta por trás dos dois é a mesma: **quem, fora
daqui, já está segurando isto?**

## Quando a troca precisa existir: o velho redireciona E fica reservado

A trava de prazo segura a frequência, não o estrago. Trocado o identificador, o
endereço antigo vira 404 e quem o guardou não fica sabendo — e ninguém volta
para avisar. Duas linhas resolvem, e a segunda quase sempre falta:

- **O velho redireciona para o novo**, gravado na MESMA transação da troca.
  Escrito depois, uma falha no meio deixa o endereço morto — que é exatamente o
  estrago que a linha existe para evitar.
- **O velho NÃO volta para o mercado.** Liberado, outra pessoa pode tomá-lo, e
  aí o link antigo deixa de dar 404 para passar a abrir o perfil de um estranho.
  Numa vitrine de pessoas isso é pior que morrer: quem clicou acha que chegou.

Daí sai uma consequência que surpreende: **a conferência de disponibilidade
passa a olhar duas tabelas**, a dos vivos e a dos aposentados. E ela precisa ser
uma função só — no [[Privello]] são três telas perguntando a mesma coisa (o
cadastro, a troca, e a checagem que roda enquanto a pessoa digita), e três
cópias é como uma passa a oferecer o que a outra recusa.

O vivo sempre ganha do aposentado na hora de resolver o endereço. Se alguém está
usando aquele identificador agora, é para ele que o link leva — a ordem das duas
consultas é a regra, não estilo.

## Conexões
- Irmã: [[Índice só é identidade enquanto a coleção não muda]]
- Depende de: [[Um invariante se garante na estrutura, não no processo]]
- Técnica que aplica: [[Registro que muda de casa leva junto o token já distribuído]] ·
  [[Renomear coluna é migration à mão; a gerada derruba e recria]]
- Visto em: [[Privello]] · [[Navetech Hub]]
- Mapa: [[Base]] · [[Backend]]
