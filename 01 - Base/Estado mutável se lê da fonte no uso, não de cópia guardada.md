---
tags: [tipo/atomica, camada/principio]
criado: 2026-08-18
---

# Estado mutável se lê da fonte no uso, não de cópia guardada

> Se o valor muda por fora (outra frente, outro processo, outro dispositivo), toda cópia
> guardada é uma bomba-relógio: releia da fonte persistida no momento de usar.

## A regra

Estado que mais de um caminho escreve tem UMA fonte persistida (banco). Quem precisa do
valor lê de lá na hora do uso. Cópia em memória, em localStorage ou em variável de
sessão só vale como cache de curtíssima vida — nunca como verdade.

## Por que

Dois casos no mesmo dia, no mesmo sistema (robô do piwdex):

1. **Credencial rotativa.** Três frentes renovavam o token do jogo (WebSocket, poller de
   snapshots, auto-compra), cada uma guardando a própria cópia em memória. O refresh
   rotaciona o token: a cópia de uma frente apodrecia quando outra renovava, e toda
   venda/compra daquela frente morria em 401 — silencioso, por semanas ("vive falhando").
2. **Config no cliente.** As travas de venda moravam no localStorage do navegador e só
   eram enviadas ao servidor por UM caminho da UI. Hunt iniciada por outro botão nascia
   sem venda, com a tela dizendo "Ligado". A config que o servidor executa tem que morar
   no servidor.

O sintoma típico é intermitência inexplicável: funciona depois de religar (cópia
fresca), degrada com o tempo (cópia velha), "às vezes funciona" (depende do caminho).

## Na prática

- Toda frente que renova credencial persiste o novo valor NA HORA; toda frente que vai
  usar credencial relê do banco antes da chamada.
- Config de automação: o cliente edita e salva no servidor; o motor aplica a config
  salva em todo início de fluxo. O cliente nunca é o transportador da config.
- Cache em memória é aceitável com invalidação clara ou TTL curto — e nunca para valor
  que outra frente rotaciona.

## Conexões
- Irmã: [[Estado vivo se empurra, não se pergunta]] · [[Guarde a intenção e o processo se reconstrói dela]]
- Técnica que aplica: [[Token que rotaciona não tolera cópia longeva, releia do banco antes do uso]] · [[Config que o motor executa mora no servidor e se aplica em todo início de fluxo]]
- Mapa: [[Base]]
