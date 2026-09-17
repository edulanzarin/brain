---
tags: [tipo/projeto, projeto/telebot]
criado: 2026-09-17
status: ativo
codigo_em: ~/Dev/telebot
---

# telebot

> Plataforma SaaS para vender acesso a grupo VIP no Telegram. O criador conecta um
> bot, cadastra os planos, e o bot cobra no Pix, entrega um convite de uso único
> quando o pagamento cai, avisa antes de vencer e remove quem não renovou.

Código em: `~/Dev/telebot`

Nome provisório — a decidir antes de qualquer registro de domínio ou marca.

## Estado atual

Ponta a ponta com o provedor simulado, build de produção passando. Estão de pé:
vitrine, criar conta e entrar, painel com checklist de ativação do bot, ofertas e
planos, textos do bot editáveis, vendas, assinantes, carteira com saque, fila de
saques do admin, catálogo em `/sistema`, webhook do Telegram, notificação de
pagamento e a rodada de manutenção.

Conferido de verdade, não só compilando: três notificações do mesmo pagamento
geram **um** crédito, e o cron marca o acesso vencido mesmo quando a remoção no
Telegram falha, registrando a falha em vez de derrubar a rodada.

Sem remote git.

## Infra

Slug `telebot` · app `telebot-app` na `4081` · banco `telebot-db` na `5081`.
Chassi e mapa de portas em [[Infra]].

Em desenvolvimento o banco é cluster portátil, porque esta máquina não virtualiza
— [[Sem virtualização na BIOS não há Docker no Windows; o banco de dev vira Postgres portátil]].
Falta escrever o compose de produção.

## Stack

Next 16 (App Router, Server Actions), React 19, Tailwind v4, Postgres via `pg`.
Sem ORM: migrations em SQL puro com runner próprio
([[Runner de migration em SQL puro dispensa o CLI do ORM]]). Senha em scrypt do
Node, sessão opaca em tabela. Sem dependência de SDK para Telegram nem para
pagamento — as duas APIs são HTTP com JSON.

## Decisões importantes

- **Tema único, escuro.** Manter dois custa validar contraste e revisar arte em
  dobro, e o público olha isso de madrugada no celular. Segue
  [[Estética é por projeto, princípio de design é que se reusa]].
- **Token do BotFather cifrado no banco** (AES-256-GCM), com só os quatro últimos
  caracteres na tela. Em claro, um dump entregaria os bots de todos os clientes
  de uma vez.
- **Preço, taxa e duração congelados no pedido.** Reajuste de hoje não reescreve
  a comissão de uma venda de ontem — [[O acordo congela na linha, a política vale do próximo em diante]].
- **Carteira é tabela que só cresce**; o saldo na conta é espelho escrito na mesma
  transação do movimento. Erro vira movimento de ajuste, nunca edição do antigo.
- **O grupo é detectado, não digitado**: a pessoa promove o bot a admin e o
  `my_chat_member` entrega o `chat_id`. Pedir o número na mão seria pedir algo que
  ninguém sabe onde achar.
- **Convite com `member_limit: 1`.** Link comum vira print no grupo de pirataria e
  o acesso pago passa a valer para a internet inteira.
- **Remover é banir e desbanir em sequência.** O ban sozinho impede a pessoa de
  voltar mesmo comprando de novo, o que transforma renovação em suporte.
- **O painel abre com checklist do que falta para o bot vender.** Sem ele, um bot
  mal configurado parece pronto, e quem descobre o problema é o comprador que não
  entrou no grupo.

## Aprendizados (viraram notas)

- [[Fornecedor externo entra pelo contrato do app, não o app pelo dele]] — o
  princípio que faltava na Base; o adapter de pagamento foi o terceiro caso.
- [[Webhook de dinheiro precisa de duas travas, a do evento e a do efeito]]
- [[Dublê que não fecha o fluxo deixa o caminho sem ninguém passar]]
- [[Rolagem horizontal que não se anuncia esconde a coluna que decide]]
- [[Sem virtualização na BIOS não há Docker no Windows; o banco de dev vira Postgres portátil]] —
  ganhou a parte do pacote que traz só o servidor e a do `pg_ctl` que prende o terminal.

## Próximos passos

- [ ] Decidir o nome de verdade.
- [ ] Credencial do Mercado Pago e domínio com HTTPS (sem isso o webhook do
      Telegram não tem onde ser entregue).
- [ ] Cobrar a mensalidade da plataforma: hoje o plano da conta é só um campo, e
      ninguém cobra por ele. A taxa por venda já é descontada e congelada.
- [ ] Compose de produção e agendador da manutenção.
- [ ] Renovação antes do vencimento, dentro do mesmo chat.

## Conexões
- Usa: [[Design]] · [[Infra]] · [[Backend]]
- Mapa: [[Projetos]]
