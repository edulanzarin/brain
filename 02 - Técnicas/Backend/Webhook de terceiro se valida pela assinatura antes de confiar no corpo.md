---
tags: [tipo/atomica, camada/padrao, dev/backend, seguranca]
criado: 2026-08-10
---

# Webhook de terceiro se valida pela assinatura antes de confiar no corpo

> Um endpoint de webhook é uma URL pública: qualquer um pode fazer POST nela. Antes de
> processar o corpo, prove que veio mesmo do provedor — pela **assinatura HMAC** que ele põe
> no header, calculada sobre o **corpo cru** com o **segredo do app**. Sem assinatura válida,
> rejeita (401) e não toca em nada.

## O mecanismo

O provedor manda `X-Hub-Signature-256: sha256=<hmac>`, onde o hmac é
`HMAC-SHA256(rawBody, appSecret)`. Você recalcula e compara:

```
const raw = await req.text();               // o corpo CRU, antes de qualquer parse
const expected = "sha256=" + hmac("sha256", appSecret).update(raw).digest("hex");
if (!timingSafeEqual(header, expected)) return 401;
const payload = JSON.parse(raw);            // só agora
```

Dois detalhes que quebram silenciosamente:

- **Assine o corpo cru, não o objeto re-serializado.** `JSON.stringify(JSON.parse(raw))` muda
  espaços e ordem de chaves — o hmac não bate. Leia `req.text()` primeiro, verifique, depois
  faça o parse. (No Next App Router, `req.json()` consome o corpo e você perde o cru.)
- **Sem segredo configurado, rejeite** em vez de "pular a checagem". Um endpoint que aceita
  tudo quando o env está vazio é pior que um que recusa tudo.

## Duas verificações diferentes

- **GET de verificação** (handshake inicial): o provedor manda um `verify_token` que você
  escolheu e guardou no env; devolva o `challenge` só se bater. É one-time, no cadastro.
- **POST de evento**: a assinatura HMAC acima, em toda entrega.

Não confunda: o verify token autentica o **cadastro**; a assinatura autentica **cada payload**.

## Conexões
- Irmã: [[Webhook de provedor chega repetido e fora de ordem, a borda tolera os dois]]
- Visto em: [[navetalks]]
- Mapa: [[Backend]]

<!-- Folha por ora: o princípio-mãe ("entrada de fora da fronteira é hostil até se provar o
     contrário") ainda não está na Base. Ecoa em validar dígito verificador na entrada e em
     escopo clampado no servidor, mas ali é conteúdo/permissão, aqui é autenticidade da
     origem. Candidato a princípio num segundo caso concreto de assinatura/HMAC. Não inventar
     o link agora (regra dos dois casos). -->
