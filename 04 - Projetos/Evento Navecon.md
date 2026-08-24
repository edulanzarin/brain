---
tags: [tipo/projeto, projeto/evento-navecon]
criado: 2026-07-23
status: ativo
codigo_em: ~/Dev/evento-navecon
---

# Evento Navecon

> Landing page da imersão presencial da Navecon (marketplaces + inteligência
> tributária) com **inscrição e pagamento pelo Mercado Pago**. Inscrição grava
> no Postgres, gera a cobrança e, ao confirmar, dispara e-mail.

Código em: `~/Dev/evento-navecon`

## Estado atual

Pronto e **mergeado na `main`** (push feito). A stack sobe via
`docker compose up --build`. O **Mercado Pago foi validado com o token real** —
`/api/register` gera um checkout de verdade. Vai rodar em
**`navecon.net.br/imersao`** (subcaminho atrás do servidor web do TI, que já
serve a raiz); o suporte a subcaminho está pronto — ver
[[App sob subcaminho fica na raiz e o proxy tira o prefixo]]. Hardening feito
(helmet/CSP, rate limit, banco em loopback). Falta: o TI subir em `/imersao`
(proxy com strip do prefixo) e **validar o SMTP no ambiente real** (não testei
daqui pra não disparar alerta de login do Gmail).

**Cupom de cortesia 100% + ingresso digital (mergeado na `main`):** ingresso na
tela de sucesso com nome/data/local; cupom libera vaga grátis sem passar pelo MP.

**Painel `/admin` (mergeado na `main`):** inscrições e pagamentos com login por
usuário/senha da env, filtro/busca, CSV e ações (marcar pago manual, reenviar
link). Falta em produção: preencher `ADMIN_USER`/`ADMIN_PASSWORD` no `.env`.

**Cupom com código escolhido e N usos (12/08/2026, branch
`feat/cupom-multiuso-admin`):** trocou o código aleatório de uso único (e o CLI)
por código memorável (`NAVECON100`) com limite de usos, criado e gerido na aba
Cupons do painel. Validado ponta a ponta (criar, limite sob corrida, esgotado,
inativo, quem usou, excluir). A gestão de cupons agora é toda no `/admin`.

**Cupom com porcentagem de desconto (24/08/2026, mergeado na `main`):** o cupom
deixou de ser só cortesia — agora carrega `discount_percent` (1–100) escolhido no
painel. 100% continua pulando o Mercado Pago; 1–99% (o caso que pediram, 15%)
apenas abate o valor e o checkout segue normal. Validado contra o banco: criar,
resgatar, valor abatido, limite e esgotamento.

## Infra

Slug `evento-navecon` · app `evento-navecon-app` na `4099` · banco
`evento-navecon-db` na `5099`. Migrations em `evento-navecon-migrate`. Chassi e
mapa de portas em [[Infra]].

## Stack

Um app **Node/Express (TS, rodado com tsx)** serve o SPA já buildado **e** a API
na mesma origem · **Postgres 16** · **nodemailer** com SMTP do Gmail (senha de
app) · **Mercado Pago Checkout Pro** (redirect).

## Decisões importantes

- **Postgres, não planilha.** A ideia inicial era Google Sheets + Apps Script;
  trocada no meio por banco de verdade, já que sobe com Docker de qualquer jeito.
- **Um processo serve SPA + API na mesma origem.** Sem CORS, um container só
  (`<slug>-app`), o frontend chama `/api/register` relativo. Segue o chassi de
  [[Infra]].
- **Segredo de pagamento só no servidor.** O access token do MP fica no `.env`;
  a public key nem é usada (Checkout Pro por redirect não tokeniza cartão no
  cliente). Instância de [[Configuração vem do ambiente, não do código]].
- **Conciliação idempotente com dois gatilhos** (webhook + poller): o pix é pago
  depois e o deploy pode não ter IP público. Ver
  [[Polling substitui webhook quando não há IP público]].
- **6x com juros pro cliente** (a loja recebe o valor cheio) — decisão de
  negócio; muda só a config no painel do MP e o texto exibido.
- **Cupom de cortesia (100%) desvia do Mercado Pago.** Convidado ganha entrada
  por cupom; como o MP não aceita cobrança de R$ 0, o cupom válido pula o checkout
  — a inscrição nasce `paid`/`cortesia` e o resgate é um UPDATE atômico (o limite
  é do banco, não do fluxo). Inscrição órfã se desfaz se o resgate perde a corrida.
  Ver [[Consumir recurso de uso único é UPDATE condicional, não checar antes]].
- **Cupom é código escolhido com N usos, gerido no `/admin`.** Começou como código
  aleatório de uso único via CLI, mas trocou a pedido: código memorável
  (`NAVECON100`) com limite de usos, criado na aba Cupons do painel — dá pra
  divulgar um código só pra vários convidados. O resgate atômico virou
  `uses < max_uses`; "quem usou" vem de `registrations.coupon_code` (uma linha por
  resgate). O CLI saiu.
- **A porcentagem do cupom é que escolhe o fluxo.** Em vez de dois tipos de cupom,
  um campo só: 100% desvia do Mercado Pago (cortesia), 1–99% só muda o valor da
  cobrança. O resgate atômico devolve a porcentagem junto com a vitória na corrida
  (`RETURNING`), então ele vem antes de gravar a inscrição e a falha da gravação
  devolve o uso ao cupom. Ver
  [[Consumir recurso de uso único é UPDATE condicional, não checar antes]].
  Efeito colateral aceito: quem pega um cupom parcial e abandona o checkout queima
  um uso — o limite é de retirada, não de pagamento.
- **Ingresso digital na tela de sucesso.** Quem paga ou entra por cortesia vê um
  cartão com nome, data, local e localizador (8 chars do id da inscrição). O
  `GET /api/payment/status` passou a devolver o primeiro nome e o localizador só
  pra montar o cartão — nenhum outro dado pessoal sai daí.
- **Nada exposto direto.** App e Postgres publicam só em `127.0.0.1`. Em domínio
  próprio, o Caddy (compose de prod) termina o TLS; em `navecon.net.br/imersao`,
  quem termina o TLS e faz o proxy é o servidor web do TI. Mais helmet/CSP e
  rate limit no `/api/register`.
- **Subcaminho sem tocar no backend.** O app fica montado na raiz; o build do
  frontend deriva assets/endpoint/rota de um caminho base só (`APP_BASE_PATH`),
  e o proxy do TI tira o prefixo `/imersao`. Ver
  [[App sob subcaminho fica na raiz e o proxy tira o prefixo]].
- **Fonte da marca no allowlist da CSP.** Montserrat vem do Google Fonts; a CSP
  do helmet precisou liberar `fonts.googleapis.com`/`fonts.gstatic.com` — só
  quebrava em produção. Ver [[CSP só aparece no build de produção, toda origem externa vai no allowlist]].
- **Painel `/admin` server-rendered, não no SPA.** O painel é HTML gerado no
  Express (sem JS no cliente), o que evita mexer no build do Vite, no roteamento
  do SPA e na CSP (que só libera `script 'self'`) — as ações são forms POST. Fica
  no mesmo container, subcaminho `/admin`, sem link na home. Login por
  usuário/senha da env com sessão num cookie assinado (HMAC), ver
  [[Sessão de painel interno é um cookie assinado, não uma tabela de sessões]].
  Todo dado dinâmico é escapado (defesa contra XSS num render de string). Duas
  abas: **Inscrições** (resumo, filtro/busca, CSV, marcar pago, reenviar link) e
  **Cupons** (criar, ver uso X/N e quem usou, ativar/desativar, excluir).

## Fluxo operacional e lacunas

Como funciona hoje (uma via só): o visitante preenche o form → a inscrição é
gravada como `pending` **antes** do redirect e a responsável do mkt recebe um
e-mail de "nova inscrição" → o visitante é mandado pro checkout do MP → ao
aprovar, a conciliação vira `paid` e sai o e-mail de confirmação **pro inscrito**
(cópia pro mkt). Dois e-mails, ambos best-effort.

- **"Quero conversar antes de pagar" já é coberto em parte:** o lead não se
  perde — os dados ficam salvos em `pending` e o mkt é avisado, mesmo se a pessoa
  fechar o checkout.
- **Assistido pelo WhatsApp funciona:** o mkt cadastra os dados da pessoa no
  próprio site, pega o link do checkout do MP e manda pelo WhatsApp; a pessoa
  paga por esse link e o sistema confirma sozinho (e-mail pro e-mail que o mkt
  digitou).
- **Cadastrar sem passar pelo checkout já existe para convidados:** o cupom de
  cortesia é justamente uma via de inscrição sem pagamento (marca `paid`/`cortesia`
  direto). Não cobre "pagou por fora do MP" — para isso há o "marcar pago manual".
- **Painel `/admin` cobre a operação:** lista de leads/pagamentos com filtro/busca,
  export CSV, **marcar pago manual** (paga por fora do MP) e **reenviar o link** de
  pagamento sem recadastrar. Resolveu as lacunas operacionais que faltavam.
- **Lacunas que sobram:** e-mail automático de "faltou pagar" (lembrete); múltiplos
  usuários de admin com perfis (hoje é um login só, sessão stateless).

## Aprendizados (viraram notas)

Só links. O texto mora na nota de técnica/princípio.

- [[Polling substitui webhook quando não há IP público]]
- [[Migrations em container próprio no Docker Compose]]
- [[App sob subcaminho fica na raiz e o proxy tira o prefixo]]
- [[CSP só aparece no build de produção, toda origem externa vai no allowlist]]
- [[Consumir recurso de uso único é UPDATE condicional, não checar antes]]
- [[Sessão de painel interno é um cookie assinado, não uma tabela de sessões]]

## Próximos passos

- [x] Merge da branch `feat/mercadopago-checkout` na `main` (push feito)
- [ ] TI subir em `navecon.net.br/imersao`: `APP_BASE_PATH=/imersao/`,
  `PUBLIC_BASE_URL=https://navecon.net.br/imersao`, proxy com strip do prefixo
- [ ] Ativar **pix** e **parcelamento até 6x** no painel do Mercado Pago
- [ ] Validar o envio de e-mail no servidor real (SMTP não testado do ambiente de dev)
- [ ] Testar o pagamento em **modo de teste do MP** (token `TEST-` + cartões de teste)
- [ ] **Rotacionar** o access token e a senha de app (foram colados em chat)
- [ ] Decidir se entram as lacunas operacionais (pagar depois / painel / marcar pago manual)

## Conexões
- Usa: [[Design]] · [[Infra]]
- Mapa: [[Projetos]]
