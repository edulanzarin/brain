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

Landing pronta (React/Vite). Backend de pagamento construído e verificado
end-to-end **localmente** (migration, inscrição, `/api/payment/status`, fallback
do SPA) com token do MP e SMTP falsos — o fluxo até o Mercado Pago está provado.
Falta rodar com **credenciais reais**, definir o domínio público
(`PUBLIC_BASE_URL`) e o deploy. Vive na branch `feat/mercadopago-checkout`.

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

## Aprendizados (viraram notas)

Só links. O texto mora na nota de técnica/princípio.

- [[Polling substitui webhook quando não há IP público]]
- [[Migrations em container próprio no Docker Compose]]

## Próximos passos

- [ ] Preencher o `.env` real (access token do MP, senha de app do Gmail, senha do Postgres)
- [ ] Ativar **pix** e **parcelamento até 6x** no painel do Mercado Pago
- [ ] Definir `PUBLIC_BASE_URL` (domínio) e, com IP público, o webhook `/api/mp/webhook`
- [ ] **Rotacionar** o access token e a senha de app (foram colados em chat)
- [ ] Deploy e merge da branch `feat/mercadopago-checkout`

## Conexões
- Usa: [[Design]] · [[Infra]]
- Mapa: [[Projetos]]
