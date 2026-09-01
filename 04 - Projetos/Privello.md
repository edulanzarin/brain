---
tags: [tipo/projeto, projeto/privello]
criado: 2026-08-29
status: ativo
codigo_em: ~/Dev/privello
---

# Privello

> Classificado de acompanhantes por cidade, com verificação de documento antes
> da publicação. Planos para quem anuncia (teto de mídia, story e destaque), um
> passe para quem procura, e a assinatura de perfil — o feed pago da própria
> acompanhante, com repasse. Domínio já registrado; a referência de mercado é o
> FatalModel na primeira metade e o Privacy na segunda.

Código em: `~/Dev/privello`

## Estado atual

**Primeiro corte fechado (ago/2026).** Vitrine pública, painel de quem anuncia e
moderação, rodando no compose e com build de produção passando. Histórico
linear na `main`, sem remote ainda.

Ponta a ponta exercitado: cadastro, criação do anúncio, envio de documento,
aprovação na moderação, publicação, upload de mídia dentro do teto do plano,
denúncia e suspensão. `npm run verificar` cobre 88 regras contra o banco de dev.

**O lado de quem procura fechou em 01/09/2026.** Até aqui a nota afirmava story,
compra de VIP e avaliação como exercitados, e os três eram leitura sem escrita:
o trilho de story desenhava sem haver como enviar um, `assinarPlano` só aceitava
plano de anúncio, e nada criava uma `Avaliacao` — os números do perfil vinham da
semente. Favoritar era o caso mais nu: o botão não tinha ação e `/favoritas` lia
uma tabela que ninguém escrevia. Virou princípio:
[[Recurso sem escrita parece pronto quando a semente preenche a leitura]].

**O Privacy entrou (01/09/2026).** O segundo corte combinado deixou de ser só o
catálogo em `/design/privacy`. Quem publica põe preço no próprio feed, escreve
post aberto ou exclusivo, e o que entra vira saldo com extrato e saque; a casa
fica com 20% por padrão, e o percentual mora no anúncio porque o acordo é por
pessoa. Modelo, portão e carteira nas Decisões abaixo.

**O feed virou área própria (01/09/2026).** Ele nasceu como bloco dentro do
perfil e saiu de lá: `/feed` é a timeline de quem a pessoa assina, `/feed/@duda`
é o de uma pessoa, e o anúncio ficou só com a chamada. Casca própria, fora do
grupo `(site)` — [[A casca se compartilha por público, não por marca]]. Junto
entraram curtir e comentar, com uma regra só: você interage com o post que você
consegue ver.

**A fronteira de pagamento (01/09/2026).** Esta nota afirmava que a interface
`ProvedorPagamento` estava pronta para receber um adquirente. Não estava: eram
quatro `provedor: "simulado"` escritos à mão, e a assinatura nascia já valendo
com o pagamento gravado `PAGO` na mesma transação — o `PENDENTE` existia no enum
e nada o escrevia. Agora a venda tem duas metades e as duas rodam: `vender` cria
a compra sem valer e chama o provedor, `confirmarPagamento` é a única porta que
faz uma assinatura passar a valer, e é ela que o webhook vai chamar sem mudar uma
linha. Detalhe em
[[Provedor de pagamento entra por interface, e o simulado é a primeira implementação]].

**Editar o anúncio (01/09/2026).** `valorHoraCentavos` só era escrito na
criação: errar o preço no cadastro obrigava a refazer o anúncio inteiro, e é o
dado que muda toda semana. Entram três telas no painel — valores e contato, onde
atende, sobre mim —, com o mesmo vocabulário dos passos do cadastro. A regra de
cada campo saiu para `anuncioCore` e o cadastro passou a usar os mesmos leitores:
[[Dado escrito por dois caminhos precisa de uma regra só, fora dos dois]].

**Favoritar, passe e avaliação (01/09/2026).** As três escritas que faltavam do
lado de quem procura, mais a página `/passe` com endereço próprio — o perfil
mandava "assinar o passe" para `/anunciar`, que é a tela do outro público. Duas
das três coisas que o passe vende não valiam nada: o telefone de contato fechado
não saía nem para quem assinasse, e o story exclusivo era um selo por cima da
foto verdadeira.

**Story ganhou porta (01/09/2026).** O visor, o trilho e o prazo já existiam;
faltava o envio. Sobe por `/api/story` pelo mesmo motivo da mídia, e o teto do
plano conta os que estão NO AR em vez dos do dia de calendário.

**Ponta a ponta fechado (30/08/2026).** O anúncio agora nasce, se monta e vai
ao ar sem passo manual: cadastro em cinco passos, mídia com upload de verdade,
expediente, etiquetas, documento com selfie, fila de conferência em `/admin` e
plano com pagamento simulado. Falta a fila de denúncias no admin, e o
adquirente de verdade.

**Denúncia e suspensão (30/08/2026).** O botão de denunciar era fachada — fechava
o modal e mostrava torrada de sucesso sem escrever linha nenhuma. Agora grava, e
existe fila em `/admin/denuncias` agrupada por anúncio, com tirar do ar, voltar
ao ar e descartar, todos escrevendo evento de moderação. Junto nasceu o
`npm run verificar`: 33 regras exercitadas contra o banco de dev.

**Vídeo de comparação (30/08/2026).** A coluna `midiaDeComparacao` existia sem
nenhum caminho que a preenchesse. Agora tem: a casa sorteia um código, ela
escreve num papel e grava segurando, a moderação confere numa fila própria e o
anúncio ganha o selo de vídeo verificado. Opcional e fora da régua de passos.

**O caminho fechou (30/08/2026).** A revisão deixou de ser um beco: o nono passo
carrega o desfecho no rodapé — resolver o que trava, concluir, ou abrir o perfil
no ar —, e a recusa da moderação voltou a travar a publicação em vez de passar
por documento entregue.

**Identidade do perfil (30/08/2026).** O endereço deixou de sair do nome e virou
um @ escolhido por quem anuncia, com trava de 30 dias entre trocas; entrou o
recado do dia, com prazo de 24 h; e o expediente saiu da ficha para um modal.
Detalhe abaixo, em Decisões.

## Infra

Slug `privello` · app `privello-app` na `4075` · banco `privello-db` na `5075`.
Compose com app e db, imagem standalone, migrations e seed no entrypoint, mídia
em volume próprio. Chassi e mapa de portas em [[Infra]].

## Stack

Next.js 16 (App Router, Server Components, Server Actions) · React 19 ·
TypeScript · Postgres 16 com Prisma 7 · sessão em cookie assinado com HMAC,
escrita à mão em `server/sessao.ts` (sem Auth.js: o que o produto precisa é
guardar um id de um jeito que o navegador não forje, e isso é dez linhas —
[[Sessão de painel interno é um cookie assinado, não uma tabela de sessões]]) ·
Tailwind v4 (tokens em `@theme`, classes em `@layer components`).

## Decisões importantes

- **Papel é permissão, não conta separada.** Todo mundo entra como `CLIENTE`;
  abrir anúncio vira `ACOMPANHANTE` no mesmo login. Quem anuncia também assina o
  VIP, e nenhum dos dois lados precisa de segundo cadastro.
- **Nenhum caminho leva o anúncio de rascunho ao ar.** Ele passa obrigatoriamente
  por `EM_ANALISE`, e quem publica é a moderação depois de conferir documento com
  selfie. É a trava dos 18 anos, e ela é estrutural: não existe transição direta
  no código.
- **O passe VIP destrava três coisas, e só três**: story marcado como exclusivo,
  contato de quem escolheu atender só assinante, e as avaliações (ler e
  escrever). Plano único, o que muda é a duração. As três checagens moram no
  servidor — quando o contato é fechado, o telefone não chega ao HTML.
- **A receita é espaço de anúncio e assinatura, nunca percentual sobre
  encontro.** Não existe campo de comissão no schema, e a ausência é deliberada:
  cobrar pelo anúncio é classificado, comissionar o programa é outra natureza de
  serviço.
- **Documento de verificação não divide nada com a mídia pública** — outro model,
  outra pasta, outra rota, aberta só para a moderação.
- **Cidade não vira página só por estar no cadastro.** A rota, o índice e o
  sitemap saem da mesma consulta de "onde há anúncio ativo" —
  [[A superfície indexável sai da mesma consulta que o conteúdo]].
- **Pagamento nasce como fronteira.** O simulado existe porque Stripe, Mercado
  Pago e PagSeguro proíbem conteúdo adulto em contrato: o fluxo de venda inteiro
  precisava existir antes de haver adquirente —
  [[Provedor de pagamento entra por interface, e o simulado é a primeira implementação]].
- **Estética própria, escura, celular antes de tudo.** Não herda o dialeto de
  dashboard dos sistemas internos; o público é visitante de telefone, não
  operador — [[Estética é por projeto, princípio de design é que se reusa]].
- **Expiração de assinatura se apura na leitura** do painel e do admin, sem
  agendador — [[Rotação por período se apura na leitura, e dispensa agendador]].
- **Nome e @ são duas identidades com prazos diferentes.** O nome é como ela se
  apresenta hoje e muda quantas vezes quiser; o @ é o endereço, trava 30 dias
  entre trocas e é o que a busca e o link seguram. Antes o endereço saía do nome,
  então cada troca de nome matava calada todo link já mandado —
  [[Identificador que já circulou não é mais seu para mudar]]. O cartão da
  vitrine continua mostrando o NOME: quem identifica é o @, quem se lê é o nome.
- **O @ mora no anúncio, não no usuário.** Quem só procura não tem endereço
  público para alguém abrir, e a garantia é a ausência da coluna, não uma tela
  escondida — [[Um invariante se garante na estrutura, não no processo]].
- **A arroba não entra no endereço**, como no Instagram: mostra `@duda`, endereça
  `/duda`. Não é gosto — o App Router recusa segmento que começa com `@` e
  devolve 404 antes de a página rodar
  ([[Segmento de URL que começa com @ não chega ao App Router]]). Em compensação
  o atalho `/duda` convive com `/rs` na raiz, porque sigla de estado tem duas
  letras e @ tem no mínimo três.
- **A trava de 30 dias é do servidor.** O botão desabilitado é conveniência; quem
  recusa é a ação, que compara `arrobaTrocadaEm` com agora —
  [[Permissão se valida no servidor, não na interface]].
- **Recado do dia vale 24 h**, como o story e pelo mesmo mecanismo: o texto fica
  na linha e a leitura pergunta se ainda vale, sem agendador. Reescrever renova o
  prazo; herdar as duas horas que sobraram derrubaria o recado novo antes de
  alguém ler.
- **O expediente é modal, não bloco da ficha.** Sete linhas de horário no meio do
  perfil cobram uma tela de rolagem de todo mundo para responder a pergunta de
  poucos; o selo "em expediente" ao lado do nome já responde a pergunta comum —
  [[O que responde pergunta rara não ocupa a rolagem de todo mundo]]. O quadro
  mostra os sete dias, inclusive os fechados, porque o que sobra é "atende
  terça?".
- **O player de vídeo é nosso, não o nativo.** O do navegador traz barra cinza
  de altura fixa, tipografia do sistema e um menu com "baixar vídeo" — num
  perfil de acompanhante, um convite que a casa não quis fazer —, e cada
  navegador desenha o seu, então a mesma tela tinha três caras. O nosso é feito
  para o polegar: a área toda toca e pausa, a barra some enquanto toca, e o
  primeiro toque só acorda em vez de pausar.
- **Um passo a passo só, do plano ao ar.** Nove passos: os cinco primeiros
  antes de o anúncio existir (rascunho no navegador, preso à conta) e os
  quatro últimos depois. O anúncio nasce entre o quinto e o sexto, calado.
  Duas réguas para o mesmo trabalho era o mesmo que régua nenhuma.
- **O plano é o primeiro passo.** É ele que decide quantas fotos e se cabe
  vídeo; perguntado no fim, a pessoa monta tudo com um teto que ninguém
  ofereceu. Trocar depois é tela do painel, não passo do caminho.
- **O painel é a continuação do cadastro, não uma parede.** Cada assunto tem
  página própria e a home responde uma pergunta: o que falta agora. A tela de
  revisão fecha o caminho — mostra o que trava, o que só recomenda, e o cartão
  REAL da vitrine com os dados dela.
- **A revisão fecha o caminho, e o desfecho depende de onde o anúncio está.** São
  três, e o rodapé mostra um: resolver a primeira falta, concluir (quando o que
  falta é a moderação olhar), ou ver o perfil no ar. Nove passos terminando em
  rodapé vazio faziam a pessoa percorrer a régua inteira sem descobrir se acabou
  ([[Último passo sem desfecho transforma a régua em beco]]).
- **A dona vê o próprio anúncio antes do ar.** Prévia com faixa, e 404 para
  qualquer outra pessoa. Sem isso o primeiro a ver como o perfil ficou era o
  público.
- **O vídeo de comparação é desafio, não upload.** Um vídeo qualquer prova que
  existe um vídeo daquela pessoa em algum lugar; o código sorteado, com prazo de
  48 h, é o que prova que a gravação é de depois do pedido
  ([[Artefato prova existência; só um desafio prova o momento]]). Model próprio
  e não coluna em `Verificacao`: o documento é obrigatório e privado para
  sempre, o vídeo é opcional e vira conteúdo público, e só o vídeo tem a fase
  entre pedir o código e mandar a gravação.
- **Dois selos, e nunca um.** "Documento conferido" a casa viu e ninguém mais
  vê; "Vídeo verificado" está logo abaixo no perfil e qualquer um confere
  sozinho. Um selo só para os dois faria quem procura achar que viu o que não
  viu — e é o segundo que sustenta a decisão de chamar, porque é o único que ela
  pode auditar.
- **Vídeo aprovado MUDA de pasta.** Enquanto está na fila mora em `documento/`,
  sem rota aberta; aprovar move para `publico/`. Gravar o caminho no anúncio sem
  mover deixaria o perfil apontando para uma rota que recusa por prefixo
  ([[Se quem decide o acesso é a pasta, aprovar é mover o arquivo]]). Recusado
  sai do disco.
- **A fila de denúncias agrupa por ANÚNCIO e ordena por quantidade.** Sete
  denúncias do mesmo perfil são uma decisão, não sete
  ([[Fila de decisão agrupa pelo alvo, não pelo aviso]]). É o inverso da fila de
  conferência, que ordena pela mais antiga — lá cada item é alguém esperando
  resposta, aqui é gravidade. As duas moram no mesmo painel e ordenam ao
  contrário de propósito.
- **Suspender não apaga nada, e por isso pode ser rápido.** Fotos, expediente e
  avaliações ficam; muda o estado, que é o que a vitrine consulta. Voltar ao ar
  é um clique, e é por isso que os suspensos ficam na mesma tela da fila. A
  decisão difícil seria apagar, e ela não existe no admin.
- **Restaurar não é porta dos fundos para PUBLICADO.** Anúncio suspenso que
  nunca teve documento aprovado volta para a fila de conferência, não para o ar.
  É a mesma trava dos 18 anos aplicada à transição que parece só desfazer o que
  a própria moderação fez — e é a regra que o `npm run verificar` mais protege.
- **A moderação é a única porta para PUBLICADO.** Nem o cadastro nem a compra
  de plano movem um anúncio para lá: a transição só existe em
  `server/moderacao.ts`, depois de alguém olhar documento e selfie lado a
  lado. É assim que a trava dos 18 anos é estrutural e não combinada.
- **Documento não divide pasta com a mídia pública.** `publico/` tem rota
  aberta; `documento/` não tem rota nenhuma além da de admin, que confere
  sessão E prefixo de pasta. Conferido: a própria dona do anúncio recebe 404
  ao pedir o próprio documento.
- **Assinar não publica, e a tela diz isso antes do clique.** Pagar e
  continuar fora do ar é a decepção mais cara que este produto pode entregar.
- **Sem anúncio, o painel É o formulário de criação.** E é lá que o @ se
  escolhe — primeiro campo, sozinho no cartão, com a trava de 30 dias dita
  antes e a disponibilidade conferida enquanto digita
  ([[Campo que trava depois de escolhido não vai no meio do formulário]]). O
  anúncio nasce em RASCUNHO: não existe caminho do formulário para PUBLICADO.
- **"Endereço" não aparece em texto que quem anuncia lê.** A palavra já tem
  dono neste domínio — é onde ela mora, o único dado que não pode ser
  publicado. Fala-se do @ e do link
  ([[Palavra da interface é lida com o dicionário do usuário, não com o seu]]).
- **O nome da conta e o nome do anúncio são duas pessoas.** A conta tem o nome
  de registro, o anúncio tem o público, e o campo do anúncio NÃO vem preenchido
  com o da conta: o padrão aceito sem pensar publicaria o nome de registro dela.
- **A conta é da conta, o painel é do anúncio.** E-mail, senha e favoritos
  ficam em `/conta`; nome, @, recado, valores e números ficam no painel. Dois
  lugares para editar o mesmo @ é onde um deles envelhece.
- **Nome, @ e recado editam no mesmo cartão.** É a vizinhança que ensina os três
  prazos — muda quando quiser, trava 30 dias, some em 24 h. Espalhados por três
  telas, cada prazo vira surpresa na hora do erro.
- **Preço se corrige sem passar pela moderação.** O documento prova que ela é
  maior de idade, e preço não é afirmação sobre isso: mandar cada correção de
  tabela para a fila faria a fila crescer com o que ninguém precisa conferir. O
  que a edição CONTINUA travando é o nascimento: `lerSobre` recusa data de menor
  de 18, porque depois da conferência a data segue editável e o perfil poderia
  passar a anunciar 16 anos com o selo de conferido ao lado.
- **Um teto de preço, e ele existe contra o engano de digitação.** R$ 99.999 numa
  hora é quem digitou centavos achando que digitava reais; o erro entra no `Int`
  sem reclamar e some do fim de toda ordenação por preço.
- **Favoritar é uma ação, não duas** —
  [[Alternar é uma ação só, porque quem sabe o estado é o banco]]. E sem login não
  é recusa, é `precisaEntrar`: favoritar é a única coisa no site que dá motivo
  para quem só procura criar conta, e gastar esse momento com "faça login" é
  perdê-lo.
- **O passe tem tela própria e argumento próprio.** `EscolhaDePasse` não é
  `EscolhaDePlano` com um interruptor: o que cada uma vende é a frase antes do
  clique, e são frases opostas — lá é "assinar não publica", aqui é "isto abre
  agora".
- **Avaliação é uma por pessoa por anúncio, e escrever de novo troca.** A
  alternativa é nota errada por engano de clique virando permanente, e quem quer
  corrigir sem poder cria outra conta. A data NÃO se move na troca: `criadoEm` é
  quando o atendimento foi relatado, e é por ela que a lista ordena — renovada a
  cada edição, bastaria trocar uma vírgula para voltar ao topo do perfil.
- **Apagar a própria avaliação não pede passe.** O que ela escreveu continua
  sendo dela depois de a assinatura vencer; prender o texto no perfil de outra
  pessoa por falta de pagamento é cobrança, não portão.
- **A marca de exclusivo do story se escolhe ANTES do arquivo.** Story dura 24 h:
  uma foto que entra aberta e vira exclusiva dez minutos depois já foi vista por
  quem não assina, e a marca passa a mentir sobre o que aconteceu.
- **Story é foto, e a recusa é dita.** O visor desenha `<img>`, e vídeo apareceria
- **A assinatura nasce com o intervalo vazio, e é isso que a faz não valer.** Não
  existe coluna dizendo "confirmada": quem responde se uma assinatura está de pé é
  `fimEm > agora`, e sempre foi. Uma bandeira a mais seriam duas verdades sobre a
  mesma pergunta.
- **O prazo se conta na confirmação, não na venda.** Entre pedir e pagar pode
  passar um dia, e é isso que mantém honesta a soma de quem renova antes de vencer.
- **O que é consequência de ativar mora na confirmação**, e não em quem vendeu —
  acender o destaque, no caso. Deixado do lado da venda, o webhook ativaria a
  assinatura e esqueceria a bandeira.

- **A assinatura de perfil é model próprio, não mais uma linha em `Assinatura`.**
  Lá o que se compra é um plano do cadastro, com preço da casa e sem repasse;
  aqui o preço é de quem publica e parte volta para ela. A mesma coluna
  significando "assinatura PARA este anúncio" e "assinatura DESTE anúncio" seria
  a divergência mais cara que este schema poderia ter.
- **Mídia de post é tabela própria** pelo mesmo tipo de razão: a do anúncio
  carrega papéis que só existem na vitrine, e pendurar post na mesma obrigaria
  toda consulta da vitrine a lembrar de excluir o que é do feed — a primeira que
  esquecesse publicaria conteúdo pago na busca da cidade.
- **Preço e repasse congelam na compra** —
  [[O acordo congela na linha, a política vale do próximo em diante]]. Baixar a
  taxa faria o saldo de todo mundo subir sozinho, inclusive o de quem já sacou.
- **Não existe tabela de saldo.** Ele é a soma dos repasses liberados menos a
  dos saques não recusados, então recusar um saque devolve o valor sozinho e
  não há estorno a escrever. Saque pedido já sai do disponível, senão a pessoa
  pede duas vezes o mesmo dinheiro —
  [[Balancete é movimento do período, saldo é consequência]].
- **Sete dias até liberar**, porque estorno e contestação chegam depois do
  pagamento: repassar na hora é a casa pagar do próprio bolso quando o dinheiro
  volta.
- **O feed se desliga pela ausência do preço**, não por uma bandeira a mais
  dizendo a mesma coisa. Tirar o preço para de vender e não cancela o que já foi
  vendido: quem assinou continua até o prazo acabar.
- **O primeiro post é a amostra.** Feed inteiramente trancado não tem o que
  vender, porque quem não assina não vê nem que existe conteúdo. O portão manda
  QUANTAS mídias existem e não manda o caminho de nenhuma.
- **A marca de exclusivo é do compositor, não da grade.** Post que entra aberto
  e fecha dez minutos depois já foi visto por quem não assina.
- **Marcar saque como pago não move dinheiro**: o Pix sai por fora enquanto não
  houver adquirente. A tela registra que saiu, para quem pediu saber e para a
  próxima pessoa da casa não pagar duas vezes.
- **O feed não mora no anúncio.** O classificado se lê uma vez para decidir
  chamar; o feed se volta a abrir toda semana. Empilhados, o segundo vira rodapé
  do primeiro, e quem assina passaria a entrar pela parte que já leu. No anúncio
  fica a chamada; o feed tem área e casca próprias.
- **A timeline é o que faz a assinatura durar.** Sem ela, quem assina três
  pessoas abre três endereços e acaba não abrindo nenhum — a renovação deixa de
  acontecer por esquecimento, não por decisão.
- **Uma regra decide curtir e comentar**, e é a mesma do portão: `podeLer`.
  Duas checagens separadas é como uma passa a aceitar o que a outra recusa.
- **Post fechado não manda contador.** Quantas curtidas tem o que a pessoa não vê
  não conta nada a ela, e conta para quem quer medir o feed de outra sem pagar.
- **Comentário apaga quem escreveu E a dona do feed**, que senão hospeda no
  próprio post o que não pode tirar.
- **Os comentários chegam no primeiro clique**, não com a leva do feed: doze
  posts com trinta comentários cada é meio mega de resposta para o que ninguém
  abriu.
- **Peça sem ação não desenha ação.** `Post` virou burra — `curtido` chega pronto
  e curtir vem de fora — e `Midia` parou de desenhar um menu que não abria nada.
  como quadro em branco correndo sozinho.

## Aprendizados (viraram notas)

- [[A superfície indexável sai da mesma consulta que o conteúdo]]
- [[Página que consulta o banco não pode nascer no build]]
- [[Portão de conteúdo cobre a tela, não o HTML]]
- [[Linha no banco não garante o arquivo no disco]]
- [[Propriedade com prefixo escrita à mão pode perder a versão padrão no build]]
- [[Identificador que já circulou não é mais seu para mudar]] — o princípio que
  saiu daqui, com o segundo caso vindo do [[Navetech Hub]].
- [[O que responde pergunta rara não ocupa a rolagem de todo mundo]]
- [[Segmento de URL que começa com @ não chega ao App Router]]
- [[A segunda ação do formulário se marca no botão, não no estado]]
- [[Renomear coluna é migration à mão; a gerada derruba e recria]]
- [[Nome de migration do Prisma é UTC, e é o nome que ordena]]
- [[Margem negativa em item de flex centralizado vale metade]]
- [[Ponto em preço brasileiro é ambíguo, e quem desempata é a contagem de casas]]
- [[Campo que trava depois de escolhido não vai no meio do formulário]]
- [[Palavra da interface é lida com o dicionário do usuário, não com o seu]]
- [[Escolha única e múltipla não usam o mesmo controle]]
- [[Arquivo não sobe por server action, o corpo dela tem 1 MB]]
- [[Consulta sem ordem não é determinística, e semente que usa o índice muda sozinha]]
- [[Rascunho no navegador leva o dono na chave]]
- [[Ajustar estado no render é legítimo, empurrar rota não é]]
- [[Último passo sem desfecho transforma a régua em beco]]
- [[Registro com estado não se confere pela existência]]
- [[Peça de grade mostrada sozinha vai centrada num palco]]
- [[Artefato prova existência; só um desafio prova o momento]]
- [[Se quem decide o acesso é a pasta, aprovar é mover o arquivo]]
- [[Código que a pessoa copia à mão não pode ter caractere ambíguo]]
- [[Tela que manda comparar duas coisas mostra as duas]]
- [[A regra mora fora da porta que a chama]] — o princípio que saiu daqui, com o
  primeiro caso vindo do [[monofire]].
- [[Peça de mentira que não se anuncia vira fundação de coisa real]]
- [[Fila de decisão agrupa pelo alvo, não pelo aviso]]
- [[Recurso sem escrita parece pronto quando a semente preenche a leitura]] — o
  princípio que saiu daqui, com quatro casos no mesmo projeto.
- [[Dado escrito por dois caminhos precisa de uma regra só, fora dos dois]]
- [[Alternar é uma ação só, porque quem sabe o estado é o banco]]
- [[Estado bloqueado aponta para a chave]]
- [[O acordo congela na linha, a política vale do próximo em diante]]
- [[A casca se compartilha por público, não por marca]]

## Próximos passos

- [ ] Limite de denúncia por origem. Denunciar funciona sem login, e sem login
      não há identidade: hoje nada impede vinte avisos do mesmo navegador. É
      escolha consciente — o aviso de menor de idade não pode custar cadastro —,
      mas a fila fica exposta a enxame.
- [ ] **Gateway real.** É o bloqueio comercial, não técnico: precisa de PIX
      direto por PSP que aceite o segmento, ou adquirente high-risk. Agora a
      interface existe mesmo (01/09/2026), e o que falta do lado do código é uma
      implementação de `ProvedorPagamento` e a rota de webhook que chama
      `confirmarPagamento` — que já roda hoje, pela boca do simulado.
- [ ] Saque automático por Pix. Hoje a casa marca "paguei" e manda por fora, e
      automatizar depende do mesmo adquirente que ainda não existe.
- [ ] Direto (mensagem) entre quem assina e quem publica. A peça `Conversa` já
      está no catálogo servindo os dois lados, e não tem escrita do outro lado.
- [ ] Rolagem infinita na timeline. Hoje ela traz a primeira leva e para; a
      consulta já devolve `temMais`.
- [ ] Mídia em armazenamento de objeto (R2 ou Backblaze) implementando
      `Armazenamento` — disco local não passa de uma máquina.
- [ ] Revisão jurídica de termos e privacidade, e definição do encarregado de
      dados da LGPD.
- [ ] Redimensionar imagem no upload; hoje o arquivo original é servido como veio.
- [ ] Busca por nome e filtro de faixa de preço na listagem da cidade.
- [ ] Redirecionar o endereço antigo do @ depois de uma troca. Hoje a troca é
      permanente e o link velho morre — a trava de 30 dias segura a frequência,
      não o estrago.

## Conexões
- Usa: [[Design]] · [[Infra]]
- Mapa: [[Projetos]]
