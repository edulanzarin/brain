---
tags: [tipo/atomica, camada/padrao, dev/backend]
criado: 2026-08-03
---

# Persistir a mensagem não espera a entrega, a entrega é status

> Ao mandar uma mensagem por um canal externo (WhatsApp, e-mail, push), separe dois
> passos que a intuição junta: **gravar a mensagem** (durável, interno, sempre real) e
> **entregar ao destinatário** (efeito externo, pode falhar, pode não estar ligado). A
> entrega não é pré-requisito da gravação — vira um **status** da mensagem já salva:
> `pendente → enviado → entregue → lida`. Nada é perdido nem fingido.

## O problema

Se o envio é uma coisa só ("salvar = ter entregue"), o app fica refém do provider: sem
conector ligado não dá pra responder, uma falha de rede perde a resposta digitada, e o
código simula "enviado" pra não travar a UI — mentira que aparece como bug depois. Pior:
os recibos de entrega/leitura chegam **depois**, por webhook, e não há onde encaixá-los.

## A costura

A ação de escrever **sempre persiste** a mensagem interna e move o estado que ela implica
(resumo da conversa, assume o atendimento, tira da fila). Só então chama o seam de
entrega, que **nunca lança**: devolve um status.

- Conector configurado e canal no ar → entrega de verdade, grava o id externo (o `wamid`
  do WhatsApp), status `enviado`; os webhooks depois sobem pra `entregue`/`lida`,
  correlacionados por esse id.
- Sem conector ou canal fora do ar → status `pendente`. A mensagem existe, sai quando o
  número conectar. O id externo fica nulo até lá.

```
sendText(conv, body):
  msg = persist(OUT, body, status = deliver(...).status, externalId = ...)
  update(conv, snippet, assume, sai-da-fila)       # interno, sempre
  # deliver() decide enviado|pendente; jamais bloqueia a persistência
```

O status guardado **reflete a realidade** — a UI mostra relógio (pendente), um tique
(enviado), dois tiques (entregue), dois azuis (lida). Não há estado fingido.

## O que fez valer

- **Ligar o WhatsApp de verdade não mexe em mais nada:** a escrita já é real, falta só o
  seam entregar. É o "último passo" isolado num arquivo.
- **O id externo mora na mensagem desde o envio**, então o webhook de status tem onde
  cair sem procurar. Mesmo espírito da ingestão idempotente do
  [[Adapter de canal isola o app do provider de mensageria]].
- **A UI otimista fica honesta:** a bolha aparece na hora (`useOptimistic`) e é
  reconciliada com o que o servidor persistiu — o otimismo é de latência, não de verdade.

## Conexões
- Irmã: [[Adapter de canal isola o app do provider de mensageria]]
- Visto em: [[navetalks]]
- Mapa: [[Backend]]

<!-- Folha por ora, como a irmã: o princípio-mãe ("efeito externo trocável/falível fica
     atrás de uma costura, e o durável não o espera") ainda não está na Base. Dois casos
     já: o worker que entrega fila da linhagem A e este seam da linhagem B. Terceiro caso
     candidato promove à Base — não inventar o princípio agora (regra dos dois casos). -->
