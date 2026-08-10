---
tags: [tipo/atomica, camada/referencia, dev/backend, whatsapp]
criado: 2026-08-10
---

# Janela de 24h do WhatsApp - dentro é grátis, fora é só template pago

> No WhatsApp Business (Cloud API) a última mensagem do cliente abre uma **janela de
> 24h**. DENTRO dela você responde **texto livre de graça**. FORA dela, o texto livre é
> **rejeitado** pela Meta — só sai **template pré-aprovado**, e isso é o que é **cobrado**.
> A janela governa o produto inteiro, não é detalhe de cobrança.

## As duas metades

- **Service message** (dentro da janela): qualquer conteúdo livre — texto, mídia. É a
  resposta ao cliente que acabou de escrever. **Grátis, ilimitado.** Cobre ~todo o atendimento.
- **Template** (fora da janela, ou pra iniciar conversa): mensagem pré-aprovada com
  placeholders `{{1}}`, `{{2}}`. **É o único jeito de reengajar** depois de 24h. Cobrado
  por mensagem, por categoria:
  - **Marketing** — pago (o mais caro).
  - **Utility** (ex.: "seu documento está pronto") — pago, **mas grátis se enviado com a
    janela ainda aberta**.
  - **Authentication** (OTP) — pago.

Mudança de jul/2025: a Meta passou de "por conversa de 24h" pra **por mensagem** de template;
service messages ficaram de graça.

## O que isso força no código

- O composer tem **dois modos**: janela aberta → texto livre; janela fechada → só o picker
  de template. Calcular a janela é olhar o timestamp da **última mensagem IN** (< 24h).
- Tentar texto livre fora da janela volta o erro **131047** ("re-engagement message"). Trate
  como "use template", não como falha genérica — não persista uma mensagem fantasma que
  nunca vai sair.
- O template é **autorado e aprovado no WhatsApp Manager** (lado da Meta). O app só espelha
  (sync via `GET /{WABA_ID}/message_templates`) e envia o que está `APPROVED`. Não dá pra
  "criar template" de dentro do seu app.

## Conexões
- Depende de: [[Persistir a mensagem não espera a entrega, a entrega é status]]
- Visto em: [[navetalks]]
- Mapa: [[Backend]]
