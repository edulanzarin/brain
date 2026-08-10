---
tags: [tipo/atomica, camada/padrao, dev/backend]
criado: 2026-08-10
---

# Webhook de provedor chega repetido e fora de ordem, a borda tolera os dois

> A entrega de webhook de terceiro é **at-least-once e sem ordem garantida**: o mesmo
> evento chega mais de uma vez, e eventos correlatos chegam trocados. Não conserte isso com
> lógica de dedupe ad-hoc — deixe a **estrutura** garantir. Idempotência mora numa **chave
> única**; ordem, num **status que só avança**.

## Repetido → chave única

Todo evento traz um id do provedor (no WhatsApp, o `wamid`). Grave-o numa coluna **`@unique`**
(`Message.externalId`). Antes de inserir a mensagem recebida, um `findUnique` por esse id: se
já existe, ignora. A reentrega vira no-op sem race, porque quem barra o duplicado é o banco,
não um `if`. É [[Um invariante se garante na estrutura, não no processo]] aplicado à borda.

Corolário: responda **200 rápido** ao webhook mesmo em erro de processamento — o provedor
reentrega em cima de qualquer não-200, e a idempotência já cobre a reentrega. Devolver 500
por um bug seu só gera loop de reentrega; loga e dá 200.

## Fora de ordem → status monotônico

Recibos de status (`enviado → entregue → lida`) chegam trocados. Rankeie os estados e **só
avance**: `lida` não volta pra `entregue` só porque o webbook de "entregue" chegou depois.

```
const RANK = { ENVIADO: 1, ENTREGUE: 2, LIDA: 3 };
if ((RANK[atual] ?? 0) >= RANK[novo]) return; // nunca rebaixa
```

A mesma chave única correlaciona o recibo com a mensagem enviada (`externalId = wamid`).

## Por que na borda

Idempotência e monotonicidade são propriedades do **handler**, não de cada produtor de
evento. Concentre as duas na função única de ingestão — um provider novo (ou um segundo
gatilho, tipo polling) herda de graça. Mesma família de
[[Adapter de canal isola o app do provider de mensageria]], onde a ingestão é peça única.

## Conexões
- Princípio: [[Um invariante se garante na estrutura, não no processo]]
- Irmã: [[Webhook de terceiro se valida pela assinatura antes de confiar no corpo]]
- Depende de: [[Adapter de canal isola o app do provider de mensageria]]
- Visto em: [[navetalks]]
- Mapa: [[Backend]]
