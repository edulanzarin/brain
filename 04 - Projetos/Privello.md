---
tags: [tipo/projeto, projeto/privello]
criado: 2026-08-29
status: ativo
codigo_em: ~/Dev/privello
---

# Privello

> Classificado de acompanhantes por cidade, com verificação de documento antes
> da publicação. Planos para quem anuncia (teto de mídia, story e destaque) e um
> passe VIP para quem procura. Domínio já registrado; a referência de mercado é
> o FatalModel, e a segunda metade — venda de conteúdo por assinatura de perfil,
> tipo Privacy — fica para o corte seguinte.

Código em: `~/Dev/privello`

## Estado atual

**Primeiro corte fechado (ago/2026).** Vitrine pública, painel de quem anuncia e
moderação, rodando no compose e com build de produção passando. Sete commits na
`main`, sem remote ainda.

Ponta a ponta exercitado: cadastro, criação do anúncio, envio de documento,
aprovação na moderação, publicação, upload de mídia dentro do teto do plano,
story com prazo de 24 h, compra de plano e de VIP (provedor simulado),
avaliação, denúncia e suspensão. `npm run verificar` cobre 27 regras contra o
banco de dev.

**Ponta a ponta fechado (30/08/2026).** O anúncio agora nasce, se monta e vai
ao ar sem passo manual: cadastro em cinco passos, mídia com upload de verdade,
expediente, etiquetas, documento com selfie, fila de conferência em `/admin` e
plano com pagamento simulado. Falta a fila de denúncias no admin, e o
adquirente de verdade.

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
- **O painel é a continuação do cadastro, não uma parede.** Cada assunto tem
  página própria e a home responde uma pergunta: o que falta agora. A tela de
  revisão fecha o caminho — mostra o que trava, o que só recomenda, e o cartão
  REAL da vitrine com os dados dela.
- **A dona vê o próprio anúncio antes do ar.** Prévia com faixa, e 404 para
  qualquer outra pessoa. Sem isso o primeiro a ver como o perfil ficou era o
  público.
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

## Próximos passos

- [ ] Fila de denúncias no `/admin`. A de conferência existe; a de denúncia
      ainda não tem tela, e o modelo já guarda tudo que ela precisa.
- [ ] Suspender e despublicar pelo admin, com o evento de moderação junto.
- [ ] **Gateway real.** É o bloqueio comercial, não técnico: precisa de PIX
      direto por PSP que aceite o segmento, ou adquirente high-risk. A interface
      `ProvedorPagamento` já está pronta para receber.
- [ ] Assinatura por perfil (a metade "Privacy"): feed pago da acompanhante, com
      repasse. É o segundo corte combinado.
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
