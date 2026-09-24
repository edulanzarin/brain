---
tags: [tipo/projeto, projeto/telebot]
criado: 2026-09-17
status: ativo
codigo_em: C:/Dev/telebot
---

# telebot

> Plataforma para vender acesso a grupo VIP no Telegram. O criador conecta um
> bot, cadastra os planos, e o bot cobra no Pix, entrega um convite de uso único
> quando o pagamento cai, avisa antes de vencer e remove quem não renovou. O
> painel acompanha tudo ao vivo.

Código em: `C:/Dev/telebot` (sem remote git).

Nome provisório, a decidir antes de qualquer registro de domínio ou marca.

## Histórico

- **17/09/2026, v1**: primeira versão, ponta a ponta com provedor simulado.
  Existe no histórico da `main` até o commit `8754c3f`.
- **24/09/2026, reescrita do zero** a pedido do Eduardo ("mais bonito, dinâmico,
  moderno, robusto"), sem reaproveitar o código da v1, só o domínio. Feita num
  worktree na branch `feat/reescrita` e levada à `main` por fast-forward.
- **24/09/2026, identidade visual nova.** O Eduardo olhou a reescrita e disse
  "ainda não curti muito, tem como fazer melhor". O diagnóstico: cara de template
  (tudo o mesmo cartão azul-marinho, cor emprestada do próprio Telegram, selo "Pago"
  em toda linha, nada se mexia quando a venda chegava). Refeito na branch
  `feat/visual-ingresso` (commits `ebe98c1` a `d6365af`).

Existe ainda um `C:/Dev/bitpay-bots` parado desde a formatação de set/2026 com
a mesma ideia (Next + Prisma + Stripe). Não é base deste; fica registrado para os
dois não parecerem duplicata.

## Estado atual

Ponta a ponta com o provedor simulado, conferido em execução real, não só
compilando:

- 48 testes de unidade (regras de taxa, acesso, aviso, chave Pix, textos,
  formatação, cifra, fila, assinatura do Mercado Pago) e 8 de integração contra
  Postgres real: cinco confirmações simultâneas creditam uma vez só, renovação
  empilha o prazo, livro-caixa recusa edição, saque, rodada de manutenção,
  conversa inteira do bot pelo dublê do Telegram e vínculo de grupo por código.
- Build de produção, prints de todas as telas em 1440 e 390, login dirigido no
  Edge headless, SSE recebendo o evento gravado no banco, webhook do Telegram
  (segredo, reentrega) e o standalone montado numa pasta limpa, como o
  Dockerfile monta, subindo só com variáveis de ambiente.
- O compose rodou local em 24/09/2026 (`docker compose up -d --build`): db, migrate
  e app saudáveis, painel logado respondendo em http://localhost:4081.

Não está no ar: falta domínio com HTTPS e credencial do Mercado Pago.

## Infra

Slug `telebot` · app `telebot-app` na `4081` · banco `telebot-db` na `5081`.
Compose com `telebot-db`, `telebot-migrate` e `telebot-app`, sem Caddy. Roda
só local: o Eduardo não quer o telebot no ts05 (é projeto pessoal, não da Navecon), e o destino público ainda não foi escolhido. O agendador separado da
v1 saiu: o trabalhador (fila, manutenção, polling) roda no processo do app, pelo
`instrumentation.ts`.

A v1 usava Postgres portátil na 5081; a reescrita usa o container, pela
convenção. O cluster portátil antigo (`%LOCALAPPDATA%\pgdata\telebot`) guarda o
esquema da v1 e não serve para a nova: parado, fica como estava.

## Stack

Next 16.3 (App Router, Server Actions, instrumentation), React 19.3, Tailwind v4,
Postgres via `pg` com SQL puro e runner próprio, zod, qrcode, lucide. Sem SDK de
Telegram nem de pagamento: as duas APIs são HTTP com JSON. Vitest. Fontes
Bricolage Grotesque (títulos) e Geist (corpo e números).

## Decisões importantes

- **Visual (desde 24/09/2026, "ingresso")**: o produto vende entrada, então plano e
  venda têm forma de ingresso, com canhoto picotado e o preço nele. Base grafite com
  fio de violeta (fundo `#0b0a11`, palco `#111018`), acento violeta `#a192ff` para
  interface, degradê violeta-rosa (`#ae97ff` → `#ea79cc`) SÓ no canhoto e no
  símbolo; menta, âmbar e vermelho seguem sendo dado; o azul do Telegram ficou só na
  prévia da conversa. A tela vive num "palco" de cantos redondos com luz violeta no
  alto; no celular a barra de baixo flutua. O momento memorável: a venda que chega
  ao vivo cai na pilha de ingressos do Início com um reflexo no canhoto, e a página
  inicial mostra o mesmo momento quando o Pix cai na conversa de demonstração.
  Número do dia em placar (Bricolage estreitada, R$ miúdo e centavos no alto).
  Listas por dia, selo só na exceção. Versão anterior (azul-marinho do Telegram)
  foi reprovada como genérica.
- **Ao vivo por `pg_notify` + SSE**: cada atividade gravada dispara NOTIFY; uma
  conexão LISTEN por processo serve todos os painéis. Segue
  [[Estado vivo se empurra, não se pergunta]].
- **Uma porta só para a venda** (`confirmarPagamento`), chamada por webhook, "Já
  paguei", simulação e varredura; idempotente com pedido `FOR UPDATE` e índice
  único de movimento de venda.
- **Livro-caixa imutável por trigger** e saldo com `check (saldo >= 0)`: a
  constraint é a trava final contra saque maior que o saldo.
- **Efeito no Telegram vai pela fila** do próprio Postgres, gravada na transação
  do estado. Ver [[Fila no Postgres entra na transação do estado, e o NOTIFY só acorda no commit]].
- **Situação do assinante é derivada** (view `membro_v`), não coluna: uma regra
  só para lead, ativo, vencido e removido.
- **Grupo e dono vinculados por código**, nunca pelo primeiro grupo onde o bot
  vira admin.
- **Bot de demonstração** com cliente dublê do Telegram, carimbado na tela: a
  conta demo mostra o produto inteiro sem bater na API.
- **Página pública de planos** (`/v/<bot>`) e link por plano que abre o bot já
  com o Pix pronto (`?start=p-<plano>`).
- **Aviso de venda no Telegram do criador**, pelo próprio bot dele.
- Mantidas da v1: token do BotFather cifrado (AES-256-GCM), preço e taxa
  congelados no pedido, convite com `member_limit: 1`, remoção como ban seguido
  de unban, checklist de ativação.

## Aprendizados (viraram notas)

- [[Sinal marca a exceção; o normal repetido em toda linha abafa o que importa]] —
  princípio novo, com [[Grade de iguais esconde o único item que funciona]] como
  segundo caso.
- [[Recorte por máscara corta a própria sombra; a sombra vai num drop-shadow do pai]]
- [[Degradê de SVG com id fixo some junto com a instância escondida que o define]]
- [[Na lista que recebe item ao vivo, a chave nova faz a entrada e a transição move o resto]]

- [[Afirmação que chega de fora só vale com um código que a casa emitiu antes]] —
  princípio novo em Segurança, promovido na segunda aparição.
- [[Bot que qualquer um pode pôr num grupo se vincula a ele por código]]
- [[Fila no Postgres entra na transação do estado, e o NOTIFY só acorda no commit]] —
  inclui a armadilha do teste disputando a fila com o app de pé.
- [[Linha do tempo ordena pelo tempo do fato, não pelo id]]
- [[Gráfico sem valor não tem escala, e o vazio tem desenho próprio]]
- [[Truncar come o fim da linha, e o valor não pode morar no fim do título]]
- Da v1: [[Fornecedor externo entra pelo contrato do app, não o app pelo dele]] ·
  [[Webhook de dinheiro precisa de duas travas, a do evento e a do efeito]] ·
  [[Dublê que não fecha o fluxo deixa o caminho sem ninguém passar]] ·
  [[Rolagem horizontal que não se anuncia esconde a coluna que decide]] ·
  [[Sem Docker na máquina, a imagem se confere montando o standalone numa pasta limpa]]

## Próximos passos

- [x] Passada de celular (24/09/2026): 390 e 360px sem estouro, folha de ações, saque antes do extrato, prévia de mensagem alternável.
- [x] Identidade própria ("ingresso"), 24/09/2026.
- [ ] Eduardo olhar a identidade nova e dizer o que muda.
- [ ] Decidir o nome de verdade.
- [ ] Escolher onde publicar (VPS e domínio próprios, fora do ts05) e credencial do Mercado Pago.
- [x] Rodar o compose inteiro (24/09/2026, local).
- [ ] Saque automático por Pix de saída (hoje a administração paga à mão).
- [ ] Remote git.

## Conexões
- Usa: [[Design]] · [[Infra]] · [[Backend]] · [[Dados]]
- Mapa: [[Projetos]]
