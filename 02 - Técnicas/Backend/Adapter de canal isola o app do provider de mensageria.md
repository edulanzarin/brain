---
tags: [tipo/atomica, camada/padrao, dev/backend]
criado: 2026-07-28
---

# Adapter de canal isola o app do provider de mensageria

> Quando o fornecedor de um canal externo é trocável — WhatsApp por Baileys (não oficial)
> hoje, Cloud API oficial da Meta amanhã —, programe contra uma INTERFACE de canal, não
> contra o fornecedor. O inbox e o worker só falam com a interface; trocar (ou somar) um
> provider é um `case` novo numa fábrica, sem tocar no resto.

## O problema

O provider muda por razão de negócio (custo, risco de ToS, homologação), não técnica. Se
o app chama a lib do Baileys direto, migrar pra Cloud API vira uma caça: cada envio, cada
parse de mensagem recebida, cada status espalhado pelo código. E dá pra querer os DOIS ao
mesmo tempo — um número no não oficial, outro no oficial.

## A costura

Uma interface `ChannelProvider` com o mínimo que o app precisa: `connect`, `disconnect`,
`send`, e um `onEvent` por onde chegam mensagem recebida e mudança de status. Uma fábrica
`createProvider(name)` devolve a implementação. O inbox e o worker dependem só da
interface e da fábrica.

```
lib/channels/
  types.ts     ChannelProvider, InboundMessage, OutboundMessage, ChannelEvent
  stub.ts      provider de mentira (sobe sem WhatsApp real)
  baileys.ts   (a implementar) — não oficial, QR, processo vivo
  cloud.ts     (a implementar) — oficial, webhook, template
  registry.ts  createProvider(name) -> ChannelProvider
  ingest.ts    ingestInbound() — grava a mensagem recebida (idempotente)
```

O que entra por qualquer provider converge num formato só (`InboundMessage`) e cai numa
função de ingestão única — então o resto do app nunca vê Baileys nem Cloud, vê "mensagem".

## O que fez valer

- **Um stub como primeiro provider** deixou a base inteira (inbox, worker, schema) subir e
  ser verificada sem WhatsApp real nem burocracia — o provider de verdade entra depois.
- **A ingestão é a mesma peça pra vários gatilhos** (evento do worker e, no futuro, webhook
  e polling), então a idempotência mora nela, não em cada provider. Mesma família de
  [[Polling substitui webhook quando não há IP público]].

## Conexões
- Irmã: [[Polling substitui webhook quando não há IP público]]
- Visto em: [[navetalks]]
- Mapa: [[Backend]]

<!-- Folha por ora: o princípio-mãe ("recurso externo trocável fica atrás de uma costura")
     ainda não está na Base. Candidato a virar princípio num terceiro caso — já ecoa em
     polling/webhook e em trocar o backend de armazenamento sem downtime. Não inventar o
     princípio agora (regra dos dois casos). -->
